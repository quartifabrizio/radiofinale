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
                    {{ radio.country }}
                  </v-card-subtitle>
                  <v-card-actions>
                    <v-btn @click="playAudio(radio.url)" class="ms-2" icon="mdi-play" variant="text" size="small" ></v-btn>
                    <v-btn  variant="text" icon="mdi-heart" size="small" @click="saveRadio(radio)" class="ms-2"></v-btn>
                    <v-btn  class="ms-2" icon="mdi-stop" size="small" variant="text" @click="stopAudio"></v-btn>
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
export default {
  name: 'HomeView',
  data() {
    return {
      radios: [],
      savedRadios: [],
      searchText: '',
      currentAudio: null,
    }
  },
  methods: {
    getRadios() {
      fetch('https://nl1.api.radio-browser.info/json/stations/search?limit=100&countrycode=IT&hidebroken=true&order=clickcount&reverse=true')
        .then(response => response.json())
        .then(data => {
          this.radios = data;
        });
    },
    playAudio(audioUrl) {
      if (this.currentAudio) {
        this.currentAudio.pause();
      }
      const audio = new Audio(audioUrl);
      audio.play();
      this.currentAudio = audio;
    },
    getRadioImage(radio) {
  // Verifica se radio.favicon è vuoto o contiene solo spazi bianchi
  if (!radio.favicon || radio.favicon.trim() === '') {
    // Restituisci l'URL dell'immagine predefinita
    return 'data:image/png;base64,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';
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
      if (!this.savedRadios.includes(radio)) {
        this.savedRadios.push(radio);
        this.$emit('radio-saved', radio); // Emit event
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
    }
  },
  computed: {
    filteredRadios() {
      return this.radios.filter(radio => {
        return radio.name.toLowerCase().includes(this.searchText.toLowerCase());
      });
    },
    chunkedRadios() {
      return this.chunkArray(this.filteredRadios, 3);
    }
  },
  created() {
    this.getRadios();
  },
}
</script>

<style scoped>

.small-card {
  width: 493px;
  margin-bottom: 7px;
  margin-left:8px;
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


</style>
