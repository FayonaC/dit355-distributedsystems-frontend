<template>
  <div>
    <b-container>
      <h1>Appointment Booking Form</h1>
      <form>
        <label>User ID </label>
        <b-form-input
          v-model="message.userId"
          placeholder="Enter your user ID"
        ></b-form-input>

         <label>Request ID </label>
        <b-form-input
          v-model="message.requestId"
          placeholder="Enter your request ID"
        ></b-form-input>

        <label>Dentist ID </label>
        <b-form-input
          v-model="message.dentistId"
          placeholder="Enter your dentist ID"
        ></b-form-input>

         <label>Issuance Number </label>
        <b-form-input
          v-model="message.issuance"
          placeholder="Enter your issuance number"
        ></b-form-input>

        <label>Date and Time</label>
        <b-form-input
          v-model="message.time"
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
      message: {
        userId: '',
        requestId: '',
        dentistId: '',
        issuance: '',
        time: ''
      }
    }
  },

  mounted() {
    // Create a client instance
    this.subscribe()
  },
  methods: {

    subscribe() {
      // user input is taken and put into an obj called "booking"
      const booking = {
        userid: this.message.userId,
        requestid: this.message.requestId,
        dentistid: this.message.dentistId,
        issuance: this.message.issuance,
        time: this.message.time
      }

      // set callback handlers
      client.onConnectionLost = onConnectionLost
      client.onMessageArrived = onMessageArrived

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

      // called when a message arrives
      function onMessageArrived(message) {
        console.log('onMessageArrived:' + message.payloadString)
        if (message.requestid === booking.requestId) {
          console.log('Booking response received')
        }
      }
    },
    publish() {
      const booking = {
        userid: this.message.userId,
        requestid: this.message.requestId,
        dentistid: this.message.dentistId,
        issuance: this.message.issuance,
        time: this.message.time
      }

      var message = new Paho.Message(JSON.stringify(booking))
      message.topic = 'BookingRequest'
      client.publish(message)
      // this.subscribe()
    }
  }
}
</script>
