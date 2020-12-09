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
import d3 from 'd3'
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
    // var marker = new mapboxgl.Marker()

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

    var mapData = Paho.Message

    map.on('load', function () {
      // We use D3 to fetch the JSON here so that we can parse and use it separately
      // from GL JS's use in the added source. You can use any request method (library
      // or otherwise) that you want.
      d3.json(mapData,
        function (err, data) {
          if (err) throw err
          // save full coordinate list for later
          var coordinates = data.dentists.coordinate
          // start by showing just the first coordinate
          data.dentists.coordinate = [coordinates[0]]
          // add it to the map
          map.addSource('trace', { type: 'geojson', data: data })
          map.addLayer({
            id: 'trace',
            type: 'line',
            source: 'trace'
          })
          // setup the viewport
          map.jumpTo({ center: coordinates[0], zoom: 14 })
          map.setPitch(30)
          // on a regular basis, add more coordinates from the saved list and update the map
          var i = 0
          var timer = window.setInterval(function () {
            if (i < coordinates.length) {
              data.dentists.coordinate.push(
                coordinates[i]
              )
              map.getSource('trace').setData(data)
              map.panTo(coordinates[i])
              i++
            } else {
              window.clearInterval(timer)
            }
          }, 10)
        }
      )
      // mapData.dentists.forEach(function (dentist, i) {
      //  dentist.properties.id = i
      // })

    /* map.on('load', function (e) {
      map.addLayer({
        source: {
          type: 'json',
          data: mapData
        },
        layout: {
          icon: marker
        }
      })
      marker.addTo(map) */
    })
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
