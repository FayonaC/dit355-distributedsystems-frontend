<template>
  <div>
    <h1>Appointment Booking Form</h1>
    <b-container>
      <b-alert v-model="showDismissibleAlert" variant="danger" dismissible>
        {{ msg }}
      </b-alert>
      <b-alert v-model="showDismissibleSuccess" variant="success" dismissible>
        {{ msg }}
      </b-alert>
      <form>
        <label>User ID </label>
        <b-form-input type="number"
          v-model.number="booking.userId"
          placeholder="Enter your user ID"
        ></b-form-input>

        <label>Request ID </label>
        <b-form-input type="number"
          v-model.number="booking.requestId"
          placeholder="Enter your request ID"
        ></b-form-input>

        <label>Dentist ID </label>
        <b-form-input type="number"
          v-model.number="booking.dentistId"
          placeholder="Enter your dentist ID"
        ></b-form-input>

        <label>Issuance Number </label>
        <b-form-input type="number"
          v-model.number="booking.issuance"
          placeholder="Enter your issuance number"
        ></b-form-input>

        <label>Date and Time</label>
        <b-form-input
          v-model="booking.time"
          placeholder="Enter your appointment date and time"
        ></b-form-input>

      <!-- For calender and appointment time
      <label>Date and Time </label>
      <b-form-datepicker
        id="date-blah"
        v-model="message.date"
      ></b-form-datepicker>

      <label>Appointment Time </label>
      <b-form-select
        v-model="message.priority"
        :options="appointmentTimes"
      ></b-form-select> -->

        <b-button v-on:click="publish">Submit Booking</b-button>
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
