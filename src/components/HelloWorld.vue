<template>
  <div class="app-container">
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

<style scoped>
.app-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 10px;
  background: linear-gradient(135deg, #e0f7fa, #f1f8e9);
}

.map-container {
  margin-top: 0;
  width: 100%;
  height: 60vh;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  border-radius: 10px;
}

h1 {
  margin-top: 20px;
  font-size: 2em;
  color: #00796b;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.1);
}

.input-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
  margin-top: 20px;
  gap: 15px;
}

input, button {
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  font-size: 1em;
  outline: none;
}

input:focus {
  border-color: #00796b;
  box-shadow: 0 0 5px rgba(0, 121, 107, 0.3);
}

button {
  background: linear-gradient(135deg, #00796b, #004d40);
  color: white;
  cursor: pointer;
  transition: background 0.3s ease;
}

button:hover {
  background: linear-gradient(135deg, #004d40, #00796b);
}

.justified-table {
  margin-top: 20px;
  width: 100%;
  max-width: 1000px;
  border-collapse: collapse;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  border-radius: 10px;
  overflow: hidden;
}

.justified-table th, .justified-table td {
  padding: 10px;
  border: 1px solid #ddd;
  text-align: center;
}

.justified-table th {
  background-color: #00796b;
  color: white;
}

.justified-table tr:nth-child(even) {
  background-color: #e0f2f1;
}

.justified-table tr:hover {
  background-color: #b2dfdb;
}

@media (max-width: 768px) {
  .input-container {
    flex-direction: column;
    width: 100%;
    gap: 10px;
  }

  input, button {
    width: 80%;
    font-size: 0.9em;
  }
}
</style>


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
    this.map = L.map('map', {
      zoomAnimation: false,
      fadeAnimation: true,
      markerZoomAnimation: false
    }).setView([-36.820087, -73.044280], 13);
    // Agrega una capa de mapa base, como OpenStreetMap
    // Agrega una capa de mapa base, como OpenStreetMap, con noWrap para evitar que se repita
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    noWrap: true
  }).addTo(this.map);

  // Ajusta los límites del mapa para evitar que se pueda desplazar fuera de los límites visibles
  const southWest = L.latLng(-90, -180);
  const northEast = L.latLng(90, 180);
  const bounds = L.latLngBounds(southWest, northEast);

  this.map.setMaxBounds(bounds);
  this.map.on('drag', () => {
    this.map.panInsideBounds(bounds, { animate: false });
  });

  this.map.whenReady(() => {
    this.popupTemplate = L.popup(this.popupContent);
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

          // Función para agregar los terremotos con animación
          this.earthquakes.forEach((earthquake, index) => {
            setTimeout(() => {
              const coordinates = earthquake.geometry.coordinates;
              const latlng = [coordinates[1], coordinates[0]]; // Invertir las coordenadas
              const popupContent = `
                <strong>${earthquake.properties.place}</strong><br>
                Magnitud: ${earthquake.properties.mag}<br>
                Fecha: ${this.formatDate(earthquake.properties.time)}
              `;
              const popup = L.popup().setContent(popupContent);

              L.circleMarker(latlng, {
                radius: earthquake.properties.mag * 2,
              }).bindPopup(popup).addTo(this.map);
            }, index * 45); // Espaciar la aparición cada 500ms
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