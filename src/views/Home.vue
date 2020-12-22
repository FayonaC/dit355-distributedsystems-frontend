<template>
  <div>
    <b-container fluid>
      <div class="row">
        <div class="col-3">
          <div class="calendar">
            <label>Pick a date to see available dentists</label>
            <b-form-datepicker id="date" v-model="availabilityRequest.date"></b-form-datepicker>
            <b-button v-on:click="publish">Request date</b-button>
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
      message: 'none'
    }
  },
  mounted() {
    var map = L.map('map').setView([57.708870, 11.974560], 12)

    L.tileLayer('https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token={accessToken}', {
      attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
      maxZoom: 18,
      id: 'mapbox/streets-v11',
      tileSize: 512,
      zoomOffset: -1,
      accessToken: 'pk.eyJ1IjoiZGlzdHJpYnN5cyIsImEiOiJja2llZml2aG0xc2dxMnhvNW55bm1hd3U1In0.V731WqpeBOC6a8wwZUwwAA' // Insert access token here
    }).addTo(map)

    // set callback handlers
    client.onConnectionLost = onConnectionLost
    client.onMessageArrived = onMessageArrived

    // connect the client
    client.connect({ onSuccess: onConnect })

    // called when the client connects
    function onConnect() {
      // Once a connection has been made, make a subscription to a topic and send a message.
      console.log('Connected')
      var subOptions = {
        qos: 1
      }
      client.subscribe('Dentists', subOptions)
      var message = new Paho.Message('Hello from dentists')
      message.destinationName = 'Dentist'
      message.qos = 1
      client.send(message)

    /* client.subscribe('free-slots') // This might need to change
      var messageTwo = new Paho.Message('Hello from availability')
      messageTwo.destinationName = 'Availability'
      client.send(messageTwo) */
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
      var parsing = JSON.parse(message.payloadString) // Parsing the incoming JSON (message)
      parsing.dentists.forEach(function (dentists, i) { // Iterates through all the dentists in the JSON
        dentists.id = i
        var longitudeMap = (parsing.dentists[i].coordinate.longitude) // Saves the longitude in a variable
        var latitudeMap = (parsing.dentists[i].coordinate.latitude) // Saves the latitude in a variable
        var marker = L.marker([latitudeMap, longitudeMap]).addTo(map) // Uses the stored coordinates and adds them to the map as markers
        // getting data we need for the popups
        var officeName = (parsing.dentists[i].name)
        var dentistNum = (parsing.dentists[i].dentists)
        var address = (parsing.dentists[i].address)
        var city = (parsing.dentists[i].city)
        var monHours = (parsing.dentists[i].openinghours.monday)
        var tuesHours = (parsing.dentists[i].openinghours.tuesday)
        var wedHours = (parsing.dentists[i].openinghours.wednesday)
        var thursHours = (parsing.dentists[i].openinghours.thursday)
        var friHours = (parsing.dentists[i].openinghours.friday)

        // adds dental office information as popups on markers

        marker.bindPopup('<b>Dental office:</b> ' + officeName + '<br><b>Address:</b> ' + address + ', <br>' + city +
        '<br><b>Number of dentists:</b> ' + dentistNum + '<br><b>Opening hours:</b> ' + '<br>Monday: ' + monHours +
        '<br>Tuesday: ' + tuesHours + '<br>Wednesday: ' + wedHours + '<br>Thursday: ' + thursHours + '<br>Friday: ' +
        friHours + '<br><a class="btn btn-primary" href="/booking">Book appointment</a>')
      })
    }
  },
  methods: {
    publish() {
      // Creates an availability request
      var availabilityRequest = {
        date: this.availabilityRequest.date
      }
      console.log(JSON.stringify(availabilityRequest))
      var message = new Paho.Message(JSON.stringify(availabilityRequest))
      message.topic = 'AvailabilityRequest'
      message.qos = 1
      client.publish(message)
    }
  }
}
</script>
