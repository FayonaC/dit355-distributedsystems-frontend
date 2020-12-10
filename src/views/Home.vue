<template>
  <div>
    <b-container>
      <div id="map"></div>
    </b-container>
  </div>
</template>

<script>
import Paho from '../../libraries/paho.javascript-1.1.0/paho-mqtt.js'
// import mapboxgl from 'mapbox-gl'
// import d3 from 'd3'
// mapboxgl.accessToken = '' // Insert access token here
import L from 'leaflet'

export default {
  name: 'home',
  data() {
    return {
      message: 'none'
    }
  },
  mounted() {
    // var requestCount = 0
    // var mapMarkers = []
    // var locations = []
    // var markers = []
    // var geoJsons = []
    // var lines = []
    // var id_Strings = []

    var map = L.map('map').setView([57.708870, 11.974560], 12)

    L.tileLayer('https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token={accessToken}', {
      attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
      maxZoom: 18,
      id: 'mapbox/streets-v11',
      tileSize: 512,
      zoomOffset: -1,
      accessToken: 'pk.eyJ1IjoiZGlzdHJpYnN5cyIsImEiOiJja2llZml2aG0xc2dxMnhvNW55bm1hd3U1In0.V731WqpeBOC6a8wwZUwwAA'
    }).addTo(map)

    // These are hardcoded locations for the initial dental offices
    var dentistOneMarker = L.marker([57.707619, 11.969388]).addTo(map) // Adds the first dentist to the map with coordinates longitude, latitude
    console.log(dentistOneMarker)

    var dentistTwoMarker = L.marker([57.685255, 11.942625]).addTo(map) // Adds the second dentist to the map with coordinates longitude, latitude
    console.log(dentistTwoMarker)

    var dentistThreeMarker = L.marker([57.709872, 11.940386]).addTo(map) // Adds the third dentist to the map with coordinates longitude, latitude
    console.log(dentistThreeMarker)

    /* var map = new mapboxgl.Map({
      container: 'map',
      style: 'mapbox://styles/mapbox/streets-v11',
      center: [11.974560, 57.708870],
      zoom: 12
    })
    // var marker = new mapboxgl.Marker() */

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
      /* var mapData = JSON.stringify(message.data)
      console.log('The data: ' + mapData)
      var longitudeMap = mapData.longitude
      var latitudeMap = mapData.latitude
      console.log('This is longitude: ' + longitudeMap)
      var marker = L.marker(latitudeMap, longitudeMap).addTo(map)
      console.log(marker)

      /* var marker = document.createElement('div')
      marker.className = 'marker'
      markers.push(marker)

      mapMarkers.push(new mapboxgl.Marker(marker)
        .setLngLat(coordinates)
        .addTo(map))

      dentistGeoJSON(mapData) */
    }
    /* function dentistGeoJSON(mapData) {
      var geojson = {
        type: 'Feature',
        properties: {},
        geometry: {
          type: 'Point',
          coordinates: [mapData.latitude, mapData.longitude]
        }
      }
      geoJsons.push(geojson)
      map.addLayer({
        id: 'id',
        type: 'symbol',
        source: {
          type: 'geojson',
          data: geojson
        },
        layout: {
          icon: mapMarkers
        }
      })
    }
  }, */
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
