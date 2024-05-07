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

                      <v-btn @click="playSavedRadio(radio.url)" class="ms-2" size="small" icon="mdi-play">
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
export default {
  name: 'SavedRadioView',
  data() {
    return {
      savedRadios: JSON.parse(localStorage.getItem('savedRadios')) || [],
      currentAudio: null // Added to keep track of the currently playing audio
    };
  },
  methods: {
    removeSavedRadio(index) {
      this.savedRadios.splice(index, 1);
      localStorage.setItem('savedRadios', JSON.stringify(this.savedRadios));
    },
    playSavedRadio(audioUrl) {
      // Pause current audio if any
      if (this.currentAudio) {
        this.currentAudio.pause();
      }

      const audio = new Audio(audioUrl);
      audio.play();
      this.currentAudio = audio;
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

.sound-wave {
  display: flex;
  align-items: center;
  height: 20px;
  margin-left: 10px;
  margin-top: 100px;
  /* Adjust if necessary */
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

.bar:nth .small-card {
  width: 100%;
  /* Each card occupies full width of the cell */
  margin-bottom: 7px;
}

.sound-wave {
  display: flex;
  align-items: center;
  height: 20px;
  margin-left: 10px;
  margin-top: 100px;
  /* Adjust if necessary */
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