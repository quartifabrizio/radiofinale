<template>
  <div class="Salva">
    <h1>Radio Salvate</h1>
    <div v-if="savedRadios && savedRadios.length === 0">
      <p>Nessuna radio salvata.</p>
    </div>
    <div v-else>
      <table>
        <tbody>
          <tr>
            <td v-for="(radio, index) in savedRadios" :key="index" style="width: 25%;">
              <v-card color="white" class="small-card">
                <div class="d-flex flex-no-wrap justify-space-between">
                  <div>
                    <v-card-title class="text-h6">
                      {{ radio.name }}
                    </v-card-title>
                    <v-card-subtitle class="text-caption">
                      {{ radio.country }}
                    </v-card-subtitle>
                    <v-card-actions>

                      <v-btn @click="playSavedRadio(radio.url, radio)" class="ms-2" size="small" icon="mdi-play">
                      </v-btn>

                      <v-btn class="ms-2" icon="mdi-stop" size="small" variant="text" @click="pauseAudio">
                      </v-btn>

                      <v-btn icon="mdi-delete" color="red" @click="removeSavedRadio(index)" class="ms-2" size="small">
                      </v-btn>
                      
                    </v-card-actions>
                  </div>
                  <v-avatar class="ma-3" rounded="0" size="100">
                    <v-img :src="radio.favicon" width="100"></v-img>
                  </v-avatar>
                </div>
              </v-card>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import Hls from 'hls.js';

export default {
  name: 'SavedRadioView',
  data() {
    return {
      savedRadios: JSON.parse(localStorage.getItem('savedRadios')) || [],
      currentAudio: null, // Aggiunto per tenere traccia dell'audio attualmente in riproduzione
      currentPlayingRadio: null, // Aggiunto per tenere traccia della radio attualmente in riproduzione
      hls: null // Aggiunto per tenere traccia dell'istanza Hls
    };
  },
  methods: {
    removeSavedRadio(index) {
      this.savedRadios.splice(index, 1);
      localStorage.setItem('savedRadios', JSON.stringify(this.savedRadios));
    },
    playSavedRadio(url, radio) {
      // Inizializza this.currentAudio se è null
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
    pauseAudio() {
      if (this.currentAudio) {
        this.currentAudio.pause();
      }
    }
  }
}
</script>

<style scoped>
.small-card {
  width: 100%;
  /* Each card occupies full width of the cell */
  margin-bottom: 7px;
}

table {
  width: 100%;
  /* Make the table as wide as the display */
  table-layout: fixed;
  /* Fix the column widths */
}

td {
  padding: 8px;
  /* Add padding inside cells */
  box-sizing: border-box;
  /* Ensure padding doesn't affect total cell width */
}
</style>
