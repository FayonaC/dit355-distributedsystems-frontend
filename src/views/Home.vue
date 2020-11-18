<template>
  <div id="content-wrap">
    <b-jumbotron header="DentiPicks" lead="Find your new favourite dentist"></b-jumbotron>
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
    console.log('hi')
    var client = new Paho.Client(location.hostname, Number(9001), '', 'frontend')

    console.log(client)

    // set callback handlers
    client.onConnectionLost = onConnectionLost
    client.onMessageArrived = onMessageArrived

    // connect the client
    client.connect({ onSuccess: onConnect })

    // called when the client connects
    function onConnect() {
      // Once a connection has been made, make a subscription and send a message.
      console.log('onConnect')
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
  }
}
</script>
