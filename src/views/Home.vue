<template>
  <div>
    <b-container fluid>
      <div class="row">
        <div class="col-3">
          <div class="calendar">
            <label>Pick a date to see available dentists</label>
            <b-form-datepicker
              id="date"
              v-model="availabilityRequest.date"
            ></b-form-datepicker>
            <b-button v-on:click="publishAvailabilityRequest"
              >Request date</b-button
            >
          </div>
          <div>
            <label for="categories">Office Names:</label>
            <div class="col-8">
              <select class="form-control" id="name" v-model="dentist.id">
                <option
                  v-for="dentist in dentists"
                  :key="dentist.id"
                  :value="dentist.id"
                >
                  {{ dentist.name }}
                </option>
              </select>
            </div>
            <b-button v-on:click="displayOfficeTimeSlots"
              >Request office availability</b-button
            >
            <select
              class="form-control"
              id="startTime"
              v-model="appointment.startTime"
            >
              <option v-for="appointment in appointments" :key="appointment.id">
                {{ appointment.startTime }}
              </option>
            </select>
            <b-button v-on:click="setBookingRequestInfo"
              >Select time slot</b-button>
            <label for="categories">To see available appointments click above ^</label>
          </div>
          <div>
            <b-container fluid>
                  <b-container>
      {{ availabilityRequest }}
      <b-alert v-model="showDismissibleAlert" variant="danger" dismissible>
        {{ msg }}
      </b-alert>
      <b-alert v-model="showDismissibleSuccess" variant="success" dismissible>
        {{ msg }}
      </b-alert>

      <form @submit.prevent="publish">
        <label>Enter dentist office ID (for testing only)</label>
        <b-form-input
          v-model.number="booking.dentistId"
          placeholder="Ex. 1"
        ></b-form-input>
        <label>Enter appointment date and time (for testing only)</label>
        <b-form-input
          v-model="booking.time"
          placeholder="Ex. 2020-01-01 10:00"
        ></b-form-input>
        <label>Enter your user ID (numbers only)</label>
        <b-form-input
          v-model.number="booking.userId"
          placeholder="Ex. 123456"
        ></b-form-input>
        <input
          type="submit"
          class="btn-primary btn button"
          value="Confirm booking"
          v-on:click="displayOfficeTimeSlots"
        />
      </form>
    </b-container>
            </b-container>
          </div>
        </div>
        <div class="col-9">
          <b-container fluid>
            <div id="map"></div>
          </b-container>
        </div>
      </div>
    </b-container>
  </div>
</template>

<script>
import Paho from '../../libraries/paho.javascript-1.1.0/paho-mqtt.js'
import L from 'leaflet'

var client = new Paho.Client(location.hostname, Number(9001), '', 'frontend', true)

export default {
  name: 'home',
  data() {
    return {
      booking: {
        userId: null,
        requestId: null,
        dentistId: null,
        time: ''
      },
      showDismissibleAlert: false,
      showDismissibleSuccess: false,
      msg: '',
      availabilityRequest: {
        date: null
      },
      bookingRequestInfo: {
        dentistId: null,
        time: ''
      },
      dentist: {
        id: null
      },
      appointment: {
        startTime: '',
        endTime: ''
      },
      appointments: [],
      message: 'none',
      dentists: [],
      schedules: [],
      map: {},
      unavailable: [],
      layerGroup: {}
    }
  },
  mounted() {
    this.subscribe()

    this.map = L.map('map').setView([57.70887, 11.97456], 12)

    L.tileLayer(
      'https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token={accessToken}',
      {
        attribution:
          'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
        maxZoom: 18,
        id: 'mapbox/streets-v11',
        tileSize: 512,
        zoomOffset: -1,
        accessToken:
          'pk.eyJ1IjoiZGlzdHJpYnN5cyIsImEiOiJja2llZml2aG0xc2dxMnhvNW55bm1hd3U1In0.V731WqpeBOC6a8wwZUwwAA' // Insert access token here
      }
    ).addTo(this.map)

    this.layerGroup = L.layerGroup().addTo(this.map)
  },
  methods: {

    setBookingRequestInfo() {
      var stringDate = JSON.stringify(this.availabilityRequest.date)
      console.log(stringDate)
      var stringTime = JSON.stringify(this.appointment.startTime)
      console.log(stringTime)
      var combo = [stringDate, stringTime].join(' ')
      console.log(combo)
      var bookingRequestInfo = {
        dentistId: this.dentist.id,
        time: combo
      }
      console.log(bookingRequestInfo.dentistId)
      console.log(bookingRequestInfo.time)
    },
    subscribe() {
      // set callback handlers
      client.onConnectionLost = this.onConnectionLost
      client.onMessageArrived = this.onMessageArrived

      // connect the client
      client.connect({ onSuccess: onConnect })

      // called when the client connects
      function onConnect() {
        // Once a connection has been made, make a subscription and send a message.
        console.log('Connected')

        var subOptions = {
          qos: 1
        }
        client.subscribe('BookingResponse', subOptions)
        var message = new Paho.Message('Hello')
        message.destinationName = 'Availability'
        message.qos = 1
        client.send(message)

        client.subscribe('free-slots')
        var messageTwo = new Paho.Message('Hello from availability')
        messageTwo.destinationName = 'Availability'
        client.send(messageTwo)

        client.subscribe('Dentists', subOptions)
        var messageThree = new Paho.Message('Hello from dentists')
        messageThree.destinationName = 'Dentist'
        messageThree.qos = 1
        client.send(messageThree)
      }
    },
    // called when the client loses its connection
    onConnectionLost(responseObject) {
      if (responseObject.errorCode !== 0) {
        this.availabilityRequest = 'Connection Lost! Try refreshing...'
        this.showDismissibleAlert = true
        console.log('onConnectionLost:' + responseObject.errorMessage)
        client.unsubscribe('BookingResponse')
        client.disconnect()
      }
    },
    publishAvailabilityRequest() {
      // Creates an availability request
      var availabilityRequest = {
        date: this.availabilityRequest.date
      }
      console.log(JSON.stringify(availabilityRequest))
      var message = new Paho.Message(JSON.stringify(availabilityRequest))
      message.topic = 'AvailabilityRequest'
      message.qos = 1
      client.publish(message)
    },
    publish() {
      var booking = {
        // Creates the booking with all the fields
        userid: this.booking.userId,
        requestid: null,
        dentistid: this.booking.dentistId,
        issuance: Date.now(),
        time: this.booking.time
      }
      // Checks if there are users saved in the localstorage OR if the user id typed in is not the same as the existing user in the localstorage
      if (
        localStorage.getItem('user') == null ||
        JSON.parse(localStorage.getItem('user')).userId !== this.booking.userId
      ) {
        // Creates the user with their id and a requestid
        const user = {
          userId: this.booking.userId,
          requestId: 1
        }
        booking.requestid = user.requestId
        localStorage.setItem('user', JSON.stringify(user))
      } else {
        // Extra check to confirm that user id in localstorage is the same as the typed in user id (similar to line 110)
        if (
          JSON.parse(localStorage.getItem('user')).userId ===
          this.booking.userId
        ) {
          booking.requestid = JSON.parse(
            localStorage.getItem('user')
          ).requestId
          booking.requestid = booking.requestid + 1
          // Creates the user with their id and a requestid
          const user = {
            userId: this.booking.userId,
            requestId: booking.requestid
          }
          localStorage.setItem('user', JSON.stringify(user))
        }
      }
      console.log('Booking ' + JSON.stringify(booking))
      var message = new Paho.Message(JSON.stringify(booking))
      message.topic = 'BookingRequest'
      message.qos = 1
      client.publish(message)
    },
    displayOfficeTimeSlots() {
      // Creates the times slots for a selected dental office
      var dentistOffice = null
      var appointments = []

      for (var i = 0; i < this.schedules.length; i++) {
        if (this.schedules[i].dentist === this.dentist.id) {
          dentistOffice = this.schedules[i]

          for (var j = 0; j < dentistOffice.timeSlots.length; j++) {
            appointments.push(dentistOffice.timeSlots[j])
          }
        }
      }
      this.appointments = appointments
    },
    onMessageArrived(message) {
      var topic = message.topic

      switch (topic) {
        case 'Dentists':
          this.getDentists(message, this.map)
          break
        case 'free-slots' :
          this.availability(message, this.map)
          console.log(this.schedules)
          break
        case 'BookingResponse' :
          console.log(JSON.parse(message.payloadString).requestid)
          console.log(this.booking.requestId)
          // if (JSON.parse(message.payloadString).requestid === this.booking.requestId
          // ) {
          //   this.msg = 'Booking request successful!'
          //   this.showDismissibleSuccess = true
          // } else if (JSON.parse(message.payloadString).time === 'none'
          // ) {
          this.msg = 'Booking request unsuccessful!'
          this.showDismissibleSuccess = true

          break
        default:
          console.log('No topic found')
      }

      // console.log('this is the message' + JSON.stringify(message))
      // if (JSON.parse(message.payloadString).dentists) {
      //   this.getDentists(message, this.map)
      // } else if (JSON.parse(message.payloadString).schedules) {
      //   this.availability(message, this.map)
      //   console.log(this.schedules)
      // } else if (
      //   JSON.parse(message.payloadString).requestid === this.booking.requestId
      // ) {
      //   this.msg = 'Booking request successful!'
      //   this.showDismissibleSuccess = true
      // } else if (JSON.parse(message.payloadString).time === 'none'
      // ) {
      //   this.msg = 'Booking request unsuccessful!'
      //   this.showDismissibleSuccess = false
      // }
      // console.log('I have received a message')
    },
    availability(message, map) {
      this.unavailable = []
      // match dentist id to marker
      var schedules = JSON.parse(message.payloadString).schedules
      this.schedules = schedules

      var createMarkers = this.createMarkers
      var unavailable = this.unavailable

      schedules.forEach(function (schedule, i) {
        if (schedule.timeSlots.length === 0) {
          // Sets dentist id together with availability status
          var un = {
            dentist: schedule.dentist
          }
          unavailable.push(un)
        } else {
          // Sets dentist id together with availability status
        }
        createMarkers(map)
      })
    },
    createMarkers(map) {
      // Creates a dental office request
      var unavailable = this.unavailable
      this.layerGroup.clearLayers()
      var layerGroup = this.layerGroup
      this.dentists.forEach(function (dentists, i) {
        // Iterates through all the dentists in the JSON
        var longitudeMap = dentists.coordinate.longitude // Saves the longitude in a variable
        var latitudeMap = dentists.coordinate.latitude // Saves the latitude in a variable
        var marker = {}
        // Match dentist to dentist id in availability and set opacity based on that
        if (unavailable.length > 0) {
          unavailable.forEach(function (un, i) {
            if (un.dentist === dentists.id) {
              marker = L.marker([latitudeMap, longitudeMap])
                .setOpacity(0.5)
                .addTo(layerGroup)
            } else {
              marker = L.marker([latitudeMap, longitudeMap])
                .setOpacity(1)
                .addTo(layerGroup) // Uses the stored coordinates and adds them to the map as markers
            }
          })
        } else {
          marker = L.marker([latitudeMap, longitudeMap])
            .setOpacity(1)
            .addTo(layerGroup) // Uses the stored coordinates and adds them to the map as markers
        }
        // getting data we need for the popups
        var officeName = dentists.name
        var dentistNum = dentists.dentists
        var address = dentists.address
        var city = dentists.city
        var monHours = dentists.openinghours.monday
        var tuesHours = dentists.openinghours.tuesday
        var wedHours = dentists.openinghours.wednesday
        var thursHours = dentists.openinghours.thursday
        var friHours = dentists.openinghours.friday

        // adds dental office information as popups on markers

        marker.bindPopup(
          '<b>Dental office:</b> ' +
            officeName +
            '<br><b>Address:</b> ' +
            address +
            ', <br>' +
            city +
            '<br><b>Number of dentists:</b> ' +
            dentistNum +
            '<br><b>Opening hours:</b> ' +
            '<br>Monday: ' +
            monHours +
            '<br>Tuesday: ' +
            tuesHours +
            '<br>Wednesday: ' +
            wedHours +
            '<br>Thursday: ' +
            thursHours +
            '<br>Friday: ' +
            friHours
        )
      })
    },
    getDentists(message, map) {
      this.dentists = JSON.parse(message.payloadString).dentists // Parsing the incoming JSON (message)
      this.createMarkers(map, 1)
    }
  }
}
</script>
