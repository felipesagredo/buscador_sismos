<template>
  <div>
    <div id="map" style="height: 400px;"></div>
    <h1>Buscador de Sismos alrededor del Mundo</h1>
    <div>
      <label for="fechaInicio">Fecha de Inicio:</label>
      <input type="date" id="fechaInicio" v-model="fechaInicio" />
      <label for="fechaTermino">Fecha de Término:</label>
      <input type="date" id="fechaTermino" v-model="fechaTermino" />
      <label for="magnitud">Magnitud:</label>
      <input type="text" id="magnitud" v-model="magnitud" />
      <button @click="buscarTerremotos">Buscar</button>
    </div>
    <ul v-if="earthquakes.length">
      <li v-for="earthquake in earthquakes" :key="earthquake.id">
        {{ earthquake.properties.place }} - Magnitud: {{ earthquake.properties.mag }} - Fecha: {{ formatDate(earthquake.properties.time) }}
      </li>
    </ul>
    <p v-else>No se encontraron Sismos para el rango de fechas seleccionado.</p>
  </div>
</template>

<script>
import axios from 'axios';
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';

export default {
  data() {
    return {
      fechaInicio: '',
      fechaTermino: '',
      magnitud: '3.5',
      earthquakes: [],
      map: null,
    };
  },
  mounted() {
  // Crea un mapa en el div con id "map"
  this.map = L.map('map').setView([-36.820087, -73.044280], 13);

  // Agrega una capa de mapa base, como OpenStreetMap
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(this.map);


  // Maneja el evento de clic en el mapa
  this.map.on('click', (e) => {
    alert("You clicked the map at " + e.latlng);
  });
},
  methods: {
    buscarTerremotos() {
      if (this.fechaInicio && this.fechaTermino) {
        const startDate = this.fechaInicio;
        const endDate = this.fechaTermino;
        const magnitude = this.magnitud;

        const url = `https://earthquake.usgs.gov/fdsnws/event/1/query?format=geojson&starttime=${startDate}&endtime=${endDate}&minmagnitude=${magnitude}`;

        axios.get(url)
          .then(response => {
            this.earthquakes = response.data.features;

            // Limpia los marcadores anteriores del mapa
            this.map.eachLayer(layer => {
              if (layer instanceof L.Marker) {
                this.map.removeLayer(layer);
              }
            });

            // Agrega marcadores para los terremotos
            this.earthquakes.forEach(earthquake => {
            const coordinates = earthquake.geometry.coordinates;
            const latlng = [coordinates[1], coordinates[0]]; // Invertir las coordenadas

            const popupContent = `
              <strong>${earthquake.properties.place}</strong><br>
              Magnitud: ${earthquake.properties.mag}<br>
              Fecha: ${this.formatDate(earthquake.properties.time)}
            `;
                // Agrega un círculo con propiedades personalizadas
              L.circle(latlng, {
              color: 'red',
              fillColor: '#f03',
              fillOpacity: 0.5,
              radius: earthquake.properties.sig * 10
            }).addTo(this.map);

            L.circle(latlng, {
              radius: earthquake.properties.mag * 5000, // Tamaño del círculo en metros (ajústalo según tus preferencias)
            })
              .bindPopup(popupContent)
              .addTo(this.map);
            });
          })
          .catch(error => {
            console.error('Error al cargar los datos de terremotos', error);
          });
      }
    },
    formatDate(timestamp) {
      const date = new Date(timestamp);
      return date.toLocaleString();
    },
  },
};
</script>