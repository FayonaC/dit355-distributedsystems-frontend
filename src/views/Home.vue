<template>
  <div>
    <b-container>
      <div id="map" class="map pad2"></div>
    </b-container>
  </div>
</template>

<script>
import Paho from '../../libraries/paho.javascript-1.1.0/paho-mqtt.js'
import mapboxgl from 'mapbox-gl'
mapboxgl.accessToken = '' // Insert access token here

export default {
  name: 'home',
  data() {
    return {
      message: 'none'
    }
  },
  mounted() {
    var map = new mapboxgl.Map({
      container: 'map',
      style: 'mapbox://styles/mapbox/streets-v11',
      center: [11.974560, 57.708870],
      zoom: 12
    })
    map.location()

    // Create a client instance
    var client = new Paho.Client(location.hostname, Number(9001), '', 'frontend')

    // set callback handlers
    client.onConnectionLost = onConnectionLost
    client.onMessageArrived = onMessageArrived

    // connect the client
    client.connect({ onSuccess: onConnect })

    // called when the client connects
    function onConnect() {
      // Once a connection has been made, make a subscription and send a message.
      console.log('Connected')
      client.subscribe('Dentists')
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
    }
  },
  methods: {
    publish() {
      var client = new Paho.Client(location.hostname, Number(9001), '', 'frontend')

      // set callback handlers
      client.onConnectionLost = onConnectionLost
      client.onMessageArrived = onMessageArrived

      // connect the client
      client.connect({ onSuccess: onConnect })

      // called when the client connects
      function onConnect() {
        var message = new Paho.Message('Greetings')
        message.topic = 'test'
        client.publish(message)
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
      }
    }
  }
}
</script>
