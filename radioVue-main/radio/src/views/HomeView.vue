<template>
  <div>
    <router-view />
    <v-text-field v-model="searchText" label="Cerca radio" outlined></v-text-field>

    <div class="radio-cards">
      <div class="radio-card" v-for="(radio, index) in filteredRadios" :key="index">
        <h2>{{ radio.name }}</h2>
        <img :src="radio.favicon" alt="Radio Image" width="200">
        <p>{{ radio.description }}</p>
        <p>Paese: {{ radio.country }}</p>

        <v-row>
          <v-col cols="auto">
            <v-btn block elevation="4" color="green" dark @click="playAudio(radio.url)">play</v-btn>
          </v-col>

          <v-col cols="auto">
            <v-btn color="red" icon="mdi-heart" size="small" @click="saveRadio(radio)"></v-btn>
          </v-col>
        </v-row>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'HomeView',
  data() {
    return {
      radios: [],
      savedRadios: [], // Array per tenere traccia delle radio salvate
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
    saveRadio(radio) {
      if (!this.savedRadios.includes(radio)) {
        this.savedRadios.push(radio);
        this.$emit('radio-saved', radio); // Emit event
      }
    }
  },
  computed: {
    filteredRadios() {
      return this.radios.filter(radio => {
        return radio.name.toLowerCase().includes(this.searchText.toLowerCase());
      });
    }
  },
  created() {
    this.getRadios();
  },
}

</script>

<style scoped>
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

.radio-cards {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  margin-top: 20px;
}

.radio-card {
  width: 300px;
  margin: 10px;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 5px;
}
</style>
