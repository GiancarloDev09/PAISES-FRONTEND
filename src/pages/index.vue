<template>
  <v-container>
    <v-row>
      <v-col
        v-for="(country, index) in paginatedCountries"
        :key="country.code"
        cols="12"
        sm="6"
        md="4"
        lg="3"
      >
        <v-card class="mx-auto my-2" max-width="250">
          <v-img
            :src="country.imageUrl || country.flagUrl"
            :alt="country.name"
            height="140px"
            cover
          ></v-img>

          <v-card-item>
            <v-card-title class="text-subtitle-1">{{ country.name }}</v-card-title>
            <v-card-subtitle class="text-caption">{{ country.continent.name }}</v-card-subtitle>
          </v-card-item>
        </v-card>
      </v-col>
    </v-row>
    <div class="text-center mt-4">
      <v-pagination v-model="currentPage" :length="Math.ceil(countries.length / itemsPerPage)" />
    </div>
  </v-container>
</template>

<script setup>
import axios from 'axios';
import { ref, onMounted } from 'vue';

const countries = ref([]);
const unsplashAccessKey = 'rEY13Kw7mWyJq1sUueOfAC7IGJmWq8i3HFkA02bE'; // Reemplaza con tu Access Key
const currentPage = ref(1);
const itemsPerPage = 6; // Mostrar 6 items por página (2 filas x 3 columnas)


const getCountryImage = async (countryName) => {
  try {
    const response = await axios.get('https://api.unsplash.com/search/photos', {
      params: {
        client_id: unsplashAccessKey,
        query: countryName,
        per_page: 1,
      }
    });
    if (response.data.results.length > 0) {
      return response.data.results[0].urls.regular;
    } else {
      return null;
    }
  } catch (error) {
    console.error(error);
    return null;
  }
};
const paginatedCountries = computed(() => {
  const startIndex = (currentPage.value - 1) * itemsPerPage;
  const endIndex = startIndex + itemsPerPage;
  return countries.value.slice(startIndex, endIndex);
});
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
            emoji
          }
        }
      `
    });

    const countriesData = response.data.data.countries;

    // Obtener imágenes y banderas en paralelo
    await Promise.all(countriesData.map(async (country) => {
      country.imageUrl = await getCountryImage(country.name);
      // Si no hay imagen de Unsplash, usar la bandera
      if (!country.imageUrl) {
        const flagResponse = await axios.get(`https://restcountries.com/v3.1/alpha/${country.code}`);
        country.flagUrl = flagResponse.data[0].flags.png;
      }
    }));

    countries.value = countriesData;

  } catch (error) {
    console.error(error);
  }
});
</script>
