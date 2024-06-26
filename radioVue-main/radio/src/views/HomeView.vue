
<template>
  <div>
    <router-view />
    <v-text-field v-model="searchText" label="Cerca radio" outlined></v-text-field>

    <!-- Tabella per visualizzare le cards -->
    <table>
      <tbody>
        <tr v-for="(row, rowIndex) in chunkedRadios" :key="rowIndex">
          <td v-for="(radio, cellIndex) in row" :key="cellIndex">
            <v-card color="white" class="small-card"> <!-- Applichiamo la classe 'small-card' -->
              <div class="d-flex flex-no-wrap justify-space-between">
                <div>
                  <v-card-title class="text-h6"> <!-- Riduciamo la dimensione del titolo -->
                    {{ radio.name }}
                  </v-card-title>
                  <v-card-subtitle class="text-caption"> <!-- Riduciamo la dimensione del sottotitolo -->
                    {{ radio.country }}"
                  </v-card-subtitle>
                  <v-card-actions>

                    <v-btn @click="playAudio(radio.url)" class="ms-2" icon="mdi-play" variant="text"
                      size="small"></v-btn>
                    <div ref="player"></div>

                    <v-btn @click="saveRadio(radio)" variant="text" icon="mdi-heart" size="small" class="ms-2"></v-btn>

                    <v-btn class="ms-2" icon="mdi-stop" size="small" variant="text" @click="stopAudio"></v-btn>

                  </v-card-actions>
                </div>
                <v-avatar class="ma-3" rounded="0" size="100"> <!-- Riduciamo la dimensione dell'avatar -->
                  <v-img :src="getRadioImage(radio)" width="100"></v-img> <!-- Riduciamo la dimensione dell'immagine -->
                </v-avatar>
              </div>
            </v-card>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import Hls from 'hls.js';
export default {
  name: 'HomeView',
  data() {
    return {
      radios: [],
      savedRadios: JSON.parse(localStorage.getItem('savedRadios')) || [],
      searchText: '',
      currentAudio: new Audio(), // Initialize audio element
      currentPlayingRadio: null, // Initialize currentPlayingRadio
      hls: null // Initialize HLS instance
    };
  },

  methods: {
    getRadios() {
  fetch('https://nl1.api.radio-browser.info/json/stations/search?limit=100&countrycode=IT&hidebroken=true&order=clickcount&reverse=true')
    .then(response => response.json())
    .then(data => {
      this.radios = data;
      console.log('Radios:', this.radios); // Aggiungi questo console.log per visualizzare tutte le informazioni delle radio
    });
},

    playAudio(url, radio) {
  // Initialize this.audio if it's null
  if (!this.currentAudio) {
    this.currentAudio = new Audio();
  }

  if (this.currentPlayingRadio && this.currentPlayingRadio !== radio) {
    this.currentPlayingRadio.isPlaying = false;
    if (this.hls) {
      this.hls.destroy();
      this.hls = null;
    }
    this.currentAudio.pause();
  }

  if (url.endsWith('.m3u8')) {
    if (Hls.isSupported()) {
      this.hls = new Hls();
      this.hls.loadSource(url);
      this.hls.attachMedia(this.currentAudio);
      this.hls.on(Hls.Events.MANIFEST_PARSED, () => {
        this.currentAudio.play();
      });
    } else if (this.currentAudio.canPlayType('application/vnd.apple.mpegurl')) {
      this.currentAudio.src = url;
      this.currentAudio.play();
    }
  } else {
    this.currentAudio.src = url;
    this.currentAudio.play();
  }

  this.currentPlayingRadio = radio;
},
    togglePlayback(radio) {
      if (radio.isPlaying) {
        this.audio.pause();
        if (this.hls) {
          this.hls.stopLoad();
        }
      } else {
        this.playAudio(radio.url, radio);
      }
      radio.isPlaying = !radio.isPlaying;
    },
    toggleFavorite(radio) {
      radio.isFavorite = !radio.isFavorite;
      this.saveToLocalStorage();
    },
    getRadioImage(radio) {
      // Verifica se radio.favicon è vuoto o contiene solo spazi bianchi
      if (!radio.favicon || radio.favicon.trim() === '') {
        // Restituisci l'URL dell'immagine predefinita
        return 'https://img.freepik.com/vetores-premium/icone-ou-simbolo-vintage-da-estacao-de-radio-online_8071-25787.jpg';
      } else {
        // Restituisci l'URL di radio.favicon
        return radio.favicon;
      }
    },
    stopAudio() {
      if (this.currentAudio) {
        this.currentAudio.pause();
      }
    },
    saveRadio(radio) {
      const alreadySaved = this.savedRadios.some(savedRadio => savedRadio.changeuuid === radio.changeuuid);
      if (!alreadySaved) {
        this.savedRadios.push(radio);
        localStorage.setItem('savedRadios', JSON.stringify(this.savedRadios));
      }
    },
    chunkArray(array, size) {
      return array.reduce((chunks, item, i) => {
        if (i % size === 0) {
          chunks.push([item]);
        } else {
          chunks[chunks.length - 1].push(item);
        }
        return chunks;
      }, []);
    },
  },

  computed: {
    filteredRadios() {
      return this.radios.filter(radio => {
        return radio.name.toLowerCase().includes(this.searchText.toLowerCase());
      });
    },
    chunkedRadios() {
      return this.chunkArray(this.filteredRadios, 4);
    }
  },
  created() {
    this.getRadios();
  },
}
</script>
<style scoped>
.small-card {
  width: 100%;
  /* Ogni card occupa tutta la larghezza della cella */
  margin-bottom: 7px;
}

.card-with-background {
  width: 100%;
  margin-bottom: 7px;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

nav {
  padding: 30px;
}

nav a {
  font-weight: bold;
  color: #2c3e50;
}

nav a.router-link-exact-active {
  color: #42b983;
}

.sound-wave {
  display: flex;
  align-items: center;
  height: 20px;
  margin-left: 10px;
  /* Adjust this if necessary */
  margin-top: 100px;
  /* Added to push the sound wave down */
}

.bar {
  width: 4px;
  height: 100%;
  margin: 0 2px;
  background-color: #333;
  animation: pulse 0.8s infinite ease-in-out alternate;
}

.bar:nth-child(1) {
  animation-delay: 0s;
}

.bar:nth-child(2) {
  animation-delay: 0.1s;
}

.bar:nth-child(3) {
  animation-delay: 0.2s;
}

.bar:nth-child(4) {
  animation-delay: 0.3s;
}
.play-gif {
  width: 30px; /* Imposta la larghezza desiderata */
  height: 30px; /* Imposta l'altezza desiderata */
}


table {
  width: 100%;
  /* Rendi la tabella larga quanto il display */
  table-layout: fixed;
  /* Fissa la larghezza delle colonne */
}

td {
  padding: 8px;
  /* Aggiunge spazio interno alle celle */
  box-sizing: border-box;
  /* Assicura che il padding non influenzi la larghezza totale della cella */
}

@media screen and (max-width: 768px) {
  .v-card__title, .v-card__subtitle {
    font-size: 0.9rem; /* Riduci la dimensione del titolo e del sottotitolo sui dispositivi mobili */

    .small-card {
  margin-bottom: 15px;
}

.v-card__title, .v-card__subtitle {
  font-size: 1rem; /* Riduci la dimensione del titolo e del sottotitolo */
}

.v-btn {
  min-width: 36px;
  min-height: 36px;
}

.v-img {
  object-fit: cover; /* Assicura che le immagini si adattino al contenitore */
}

/* Aggiungi margini per separare le cards */
.v-card {
  margin-right: 10px;
  margin-bottom: 10px;
}

/* Imposta un padding per il container per spaziare le cards */
.v-container {
  padding: 10px;
}

/* Imposta il display inline per il titolo della card */
.v-card__title {
  display: inline;
}
  }}

</style>