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
            v-if="country.imageUrl"
            :src="country.imageUrl"
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
const unsplashAccessKey = 'mEY1iKva7mWFyJaistueoEACfTGjWGkGmqB5NfA20ME'; // Reemplaza con tu Access Key

const getCountryImage = async (countryName) => {
  try {
    const response = await axios.get('https://api.unsplash.com/search/photos', {
      params: {
        client_id: unsplashAccessKey,
        query: countryName,
        per_page: 1, // Solo necesitamos una imagen
      }
    });
    if (response.data.results.length > 0) {
      return response.data.results[0].urls.regular;
    } else {
      return null; // No se encontró ninguna imagen
    }
  } catch (error) {
    console.error(error);
    return null;
  }
};

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

    // Obtener la imagen para cada país
    for (const country of response.data.data.countries) {
      country.imageUrl = await getCountryImage(country.name);
    }

    countries.value = response.data.data.countries;
  } catch (error) {
    console.error(error);
  }
});
</script>
