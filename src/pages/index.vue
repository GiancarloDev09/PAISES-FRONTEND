<template>
  <v-container>
    <v-row cols="5" md="6">
      <v-col>
        <v-text-field
          v-model="searchQuery"
          label="Buscar país"
          prepend-icon="mdi-magnify"
          clearable
          @click:prepend="showContinents = !showContinents"
          density="compact"
        >
          <template v-slot:prepend-inner>
            <v-list v-if="showContinents" density="compact">
              <v-list-item
                v-for="continent in continents"
                :key="continent"
                @click="filterContinent = continent; showContinents = false"
              >
                {{ continent }}
              </v-list-item>
              <v-list-item @click="filterContinent = null; showContinents = false">
                Limpiar
              </v-list-item>
            </v-list>
          </template>
        </v-text-field>
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
        <v-card class="mx-auto my-2" max-width="250" @click="selectCountry(country)">
          <v-img :src="country.imageUrl || country.flagUrl" :alt="country.name" height="100px" cover></v-img>
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
const unsplashAccessKey = 'rEY13Kw7mWyJq1sUueOfAC7IGJmWq8i3HFkA02bE' // Reemplaza con tu Access Key
const currentPage = ref(1)
const itemsPerPage = 8
const searchQuery = ref('')
const filterContinent = ref(null)
const showContinents = ref(false)
const dialog = ref(false)
const selectedCountry = ref({})

const getCountryImage = async (countryName) => {
  try {
    const response = await axios.get('https://api.unsplash.com/search/photos', {
      params: {
        client_id: unsplashAccessKey,
        query: countryName,
        per_page: 1,
      },
    })
    if (response.data.results.length > 0) {
      return response.data.results[0].urls.regular
    } else {
      return null
    }
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

    // Obtener imágenes y banderas en paralelo
    await Promise.all(
      countriesData.map(async (country) => {
        country.imageUrl = await getCountryImage(country.name)
        // Si no hay imagen de Unsplash, usar la bandera
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
