<template>
  <div>
    <b-container>
      <h1>Office Name</h1>
      <h3>Available appointments:</h3>
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
      <form>
        <label>Enter your user ID (numbers only)</label>
        <b-form-input
          v-model.number="booking.userId"
          placeholder="Ex. 123456"
        ></b-form-input>
        <b-button v-on:click="publish" class="button">Confirm booking</b-button>
      </form>
    </b-container>
  </div>
</template>

<script>
import Paho from '../../libraries/paho.javascript-1.1.0/paho-mqtt.js'
var client = new Paho.Client(location.hostname, Number(9001), '', 'frontend')

export default {
  name: 'booking',
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
      msg: ''

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
        client.subscribe('BookingResponse')
        var message = new Paho.Message('Hello')
        message.destinationName = 'Availability'
        client.send(message)
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
      if (JSON.parse(message.payloadString).requestid === this.booking.requestId) {
        console.log(JSON.parse(message.payloadString).requestid)
        this.msg = 'Booking Response Successful!'
        this.showDismissibleSuccess = true
      } else {
        this.msg = 'None!'
        this.showDismissibleAlert = true
      }
    },
    publish() {
      const booking = { // Creates the booking with all the fields
        userid: this.booking.userId,
        requestid: null,
        dentistid: this.booking.dentistId,
        issuance: Date.now(),
        time: this.booking.time
      }

      if (localStorage.getItem('user') != null) {
        // Checks if there is a user in the localstorage with this userid, if not create a new user, if yes use the requestid of that user and increment it
        if (localStorage.getItem('user').userId === this.booking.userId) {
          console.log('Local storage before incrementing' + localStorage)
          booking.requestid = localStorage.user.requestId++
          localStorage.user.requestId = booking.requestid
          console.log('Printing on line 113 in the if' + localStorage)
        }
      } else {
        console.log('Should have no users saved' + localStorage)
        const user = { // Creates the user with their id and a requestid
          userId: this.booking.userId,
          requestId: 1
        }
        console.log('Local storage after a user has been created' + localStorage)
        booking.requestid = user.requestId
        localStorage.setItem('user', JSON.stringify(user))
      }
      var message = new Paho.Message(JSON.stringify(booking))
      message.topic = 'BookingRequest'
      client.publish(message)
    }
  }
}
</script>
