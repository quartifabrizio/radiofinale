<template>
  <div>
    <nav>
      <router-link to="/">Home</router-link> |
      <router-link to="/about">About</router-link> |
      <router-link to="/contatti">Contatti</router-link>
    </nav>
    <router-view/>

    <div class="radio-cards">
      <div class="radio-card" v-for="(radio, index) in radios" :key="index">
        <h2>{{ radio.name }}</h2>
        <img :src="radio.favicon" alt="Radio Image" width="200">
        <p>{{ radio.description }}</p>
        <p>Paese: {{ radio.country }}</p>
        <button @click="playAudio(radio.url)">Play</button>
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
      const audio = new Audio(audioUrl);
      audio.play();
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
