<template>
  <div class="flex-shrink-0">
    <b-container>
      <b-row>
        <b-col class="text-center" id="big-box">
          <p>Text about the website...</p>
          <p>How to book...</p>
        </b-col>
      </b-row>
      <b-row>
        <b-col cols="7" id="map">
          <p>[INSERT MAPBOX COMPONENT]</p>
        </b-col>
        <b-col id="info-box">
          <p>Info-box</p>
          <!-- Use dynamic components here for info-box functionality -->
        </b-col>
      </b-row>
    </b-container>
  </div>
</template>

<script>
import Paho from '../../libraries/paho.javascript-1.1.0/paho-mqtt.js'

export default {
  name: 'home',
  data() {
    return {
      message: 'none'
    }
  },
  mounted() {
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
      client.subscribe('test')
      var message = new Paho.Message('Hello')
      message.destinationName = 'World'
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
