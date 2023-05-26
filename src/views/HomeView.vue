<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        v-model="searchQuery"
        @input="getSearchResults"
        type="text"
        placeholder="Search for a city or state"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
      />
      <ul
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
        v-if="mapboxSearchResults"
      >
        <p v-if="searchError">Something went wrong, please try again.</p>
        <p v-if="!searchError && mapboxSearchResults.length === 0">
          No results match your query, try a different term.
        </p>
        <template v-else>
          <li
            v-for="result in mapboxSearchResults"
            :key="result.id"
            class="py-2 px-1 hover:bg-weather-primary hover:text-white cursor-pointer"
            @click="previewCity(result)"
          >
            {{ result.place_name }}
          </li>
        </template>
      </ul>
    </div>

    <div class="flex flex-col gap-4">
      <Suspense>
        <CityList />
        <template #fallback>
          <div class="flex justify-center items-center h-96">
            <i class="fa-solid fa-spinner-third animate-spin text-4xl"></i>
          </div>
        </template>
      </Suspense>
    </div>
  </main>
</template>

<script setup>
import { ref } from 'vue'
import axios from 'axios'
import { useRouter } from 'vue-router'
import CityList from '../components/CityList.vue'

const router = useRouter()
const previewCity = (result) => {
  const [city, state] = result.place_name.split(', ')
  router.push({
    name: 'cityView',
    params: {
      state: state.replace(' ', ''),
      city: city.replace(' ', ''),
    },
    query: {
      lng: result.geometry.coordinates[0],
      lat: result.geometry.coordinates[1],
      preview: true,
    },
  })
}

const mapboxAPIKey =
  'pk.eyJ1Ijoid2hpdGVoZWFkNDIxIiwiYSI6ImNsaTNvOWt1cjAwNHIzZGtiMTgxMGQ2MTgifQ.6fPM0G4pVBcuka6joEaDXQ'
const searchQuery = ref('')
const queryTimeout = ref(null)
const mapboxSearchResults = ref(null)
const searchError = ref(null)

const getSearchResults = () => {
  clearTimeout(queryTimeout.value)
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value != '') {
      try {
        const result = await axios.get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKey}&types=place`
        )
        mapboxSearchResults.value = result.data.features
      } catch {
        searchError.value = true
      }
      return
    }
    mapboxSearchResults.value = null
  }, 300)
}
</script>
