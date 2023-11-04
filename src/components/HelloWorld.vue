<template>
  <div>
    <h1>Buscador de Sismos</h1>
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
        {{ earthquake.properties.place }} - Magnitud: {{ earthquake.properties.mag }}
      </li>
    </ul>
    <p v-else>No se encontraron terremotos para el rango de fechas seleccionado.</p>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      fechaInicio: '',
      fechaTermino: '',
      magnitud: '',
      earthquakes: [],
    };
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
          })
          .catch(error => {
            console.error('Error al cargar los datos de terremotos', error);
          });
      }
    },
  },
};
</script>