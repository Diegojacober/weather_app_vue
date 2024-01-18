<script setup>
import { ref } from "vue"
import axios from "axios";
import { useRouter } from "vue-router"
import CityList from "@/components/CityList.vue";
import CityCardSkeleton from "@/components/CityCardSkeleton.vue";

const router = useRouter();

const searchQuery = ref(null);
const mapboxAPIKey = "pk.eyJ1IjoiZGllZ29qYWNvYmVyIiwiYSI6ImNscmo5bTQyYTAxNHUycWxiNm9sZnF3N20ifQ.8NPV3KHcRMnFS0q0GfJBfQ";
const queryTimeout = ref(null);
const mapbosxSearchResults = ref(null);
const searchError = ref(null);

const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== "") {
      try {
        const result = await axios.get(`https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKey}&types=place`)

        mapbosxSearchResults.value = result.data.features;
      } catch {
        searchError.value = true;
      }
      return;
    }
    mapbosxSearchResults.value = null;
  }, 300)
}

const previewCity = (searchResult) => {
  const [city, state] = searchResult.place_name.split(",");
  router.push({
    name: 'cityView',
    params: {
      state: state.replace(" ", ""),
      city: city,
    },
    query: {
      lat: searchResult.geometry.coordinates[1],
      lng: searchResult.geometry.coordinates[0],
      preview: true
    }
  })
}
</script>

<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
        type="text" placeholder="Search for city or state" v-model="searchQuery" @input="getSearchResults">
      <ul class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
        v-if="mapbosxSearchResults">
        <p v-if="searchError">Sorry, something went wrong, please try again.</p>
        <p v-if="!searchError && mapbosxSearchResults.length === 0">No results match your query, try a different term.</p>
        <template v-else>
          <li v-for="searchResult in mapbosxSearchResults" :key="searchResult.id" class="oy-2 cursor-pointer"
            @click="previewCity(searchResult)">
            {{ searchResult.place_name }}
          </li>
        </template>
      </ul>
    </div>
    <div class="flex flex-col gap-4">
      <Suspense>
        <CityList/>
        <template #fallback>
          <CityCardSkeleton/>
        </template>
      </Suspense>
    </div>
  </main>
</template>
