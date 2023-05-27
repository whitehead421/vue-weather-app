<template>
  <div v-for="city in savedCities" :key="city.id">
    <CityCard :city="city" @click="goToCityView(city)" />
  </div>

  <p v-if="savedCities.length === 0">
    No locations added. To start tracking a location, search in the field above.
  </p>
</template>

<script setup>
import axios from 'axios'
import { ref } from 'vue'
import CityCard from './CityCard.vue'
import { useRouter } from 'vue-router'
const openweatherAPIKey = import.meta.env.VITE_OPENWEATHER_API_KEY

const savedCities = ref([])
const getCities = async () => {
  if (localStorage.getItem('savedCities')) {
    savedCities.value = JSON.parse(localStorage.getItem('savedCities'))

    const requests = []

    savedCities.value.forEach((city) => {
      requests.push(
        axios.get(
          `https://api.openweathermap.org/data/2.5/weather?lat=${city.coords.lat}&lon=${city.coords.lng}&appid=${openweatherAPIKey}&units=metric`
        )
      )
    })

    const weatherData = await Promise.all(requests)

    await new Promise((resolve) => setTimeout(resolve, 1000))

    weatherData.forEach((city, index) => {
      savedCities.value[index].weather = city.data
    })
  }
}

await getCities()

const router = useRouter()
const goToCityView = (city) => {
  router.push({
    name: 'cityView',
    params: {
      state: city.state.replace(' ', ''),
      city: city.city.replace(' ', ''),
    },
    query: {
      id: city.id,
      lng: city.coords.lng,
      lat: city.coords.lat,
    },
  })
}
</script>
