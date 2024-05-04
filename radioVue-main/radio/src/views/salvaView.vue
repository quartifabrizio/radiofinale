<template>
  <div class="Salva">
    <h1>Radio Salvate</h1>
    <div v-if="savedRadios && savedRadios.length === 0">
      <p>Nessuna radio salvata.</p>
    </div>
    <div v-else>
      <div class="saved-radio" v-for="(radio, index) in savedRadios" :key="index">
        <h2>{{ radio.name }}</h2>
        <img :src="radio.favicon" alt="Radio Image" width="200">
        <p>{{ radio.description }}</p>
        <p>Paese: {{ radio.country }}</p>
        <v-btn color="red" icon @click="removeSavedRadio(index)">Rimuovi</v-btn>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'SavedRadioView',
  props: {
    savedRadios: {
      type: Array,
      required: true
    }
  },
  methods: {
    removeSavedRadio(index) {
      this.$emit('remove-saved-radio', index);
    }
  }
}
</script>

<style scoped>
.saved-radio {
  margin-bottom: 20px;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 5px;
}
</style>
