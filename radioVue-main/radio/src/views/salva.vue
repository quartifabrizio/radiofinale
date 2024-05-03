<template>
    <div>
      <h1>Radio Salvate</h1>
      <div class="radio-cards">
        <v-col cols="12" sm="6" md="4" v-for="(radio, index) in savedRadios" :key="index">
          <v-card class="radio-card">
            <h2>{{ radio.name }}</h2>
            <img :src="radio.favicon" alt="Radio Image" width="200">
            <p>{{ radio.description }}</p>
            <p>Paese: {{ radio.country }}</p>
            <!-- Aggiungi il bottone per eliminare la radio salvata -->
            <v-btn color="red" icon @click="removeSavedRadio(index)">
              <v-icon>mdi-delete</v-icon>
            </v-btn>
          </v-card>
        </v-col>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        savedRadios: [], // Array per le radio salvate
      };
    },
    methods: {
      // Metodo per rimuovere una radio salvata
      removeSavedRadio(index) {
        this.savedRadios.splice(index, 1);
        // Puoi anche aggiungere qui la logica per rimuovere la radio dal localStorage o da un altro mezzo di persistenza
      },
    },
    created() {
      // Recupera le radio salvate all'avvio del componente (puoi recuperarle da localStorage o da un altro mezzo di persistenza)
      this.savedRadios = JSON.parse(localStorage.getItem('savedRadios')) || [];
    },
  };
  </script>
  
  <style scoped>
  /* Stili per i tuoi componenti */
  </style>