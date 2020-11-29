<template>
  <div>
    <b-container>
      <h1>Appointment Booking Form</h1>
      <form>
        <label>User ID </label>
        <b-form-input
          v-model="message.name"
          placeholder="Enter your user ID"
        ></b-form-input>

         <label>Request ID </label>
        <b-form-input
          v-model="message.requestId"
          placeholder="Enter your request ID"
        ></b-form-input>

        <label>Dentist ID </label>
        <b-form-select
          v-model="message.dentistId"
          :options="dentistId"
          placeholder="Enter your dentist ID"

        ></b-form-select>

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

        <b-button>Submit Booking</b-button>
      </form>
    </b-container>
  </div>
</template>

<script>

export default {
  name: 'booking-form',
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

  methods: {
    // create a booking method that sends the correct information needed for the JSON and publishes it
  }
}
</script>
