<template>
  <div class="origin">
    <div id="map" class="map-container"></div>
    <h1>Buscador de movimientos telúricos</h1>
    <div class="input-container">
      <label for="fechaInicio">Fecha de Inicio:</label>
      <input type="date" id="fechaInicio" v-model="fechaInicio" />
      <label for="fechaTermino">Fecha de Término:</label>
      <input type="date" id="fechaTermino" v-model="fechaTermino" />
      <label for="magnitud">Magnitud:</label>
      <input type="text" id="magnitud" v-model="magnitud" />
      <button @click="buscarTerremotos">Buscar</button>
    </div>
    <table class="justified-table" v-if="earthquakes.length">
      <thead>
        <tr>
          <th>Lugar</th>
          <th>Magnitud</th>
          <th>Fecha</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="earthquake in earthquakes" :key="earthquake.id">
          <td>{{ earthquake.properties.place }}</td>
          <td>{{ earthquake.properties.mag }}</td>
          <td>{{ formatDate(earthquake.properties.time) }}</td>
        </tr>
      </tbody>
</table>
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
      magnitud: '4.2',
      earthquakes: [],
      map: null,
      popupTemplate: null
    };
  },
  mounted() {
    // Crea un mapa en el div con id "map"
    this.map = L.map('map').setView([-36.820087, -73.044280], 13);
    // Agrega una capa de mapa base, como OpenStreetMap
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png').addTo(this.map);
    this.popupTemplate = L.popup(this.popupContent);

    this.map.on('zoomend', () => {
    // Limpia los marcadores anteriores del mapa al hacer zoom
    this.map.eachLayer(layer => {
      if (layer instanceof L.Marker) {
        this.map.removeLayer(layer);
      }
    });
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
            this.map.eachLayer(layer => {
              if (layer instanceof L.Marker) {
                this.map.removeLayer(layer);
              }
            });
            this.earthquakes = response.data.features;
            // Limpia los marcadores anteriores del mapa
            // Agrega marcadores para los terremotos
            this.earthquakes.forEach(earthquake => {
              const coordinates = earthquake.geometry.coordinates;
              const latlng = [coordinates[1], coordinates[0]]; // Invertir las coordenadas
              const popupContent = `
                <strong>${earthquake.properties.place}</strong><br>
                Magnitud: ${earthquake.properties.mag}<br>
                Fecha: ${this.formatDate(earthquake.properties.time)}
              `;
                //const popup = this.popupTemplate.setContent(popupContent);
                const popup = L.popup().setContent(popupContent);
                // Agrega un círculo con el popup
                L.circleMarker(latlng, {
                  radius: earthquake.properties.mag * 0.8,
                }).bindPopup(popup).addTo(this.map);
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

<style scoped>

.map-container {
  margin-top: 0;
  width: 100%; /* Ajusta el ancho según tus necesidades */
  height: 50vh; /* Ajusta la altura según tus necesidades */
}

.justified-table {
  margin: auto;
  margin-top: 20px;
  width: fit-content;
  min-width: 100%;
  border-collapse: collapse;
}

.justified-table th, .justified-table td {
  text-align: justify;
  padding: 8px;
  border: 1px solid #dddddd;
}

.justified-table th {
  background-color: #f2f2f2;
}

.justified-table tr:nth-child(even) {
  background-color: #f2f2f2;
}

.justified-table tr:nth-child(odd) {
  background-color: #ffffff;
}
.input-container{
  text-align: justify;
  margin: auto;
  display: grid;
  width: 200px;
  height: fit-content;
}

input {
  margin-bottom: 10px;
}
</style>