<script setup>
    import axios from 'axios';
    import { useRoute, useRouter } from 'vue-router';

    const API_KEY = import.meta.env.VITE_OPENWEATHER_API_KEY;
    
    const route = useRoute();
    const { lat, lon } = route.query;
    const { city } = route.params;

    const getWeatherData = async () => {
        try {
            const { data } = await axios.get(`https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${API_KEY}&units=metric`)

            // calculate current date and time
            const localOffset = new Date().getTimezoneOffset() * 60000;
            const utc = data.dt * 1000 + localOffset;

            data.sys.sunrise = utc + 1000 * data.sys.sunrise;
            data.sys.sunset = utc + 1000 * data.sys.sunset;

            return data;
        } catch (error) {
            console.log(error)
        }
    }

    // flicker delay
    await new Promise((res) => setTimeout(res, 1000))
    
    const weatherData = await getWeatherData();

    const { sys, main, weather, name, clouds, wind } = weatherData;
    const { feels_like, temp, temp_max, temp_min } = main;
    const { country, sunrise, sunset } = sys;

    // remove city code
    const router = useRouter()

    const removeCity = () => {
        const cities = JSON.parse(localStorage.getItem('savedCities'))

        const savedCities = cities.filter(city => city.coords.lat !== lat && city.coords.lon !== lon)

        localStorage.setItem('savedCities', JSON.stringify(savedCities))

        let query = Object.assign({}, route.query);
        router.replace({ query: {
            ...query, preview: true
        } })
    }
</script>

<template>
    <div class="flex flex-col items-center flex-1">
        <!-- Banner -->
        <div 
            v-if="route.query.preview"
            class="w-full p-4 text-center text-white bg-weather-secondary"
        >
            <p>You are currently previewing this city, click that "+" icon to start tracking this city.</p>
        </div>
        <!-- Weather overview -->
        <div class="flex flex-col items-center py-12 text-white">
            <h1 class="mb-2 text-4xl">
                {{ city }}, {{ country }}
            </h1>
            <h5>{{ name }}</h5>
            <p class="mb-12 text-sm">
                {{ new Date().toLocaleDateString("en-us", {
                    weekday: "short",
                    day: "2-digit",
                    month: "long"
                })}}

                {{ new Date().toLocaleTimeString("en-us", {
                    timeStyle: "short"
                })}}
            </p>
            <p class="mb-8 text-8xl">
                {{ Math.round(temp) }}&deg;
            </p>
            <p>Feels like {{ Math.round(feels_like) }}&deg;</p>
            <p class="capitalize"> {{weather[0].description }} </p>
            <img :src="`http://openweathermap.org/img/wn/${weather[0].icon}@2x.png`" alt="" class="w-[150px] h-auto">
        </div>

        <hr class="w-full border border-white border-opacity-10" />

        <!-- Hourly weather -->
        <div class="w-full max-w-screen-md py-12">
            <div class="mx-8 text-white">
                <h2 class="mb-4 text-xl">Additional information</h2>
                <div class="flex items-center mb-6">
                    <p class="flex-1">
                        Sunrise:
                    </p>
                    <p class="flex-1">
                        {{ new Date(sunrise).toLocaleTimeString("en-us", { timeStyle: "short" })}}
                    </p>
                    <div class="flex justify-end flex-1 gap-2">
                        <p>H: {{ Math.round(temp_max) }}&deg;</p>
                    </div>
                </div>
                <div class="flex items-center mb-6">
                    <p class="flex-1">
                        Sunset:
                    </p>
                    <p class="flex-1">
                        {{ new Date(sunset).toLocaleTimeString("en-us", { timeStyle: "short" })}}
                    </p>
                    <div class="flex justify-end flex-1 gap-2">
                        <p>L: {{ Math.round(temp_min) }}&deg;</p>
                    </div>
                </div>
                <div class="flex items-center mb-6">
                    <p class="flex-1">
                        Clouds:
                    </p>
                    <p class="flex-1">
                        {{ clouds.all }}%
                    </p>
                    <div class="flex justify-end flex-1 gap-2">
                        <p>Wind: {{ wind.speed }} kph</p>
                    </div>
                </div>
            </div>
        </div>

        <div
            class="flex items-center gap-2 py-12 text-white duration-150 cursor-pointer hover:text-red-500"
            @click="removeCity"
            v-if="!route.query.preview"
        >
            <i class="fa-solid fa-trash"></i>
            <p>Remove City</p>
        </div>
    </div>
</template>
