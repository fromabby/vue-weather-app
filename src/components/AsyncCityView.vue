<script setup>
    import axios from 'axios';
    import { useRoute } from 'vue-router';

    const API_KEY = import.meta.env.VITE_OPENWEATHER_API_KEY;
    
    const route = useRoute();
    const { lat, lon, preview } = route.query;
    const { state, city } = route.params;

    const getWeatherData = async () => {
        try {
            const { data } = await axios.get(`https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=${API_KEY}&units=metric`)

            // // calculate current date and time
            // const localOffset = new Date().getTimezoneOffset() * 60000;
            // const utc = data.current.dt * 1000 + localOffset;

            // data.currentTime = utc + 1000 * data.timezone_offset;

            // // calculate hourly weather offset
            // data.hourly.forEach(hour => {
            //     const utc = hour.dt * 1000 + localOffset;
            //     hour.currentTime = utc + 1000 * data.timezone_offset;
            // })

            return data;
        } catch (error) {
            console.log(error)
        }
    }

    const weatherData = await getWeatherData();
    console.log(weatherData)

    const { sys, main, weather, name } = weatherData;
    const { feels_like, temp } = main;
</script>

<template>
    <div class="flex flex-col items-center flex-1">
        <!-- Banner -->
        <div 
            v-if="preview"
            class="w-full p-4 text-center text-white bg-weather-secondary"
        >
            <p>You are currently previewing this city, click that "+" icon to start tracking this city.</p>
        </div>
        <!-- Weather overview -->
        <div class="flex flex-col items-center py-12 text-white">
            <h1 class="mb-2 text-4xl">
                {{ city }}, {{ sys.country }}
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
                <h2 class="mb-4">Hourly weather</h2>
                <!-- No hourly forecasts included in api response -->
            </div>
        </div>
    </div>
</template>
