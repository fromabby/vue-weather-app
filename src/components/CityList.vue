<template>
    <div v-for="city in savedCities" :key="city.id">
        <CityCard :city="city" @click="goToCityView(city)"/>
    </div>

    <p v-if="savedCities.length === 0">
        No locations added. To start tracking a location, search in the field above.
    </p>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';
import CityCard from './CityCard.vue';
import { useRouter } from 'vue-router';

const savedCities = ref([])
const API_KEY = "662959707ea4e05997a2f396b5280518";

const getCities = async () => {
    if(localStorage.getItem('savedCities')) {
        savedCities.value = JSON.parse(localStorage.getItem('savedCities'));

        // create new request per city
        const requests = [];

        savedCities.value.forEach(({ coords: {lat,lon}}) => {
            requests.push(
                axios.get(`http://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${API_KEY}&units=metric`)
            );
        });

        const weatherData = await Promise.all(requests);

        // flicker delay
        await new Promise((res) => setTimeout(res, 1000))

        weatherData.forEach((value, index) => {
            savedCities.value[index].weather = value.data
        });
    }
};

await getCities();

const router = useRouter();

const goToCityView = (city) => {
    router.push({
        name: 'cityView',
        params: {
            city: city.city,
            state: city.state
        },
        query: {
            lat: city.coords.lat,
            lon: city.coords.lon
        }
    })
}
</script>