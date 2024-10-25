<template>
  <v-container>
    <v-row>
      <v-col
        v-for="country in countries"
        :key="country.code"
        cols="12"
        sm="6"
        md="4"
        lg="3"
      >
        <v-card>
          <v-img
            :src="country.image"
            :alt="country.name"
            height="200px"
          />
          <v-card-title>{{ country.name }}</v-card-title>
          <v-card-subtitle>{{ country.continent.name }}</v-card-subtitle>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import axios from 'axios';
import { ref, onMounted } from 'vue';

const countries = ref([]);

onMounted(async () => {
  try {
    const response = await axios.post('https://countries.trevorblades.com/', {
      query: `
        query {
          countries {
            code
            name
            continent {
              name
            }
          }
        }
      `
    });
    countries.value = response.data.data.countries;
  } catch (error) {
    console.error(error);
    // Manejar el error, por ejemplo, mostrando un mensaje al usuario
  }
});
</script>
