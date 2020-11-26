<template>
    <div>
        <b-container>
        <h1>Appointment Booking Form</h1>
        <form>
        <label>User ID </label>
        <b-form-input v-model="message.name" placeholder="Enter your name"></b-form-input>

        <label>Dentist </label>
        <b-form-select v-model="message.dentist" :options="dentistId"></b-form-select>

       <label>Date </label>
        <b-form-datepicker id="date-blah" v-model="message.date"></b-form-datepicker>

        <label>Appointment Time </label>
        <b-form-select v-model="message.priority" :options="appointmentTimes"></b-form-select>

        <b-button >Submit Booking</b-button>
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
        requestId: null,
        issuance: null,
        date: '',
        dentistId: ''
      }
    }
  },

  methods: {
    // create a booking method that sends the correct information needed for the JSON and publishes it
  }
}
</script>
