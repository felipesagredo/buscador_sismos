<template>
  <div class="app-container">
    <div id="map" class="map-container"></div>
    <h1>Buscador de movimientos telúricos</h1>
    <div class="input-container">
      <label for="fechaInicio">Fecha de Inicio:</label>
      <input type="date" id="fechaInicio" v-model="fechaInicio" :max="fechaMaxima" />
      <label for="fechaTermino">Fecha de Término:</label>
      <input type="date" id="fechaTermino" v-model="fechaTermino" :max="fechaMaxima" />
      <label for="magnitudMin">Magnitud Mínima:</label>
      <input type="number" id="magnitudMin" v-model="magnitudMin" step="0.1" min="0" placeholder="Ej: 4.2" />
      <label for="magnitudMax">Magnitud Máxima (opcional):</label>
      <input type="number" id="magnitudMax" v-model="magnitudMax" step="0.1" min="0" placeholder="Ej: 7.5" />
      <button @click="buscarTerremotos">Buscar</button>
      <button @click="limpiarMapa" class="clear-button">🗑️ Limpiar Mapa</button>
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
        <tr v-for="earthquake in earthquakes" 
          :key="earthquake.id" 
          @click="ubicarSismo(earthquake)" 
          class="clickable-row"
        >
          <td>{{ earthquake.properties.place }}</td>
          <td>{{ earthquake.properties.mag }}</td>
          <td>{{ formatDate(earthquake.properties.time) }}</td>
        </tr>
      </tbody>
    </table>
    <p v-else>No se encontraron Sismos para el rango de fechas seleccionado.</p>
    
    <!-- Diálogo de confirmación para búsquedas grandes -->
    <div v-if="showConfirmDialog" class="confirm-dialog-overlay" @click.self="cancelarBusqueda">
      <div class="confirm-dialog">
        <h3>⚠️ Muchos resultados encontrados</h3>
        <p>Se encontraron aproximadamente <strong>{{ estimatedCount }}</strong> sismos para los filtros seleccionados.</p>
        <p class="warning-text">Cargar todos estos resultados puede:</p>
        <ul>
          <li>Tomar varios segundos en cargar</li>
          <li>Ralentizar el navegador</li>
          <li>Consumir más recursos de la API</li>
        </ul>
        <div class="dialog-buttons">
          <button @click="continuarBusqueda" class="confirm-button">Continuar de todas formas</button>
          <button @click="cancelarBusqueda" class="cancel-button">Cancelar</button>
        </div>
      </div>
    </div>
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

.clear-button {
  background: linear-gradient(135deg, #d32f2f, #b71c1c);
}

.clear-button:hover {
  background: linear-gradient(135deg, #b71c1c, #d32f2f);
}

.justified-table {
  margin-top: 20px;
  width: 100%;
  max-width: 1000px;
  border-collapse: collapse;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  border-radius: 10px;
  overflow: hidden;
  table-layout: fixed; /* Fija el ancho de las columnas */
}

.justified-table th, .justified-table td {
  padding: 10px;
  border: 1px solid #ddd;
  text-align: center;
  overflow: hidden;
  text-overflow: ellipsis;
  word-wrap: break-word;
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

.clickable-row {
  cursor: pointer;
  transition: background-color 0.2s;
}

.clickable-row:hover {
  background-color: #b2dfdb !important; /* Un color un poco más oscuro al pasar el mouse */
  /* Usamos text-shadow en lugar de font-weight para evitar que cambie el tamaño de las columnas */
  text-shadow: 0.3px 0.3px 0.3px rgba(0, 0, 0, 0.15);
}

.confirm-dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 99999;
  backdrop-filter: blur(2px);
}

.confirm-dialog {
  background: white;
  padding: 30px;
  border-radius: 10px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
  max-width: 500px;
  width: 90%;
}

.confirm-dialog h3 {
  margin-top: 0;
  color: #d32f2f;
  font-size: 1.5em;
}

.confirm-dialog p {
  margin: 15px 0;
  line-height: 1.6;
}

.warning-text {
  font-weight: 600;
  color: #555;
}

.confirm-dialog ul {
  margin: 15px 0;
  padding-left: 25px;
}

.confirm-dialog li {
  margin: 8px 0;
  color: #666;
}

.dialog-buttons {
  display: flex;
  gap: 15px;
  justify-content: flex-end;
  margin-top: 25px;
}

.confirm-button {
  background: linear-gradient(135deg, #00796b, #004d40);
  color: white;
  border: none;
  padding: 12px 24px;
  border-radius: 5px;
  cursor: pointer;
  font-size: 1em;
  font-weight: 600;
  transition: background 0.3s ease;
}

.confirm-button:hover {
  background: linear-gradient(135deg, #004d40, #00796b);
}

.cancel-button {
  background: #e0e0e0;
  color: #333;
  border: none;
  padding: 12px 24px;
  border-radius: 5px;
  cursor: pointer;
  font-size: 1em;
  font-weight: 600;
  transition: background 0.3s ease;
}

.cancel-button:hover {
  background: #bdbdbd;
}
</style>


<script>
import axios from 'axios';
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';

export default {
  data() {
    // Función para formatear fecha en formato YYYY-MM-DD
    const formatDate = (date) => {
      const year = date.getFullYear();
      const month = String(date.getMonth() + 1).padStart(2, '0');
      const day = String(date.getDate()).padStart(2, '0');
      return `${year}-${month}-${day}`;
    };

    // Obtener fechas por defecto: ayer y hoy
    const today = new Date();
    const yesterday = new Date();
    yesterday.setDate(today.getDate() - 1);
    
    // Fecha máxima: mañana
    const tomorrow = new Date();
    tomorrow.setDate(today.getDate() + 1);

    return {
      fechaInicio: formatDate(yesterday),
      fechaTermino: formatDate(today),
      fechaMaxima: formatDate(tomorrow),
      magnitudMin: '4.2',
      magnitudMax: '',
      earthquakes: [],
      map: null,
      popupTemplate: null,
      markers: {},
      pendingTimeouts: [], // Para guardar los timeouts y poder cancelarlos
      showConfirmDialog: false,
      estimatedCount: 0,
      searchParams: null, // Para guardar los parámetros de búsqueda mientras se confirma
    };
  },
  mounted() {
    // TRUCO DE FLUIDEZ:
    // Creamos un renderer con 'padding'.
    // 0.5 significa que dibujará un 50% extra de mapa alrededor de lo que ves.
    // Esto hace que al arrastrar, los puntos ya estén ahí.
    const myRenderer = L.canvas({ padding: 0.5 });

    this.map = L.map('map', {
      zoomAnimation: false,
      fadeAnimation: true,
      markerZoomAnimation: false,
      minZoom: 2,
      renderer: myRenderer, // Asignamos el renderer optimizado
      preferCanvas: true // Forzamos el uso de Canvas
    }).setView([-36.820087, -73.044280], 3);

    // Mapa base
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '&copy; OpenStreetMap contributors',
       // updateWhenIdle: false asegura que el mapa base cargue MIENTRAS arrastras
      updateWhenIdle: false 
    }).addTo(this.map);

    this.map.whenReady(() => {
      this.popupTemplate = L.popup();
      // Realizar búsqueda automática al cargar con fechas por defecto
      this.buscarTerremotos();
    });
  },
  methods: {
    // Nueva función para definir colores
    getColor(magnitud) {
      return magnitud > 7 ? '#d32f2f' : // Rojo oscuro (Peligro)
             magnitud > 6 ? '#f44336' : // Rojo
             magnitud > 5 ? '#ff9800' : // Naranja
             magnitud > 4 ? '#1fb976' : // Verde #1fb976
                            '#4caf50';  // Verde (Menor)
    },

    limpiarMapa() {
      // Cancelar todos los timeouts pendientes
      this.pendingTimeouts.forEach(timeout => clearTimeout(timeout));
      this.pendingTimeouts = [];

      // Limpiar todos los marcadores del mapa
      this.map.eachLayer(layer => {
        if (layer instanceof L.CircleMarker) {
          this.map.removeLayer(layer);
        }
      });

      // Limpiar el objeto de marcadores
      this.markers = {};
      
      // Limpiar la lista de sismos
      this.earthquakes = [];
    },

    buscarTerremotos() {
      if (this.fechaInicio && this.fechaTermino) {
        // Limpiar el mapa ANTES de hacer la nueva búsqueda
        this.limpiarMapa();

        const startDate = this.fechaInicio;
        const endDate = this.fechaTermino;
        const minMagnitude = this.magnitudMin || '0';
        
        // Construimos la URL base
        let url = `https://earthquake.usgs.gov/fdsnws/event/1/query?format=geojson&starttime=${startDate}&endtime=${endDate}&minmagnitude=${minMagnitude}`;
        
        // Agregamos magnitud máxima solo si está definida y no está vacía
        if (this.magnitudMax !== null && this.magnitudMax !== undefined && this.magnitudMax !== '') {
          const maxMag = String(this.magnitudMax).trim();
          if (maxMag !== '') {
            url += `&maxmagnitude=${maxMag}`;
          }
        }

        // Guardar parámetros de búsqueda
        this.searchParams = { url, startDate, endDate, minMagnitude };

        // Primero verificamos cuántos resultados hay (hacemos una consulta con limit=1 para obtener metadata)
        const countUrl = url + '&limit=1';
        
        axios.get(countUrl)
          .then(response => {
            // La API de USGS devuelve metadata con el conteo total en la respuesta
            let totalCount = 0;
            
            // Verificar si existe metadata en la respuesta
            if (response.data && response.data.metadata) {
              totalCount = response.data.metadata.count || 0;
              console.log('Metadata recibido:', response.data.metadata);
              console.log('Total de resultados:', totalCount);
            } else {
              console.warn('No se encontró metadata en la respuesta. Procederemos con la búsqueda y verificaremos después.');
              totalCount = 0; // Forzar verificación posterior
            }
            
            // Si hay más de 20 resultados Y tenemos el conteo del metadata, pedimos confirmación
            if (totalCount > 20 && totalCount > 0) {
              this.estimatedCount = totalCount;
              this.showConfirmDialog = true;
              console.log('Mostrando diálogo de confirmación para', totalCount, 'resultados');
              console.log('showConfirmDialog:', this.showConfirmDialog);
            } else if (totalCount === 0) {
              // Si no pudimos obtener el conteo, proceder con la búsqueda y verificar después
              console.log('No se pudo obtener el conteo del metadata. Procediendo con búsqueda y verificando después.');
              this.ejecutarBusqueda();
            } else {
              // Si son 20 o menos resultados, procedemos directamente
              console.log('Procediendo directamente con', totalCount, 'resultados');
              this.ejecutarBusqueda();
            }
          })
          .catch(error => {
            console.error('Error al verificar cantidad de resultados:', error);
            console.error('Detalles del error:', error.response?.data || error.message);
            // Si hay error en la verificación, procedemos de todas formas
            this.ejecutarBusqueda();
          });
      }
    },

    ejecutarBusqueda() {
      if (!this.searchParams) return;

      this.showConfirmDialog = false;
      const { url } = this.searchParams;
      
      axios.get(url)
          .then(response => {
            const allEarthquakes = response.data.features || [];
            
            // Verificar si hay más de 20 resultados DESPUÉS de recibirlos
            if (allEarthquakes.length > 20) {
              // Si hay más de 20, mostrar el diálogo antes de procesarlos
              this.estimatedCount = allEarthquakes.length;
              this.showConfirmDialog = true;
              // Guardar todos los resultados para usarlos si el usuario confirma
              this.searchParams.allEarthquakes = allEarthquakes;
              console.log('Mostrando diálogo después de recibir', allEarthquakes.length, 'resultados');
              return; // No procesar aún, esperar confirmación
            }
            
            // Si son 20 o menos, procesar directamente
            this.earthquakes = allEarthquakes;
            this.procesarSismos();
          })
          .catch(error => {
            console.error('Error:', error);
            alert('Error al buscar sismos. Por favor, verifica tu conexión a internet e intenta nuevamente.');
          });
    },

    procesarSismos() {
      // Este método procesa los sismos y los muestra en el mapa
      if (!this.earthquakes || this.earthquakes.length === 0) return;

      this.earthquakes.forEach((earthquake, index) => {
              const coords = earthquake.geometry.coordinates;
              const lng = coords[0];
              const lat = coords[1];
              const mag = earthquake.properties.mag; // Obtenemos magnitud

              const popupContent = `
                <div style="text-align:center">
                  <strong>${earthquake.properties.place}</strong><br>
                  <span style="font-size: 1.2em; color: ${this.getColor(mag)}">
                    ● Magnitud: ${mag}
                  </span><br>
                  <small>${this.formatDate(earthquake.properties.time)}</small>
                </div>
              `;

              // Definimos el estilo dinámico
              const estiloCirculo = {
                radius: mag * 2.5, // Un poco más grandes para ver mejor
                fillColor: this.getColor(mag), // Color dinámico
                color: "#000", // Borde negro fino
                weight: 1,
                opacity: 1,
                fillOpacity: 0.7
              };

              // Función auxiliar para dibujar
              const dibujarSismo = (latitud, longitud) => {
               const marker = L.circleMarker([latitud, longitud], estiloCirculo)
                .bindPopup(popupContent)
                .addTo(this.map);
               this.markers[`${latitud},${longitud}`] = marker;
              };

              // Usamos setTimeout para no congelar el navegador si son muchos datos
              // pero muy rápido para que se sienta fluido
              const timeout = setTimeout(() => {
                // Dibujamos el original y los "clones" para el efecto 360 infinito
                dibujarSismo(lat, lng);
                dibujarSismo(lat, lng - 360);
                dibujarSismo(lat, lng + 360);
              }, index * 2);
              
              // Guardar el timeout para poder cancelarlo si es necesario
              this.pendingTimeouts.push(timeout); 
            });
    },

    continuarBusqueda() {
      // Cerrar el diálogo primero
      this.showConfirmDialog = false;
      
      // Si tenemos resultados guardados, usarlos; si no, ejecutar la búsqueda completa
      if (this.searchParams && this.searchParams.allEarthquakes) {
        this.earthquakes = this.searchParams.allEarthquakes;
        this.searchParams.allEarthquakes = null; // Limpiar
        this.procesarSismos();
      } else {
        this.ejecutarBusqueda();
      }
    },

    cancelarBusqueda() {
      this.showConfirmDialog = false;
      this.searchParams = null;
      this.estimatedCount = 0;
    },

    ubicarSismo(earthquake) {
      const coords = earthquake.geometry.coordinates;
      const lat = coords[1];
      const lng = coords[0];

      // Hacemos que el mapa vuele suavemente a la ubicación
      // El segundo parámetro (8) es el nivel de zoom al acercarse
      this.map.flyTo([lat, lng], 8, {
        duration: 1.5 // Duración de la animación en segundos
      });

      // Buscamos el marcador guardado y abrimos su popup
      const marker = this.markers[earthquake.id];
      if (marker) {
        marker.openPopup();
      }
    },
    formatDate(timestamp) {
      const date = new Date(timestamp);
      return date.toLocaleDateString() + ' ' + date.toLocaleTimeString();
    },
  },
};
</script>