<template>
  <div>
    <Suspense>
      <AsyncCityView />

      <template #fallback>
        <CityViewSkeleton />
      </template>
    </Suspense>
  </div>
</template>

<script setup>
import AsyncCityView from '../components/AsyncCityView.vue'
import CityViewSkeleton from '../skeletons/CityViewSkeleton.vue'
</script>
