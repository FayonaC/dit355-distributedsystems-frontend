<template>
  <div>
    <b-container>
      <h1>Book Your Appointment</h1>
      <h3>Available appointments:</h3>
        {{availabilityRequest}}
      <b-alert v-model="showDismissibleAlert" variant="danger" dismissible>
        {{ msg }}
      </b-alert>
      <b-alert v-model="showDismissibleSuccess" variant="success" dismissible>
        {{ msg }}
      </b-alert>
      <div class="button-times">
        <div class="row">
        <div class="col-3">
          <b-button pill>07:00-07:30</b-button>
        </div>
        <div class="col-3">
          <b-button pill>07:30-08:00</b-button>
        </div>
        <div class="col-3">
          <b-button pill>08:00-08:30</b-button>
        </div>
      </div>
      </div>
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
        <input type="submit" class="btn-primary btn button" value="Confirm booking"/>
      </form>
    </b-container>
  </div>
</template>

<script>
import Paho from '../../libraries/paho.javascript-1.1.0/paho-mqtt.js'
var client = new Paho.Client(location.hostname, Number(9001), '', 'frontend')

export default {
  name: 'booking-form',
  props: ['availabilityRequest'],
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
      dentists: [],
      schedules: []

    }
  },

  mounted() {
    // Create a client instance
    this.subscribe()
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
        client.subscribe('BookingResponse', subOptions)
        var message = new Paho.Message('Hello')
        message.destinationName = 'Availability'
        message.qos = 1
        client.send(message)

        client.subscribe('free-slots')
        var messageTwo = new Paho.Message('Hello from availability')
        messageTwo.destinationName = 'Availability'
        messageTwo.qos = 1
        client.send(messageTwo)

        client.subscribe('Dentists', subOptions)
        var messageThree = new Paho.Message('Hello from dentists')
        messageThree.destinationName = 'Dentist'
        messageThree.qos = 1
        client.send(messageThree)

        client.subscribe('OfficeInfo')
        var messageFour = new Paho.Message('Hello from availability')
        messageFour.destinationName = 'Availability'
        messageFour.qos = 1
        client.send(messageFour)
      }

      // called when the client loses its connection
      function onConnectionLost(responseObject) {
        if (responseObject.errorCode !== 0) {
          console.log('onConnectionLost:' + responseObject.errorMessage)
        }
      }
    },
    // called when a message arrives
    onMessageArrived(message) {
      console.log('onMessageArrived:' + message.payloadString)
      this.populateDentistArray(message)
      console.log(JSON.parse(message.payloadString).requestid)

      if (JSON.parse(message.payloadString).requestid === this.booking.requestId) {
        this.msg = 'Booking Response Successful!'
        this.showDismissibleSuccess = true
      } else {
        this.msg = 'None!'
        this.showDismissibleAlert = true
      }
    },
    publish() {
      var booking = { // Creates the booking with all the fields
        userid: this.booking.userId,
        requestid: null,
        dentistid: this.booking.dentistId,
        issuance: Date.now(),
        time: this.booking.time
      }
      // Checks if there are users saved in the localstorage OR if the user id typed in is not the same as the existing user in the localstorage
      if (localStorage.getItem('user') == null || JSON.parse(localStorage.getItem('user')).userId !== this.booking.userId) {
        // Creates the user with their id and a requestid
        const user = {
          userId: this.booking.userId,
          requestId: 1
        }
        booking.requestid = user.requestId
        localStorage.setItem('user', JSON.stringify(user))
      } else {
        // Extra check to confirm that user id in localstorage is the same as the typed in user id (similar to line 110)
        if (JSON.parse(localStorage.getItem('user')).userId === this.booking.userId) {
          booking.requestid = JSON.parse(localStorage.getItem('user')).requestId
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
    populateDentistArray(message) {
      var dentists = JSON.parse(message.payloadString).dentists
      var dentistsNew = []
      var officeName = ''
      for (var i = 0; i < dentists.length; i++) {
        officeName = dentists[i].name
        dentistsNew.push(officeName)
      }
      this.dentists = dentistsNew
    }

  }
}
</script>
