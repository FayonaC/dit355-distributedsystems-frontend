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
        issuance: null,
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
        message.destinationName = 'Toodly pipski!'
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
      const booking = {
        userid: this.booking.userId,
        requestid: this.booking.requestId,
        dentistid: this.booking.dentistId,
        issuance: this.booking.issuance,
        time: this.booking.time
      }

      var message = new Paho.Message(JSON.stringify(booking))
      message.topic = 'BookingRequest'
      client.publish(message)
      // this.subscribe()
    }
  }
}
</script>
