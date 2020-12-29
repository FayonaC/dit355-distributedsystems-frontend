<template>
  <div>
    <b-container fluid>
      <div class="row">
        <div class="col-3">
          <div class="calendar">
            <label>Pick a date to see available dentists</label>
            <b-form-datepicker id="date" v-model="availabilityRequest.date"></b-form-datepicker>
            <b-button v-on:click="publishAvailabilityRequest">Request date</b-button>
          </div>
        </div>
        <div class="col-9">
          <div id="map"></div>
        </div>
      </div>
    </b-container>
  </div>
</template>

<script>
import Paho from '../../libraries/paho.javascript-1.1.0/paho-mqtt.js'
import L from 'leaflet'

var client = new Paho.Client(location.hostname, Number(9001), '', 'frontend')

export default {
  name: 'home',
  data() {
    return {
      availabilityRequest: {
        date: null
      },
      officeRequest: {
        name: ''
      },
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

    this.map = L.map('map').setView([57.708870, 11.974560], 12)

    L.tileLayer('https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token={accessToken}', {
      attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
      maxZoom: 18,
      id: 'mapbox/streets-v11',
      tileSize: 512,
      zoomOffset: -1,
      accessToken: 'pk.eyJ1IjoiZGlzdHJpYnN5cyIsImEiOiJja2llZml2aG0xc2dxMnhvNW55bm1hd3U1In0.V731WqpeBOC6a8wwZUwwAA' // Insert access token here
    }).addTo(this.map)

    this.layerGroup = L.layerGroup().addTo(this.map)
  },
  methods: {
    subscribe() {
      // set callback handlers
      client.onConnectionLost = onConnectionLost
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
        client.subscribe('Dentists', subOptions)
        var message = new Paho.Message('Hello from dentists')
        message.destinationName = 'Dentist'
        message.qos = 1
        client.send(message)

        client.subscribe('free-slots')
        var messageTwo = new Paho.Message('Hello from availability')
        messageTwo.destinationName = 'Availability'
        client.send(messageTwo)
      }

      // called when the client loses its connection
      function onConnectionLost(responseObject) {
        if (responseObject.errorCode !== 0) {
          console.log('onConnectionLost:' + responseObject.errorMessage)
        }
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
    publishOfficeRequest() {
      var officeRequest = {
        name: this.officeRequest.name
      }
      console.log(JSON.stringify(officeRequest))
      var message = new Paho.Message(JSON.stringify(officeRequest))
      message.topic = 'OfficeRequest'
      message.qos = 1
      client.publish(message)
    },
    onMessageArrived(message) {
      if (JSON.parse(message.payloadString).dentists) {
        this.getDentists(message, this.map)
      } else if (JSON.parse(message.payloadString).schedules) {
        this.availability(message, this.map)
        console.log(this.schedules)
      }
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
      this.dentists.forEach(function (dentists, i) { // Iterates through all the dentists in the JSON
        var available = false
        var longitudeMap = (dentists.coordinate.longitude) // Saves the longitude in a variable
        var latitudeMap = (dentists.coordinate.latitude) // Saves the latitude in a variable
        var marker = {}
        // Match dentist to dentist id in availability and set opacity based on that
        if (unavailable.length > 0) {
          unavailable.forEach(function (un, i) {
            if (un.dentist === dentists.id) {
              marker = L.marker([latitudeMap, longitudeMap]).setOpacity(0.5).addTo(layerGroup)
            } else {
              marker = L.marker([latitudeMap, longitudeMap]).setOpacity(1).addTo(layerGroup) // Uses the stored coordinates and adds them to the map as markers
              available = true
            }
          })
        } else {
          marker = L.marker([latitudeMap, longitudeMap]).setOpacity(1).addTo(layerGroup) // Uses the stored coordinates and adds them to the map as markers
          available = true
        }
        // getting data we need for the popups
        var officeName = (dentists.name)
        var dentistNum = (dentists.dentists)
        var address = (dentists.address)
        var city = (dentists.city)
        var monHours = (dentists.openinghours.monday)
        var tuesHours = (dentists.openinghours.tuesday)
        var wedHours = (dentists.openinghours.wednesday)
        var thursHours = (dentists.openinghours.thursday)
        var friHours = (dentists.openinghours.friday)

        // adds dental office information as popups on markers

        marker.bindPopup('<b>Dental office:</b> ' + officeName + '<br><b>Address:</b> ' + address + ', <br>' + city +
        '<br><b>Number of dentists:</b> ' + dentistNum + '<br><b>Opening hours:</b> ' + '<br>Monday: ' + monHours +
        '<br>Tuesday: ' + tuesHours + '<br>Wednesday: ' + wedHours + '<br>Thursday: ' + thursHours + '<br>Friday: ' +
        friHours + getButton(available))
      })
      function getButton(available) {
        if (available) {
          return '<br><a class="btn btn-primary"  href="/booking">Book appointment</a>'
        } else {
          return '<br><a class="btn btn-primary disabled" href="/booking">Book appointment</a>'
        }
      }
    },
    getDentists(message, map) {
      this.dentists = JSON.parse(message.payloadString).dentists // Parsing the incoming JSON (message)
      this.createMarkers(map, 1)
    }
  }
}
</script>
