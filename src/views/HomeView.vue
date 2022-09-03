<template>
  <main class="container text-white">
    <div class="relative pt-4 mb-8">
      <input 
        v-model="searchQuery"
        type="text" 
        placeholder="Search for a city or state" 
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow-[0px_1px_0_0_#004E71]"
        @input="getSearchResults"
      />
      <ul 
        v-if="searchResults"
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
      >
        <p v-if="searchError">Sorry, something went wrong, please try again.</p>
        <p v-if="!searchError && searchResults.length === 0">No results match your query, try a different term.</p>
        <template v-else>
          <li
            v-for="searchResult in searchResults"
            :key="`${searchResult.lon}${searchResult.lat}`"
            class="py-2 cursor-pointer"
            @click="previewCity(searchResult)"
          >
            {{`${searchResult.name}, `}}{{searchResult.state ? `${searchResult.state}, ` : ''}}{{searchResult.country}}
          </li>
        </template>
      </ul>
    </div>
  </main>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';
import { useRouter } from 'vue-router';

const API_KEY = import.meta.env.VITE_OPENWEATHER_API_KEY;
const limit = 5;
const searchQuery = ref('');
const queryTimeout = ref('');

const searchResults = ref(null);
const searchError = ref(null);

const router = useRouter();
const previewCity = (searchResult) => {
    const { name: city, lat, lon } = searchResult;
    const state = searchResult.state || city;

    router.push({
        name: "cityView",
        params: { state, city },
        query: { lat, lon, preview: true }
    })
  }
  
const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if(searchQuery.value !== '') {
      try {
        searchError.value = false;
        // send api request
        const { data } = await axios.get(`http://api.openweathermap.org/geo/1.0/direct?q=${searchQuery.value}&limit=${limit}&appid=${API_KEY}`);
        
        searchResults.value = data
      } catch {
        searchError.value = true;
      }
      return;
    }
    searchResults.value = null;
  }, 300);

};
</script>