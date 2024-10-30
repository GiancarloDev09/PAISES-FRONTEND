<template>
  <v-container>
    <v-row cols="5" md="6">
      <v-col>
        <v-divider></v-divider>
        <!--Barra de busqueda de paises -->
        <v-row justify="center">
          <v-col cols="12" md="6">
            <div class="search-bar-wrapper">
              <v-text-field
                v-model="searchQuery"
                placeholder="Escribe el país que deseas ver"
                append-inner-icon="mdi-magnify"
                single-line
                outlined
                clearable
                class="search-bar"
                @click:append-inner="showContinents = !showContinents"
              ></v-text-field>
            </div>
          </v-col>
        </v-row>
        <!--Menu de continenstes-->
        <v-dialog v-model="showContinents" max-width="400px">
          <v-sheet class="continent-filter">
            <v-row dense>
              <v-col
                v-for="(continent, index) in continents"
                :key="index"
                cols="6"
                @click="setContinentFilter(continent)"
              >
                <v-card flat class="pa-2 text-center">
                  <v-img :src="continentImages[continent]" height="70px"></v-img>
                  <div>{{ continent }}</div>
                </v-card>
              </v-col>
              <v-col cols="12" class="text-center">
                <v-btn text @click="clearContinentFilter">Limpiar</v-btn>
              </v-col>
            </v-row>
          </v-sheet>
        </v-dialog>
      </v-col>
    </v-row>

    <v-divider></v-divider>

    <v-row>
      <v-col
        v-for="(country, index) in paginatedCountries"
        :key="country.code"
        cols="12"
        sm="6"
        md="4"
        lg="3"
      >
        <v-card class="mx-auto my-2" max-width="230" @click="selectCountry(country)">
          <v-img :src="country.imageUrl || country.flagUrl" :alt="country.name" height="90px" cover></v-img>
          <v-card-item>
            <v-card-title class="text-subtitle-1">{{ country.name }}</v-card-title>
            <v-card-subtitle class="text-caption">{{ country.continent.name }}</v-card-subtitle>
          </v-card-item>
        </v-card>
      </v-col>
    </v-row>

    <div class="text-center mt-4">
      <v-pagination v-model="currentPage" :length="Math.ceil(filteredCountries.length / itemsPerPage)" />
    </div>

    <!-- Modal para mostrar detalles del país -->
    <v-dialog v-model="dialog" max-width="600px">
      <v-card>
        <v-card-title>{{ selectedCountry.name }}</v-card-title>
        <v-card-subtitle>{{ selectedCountry.continent.name }}</v-card-subtitle>
        <v-card-text>
          <p><strong>Capital:</strong> {{ selectedCountry.capital }}</p>
          <p><strong>Idioma:</strong> {{ selectedCountry.languages.map(lang => lang.name).join(', ') }}</p>
          <p><strong>Moneda:</strong> {{ selectedCountry.currency }}</p>
          <p><strong>Estados:</strong> {{ selectedCountry.states.map(state => state.name).join(', ') }}</p>
        </v-card-text>
        <v-card-actions>
          <v-btn color="primary" text @click="dialog = false">Cerrar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script setup>
import axios from 'axios'
import { ref, onMounted, computed } from 'vue'

const countries = ref([])
const unsplashAccessKey = 'rEY13Kw7mWyJq1sUueOfAC7IGJmWq8i3HFkA02bE'
const currentPage = ref(1)
const itemsPerPage = 8
const searchQuery = ref('')
const filterContinent = ref(null)
const showContinents = ref(false)
const dialog = ref(false)
const selectedCountry = ref({})
const continentImages = {
  'Europa': 'https://www.mundoprimaria.com/wp-content/uploads/2021/08/paises-de-europa.jpg',
  'America': 'https://img.freepik.com/vector-premium/america-mapa-diferentes-colores_184151-34.jpg',
  'Asia': 'https://img.freepik.com/vector-premium/mapa-politico-asia-nombres-paises-ciudades-importantes-asia_476425-1498.jpg',
  'Oceania': 'https://img.freepik.com/vector-premium/mapa-politico-oceania-nombres-paises-ciudades-importantes-oceania_476425-1512.jpg',
  'Africa': 'https://img.freepik.com/vector-premium/mapa-politico-africa-nombres-paises-africa_476425-1506.jpg'
}



const getCountryImage = async (countryName) => {
  try {
    const response = await axios.get('https://api.unsplash.com/search/photos', {
      params: {
        client_id: unsplashAccessKey,
        query: countryName,
        per_page: 1,
      },
    })
    return response.data.results[0]?.urls.regular || null
  } catch (error) {
    console.error(error)
    return null
  }
}

const continents = computed(() => {
  const continentSet = new Set()
  countries.value.forEach((country) => continentSet.add(country.continent.name))
  return Array.from(continentSet)
})

const filteredCountries = computed(() => {
  let filtered = countries.value
  if (searchQuery.value) {
    const query = searchQuery.value.toLowerCase()
    filtered = filtered.filter((country) => country.name.toLowerCase().includes(query))
  }
  if (filterContinent.value) {
    filtered = filtered.filter((country) => country.continent.name === filterContinent.value)
  }
  return filtered
})

const paginatedCountries = computed(() => {
  const startIndex = (currentPage.value - 1) * itemsPerPage
  const endIndex = startIndex + itemsPerPage
  return filteredCountries.value.slice(startIndex, endIndex)
})

const selectCountry = (country) => {
  selectedCountry.value = country
  dialog.value = true
}

const setContinentFilter = (continent) => {
  filterContinent.value = continent
  showContinents.value = false
}

const clearContinentFilter = () => {
  filterContinent.value = null
  showContinents.value = false
}

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
            capital
            languages {
              name
            }
            currency
            states {
              name
            }
          }
        }
      `,
    })
    const countriesData = response.data.data.countries
    await Promise.all(
      countriesData.map(async (country) => {
        country.imageUrl = await getCountryImage(country.name)
        if (!country.imageUrl) {
          const flagResponse = await axios.get(`https://restcountries.com/v3.1/alpha/${country.code}`)
          country.flagUrl = flagResponse.data[0].flags.png
        }
      })
    )
    countries.value = countriesData
  } catch (error) {
    console.error(error)
  }
})
</script>

<style scoped>
.search-bar-wrapper {
  display: flex;
  justify-content: center; /* Centrar horizontalmente */
  padding: 1rem; /* Espaciado opcional alrededor */
}

.search-bar {
  max-width: 500px; /* Ancho máximo de la barra de búsqueda */
  width: 100%;
}

.continent-filter {
  padding: 8px;
  width: 100%; /* Asegura que el contenido se ajuste al ancho del diálogo */
}

.v-card {
  cursor: pointer;
}

.v-card-title {
  font-size: 1.1rem;
  font-weight: bold;
}


.search-bar {
  max-width: 500px; /* Ancho máximo de la barra de búsqueda */
  width: 100%;
}

</style>
