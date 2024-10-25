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
            v-if="country.image"
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
const pixabayApiKey = '46718433-3e884c781f723974885da7eda'; // Reemplaza con tu clave API

const getCountryImage = async (countryName) => {
  try {
    const response = await axios.get('https://pixabay.com/api/', {
      params: {
        key: pixabayApiKey,
        q: countryName,
        image_type: 'photo',
        per_page: 1, // Solo necesitamos una imagen
      }
    });
    if (response.data.hits.length > 0) {
      return response.data.hits[0].webformatURL;
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
      country.image = await getCountryImage(country.name);
    }

    countries.value = response.data.data.countries;
  } catch (error) {
    console.error(error);
  }
});
</script>
