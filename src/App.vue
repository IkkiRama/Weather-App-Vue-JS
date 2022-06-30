<!-- https://api.openweathermap.org/data/2.5/weather?q=London&units=metric&appid=eef18f70a09482a342c53291f43445c9 -->
<template>
<div id="main" :class=" state.isDay ? 'day' : 'night'">
    <div class="container py-5" style="max-width: 400px; min-width: 360px">
      <h1 class="title text-center text-white">Weather in</h1>
      <form class="search-location" v-on:submit.prevent="getWeather">
        <input
          type="text"
          class="form-control text-muted form-rounded p-4 shadow-sm"
          placeholder="What City?"
          autocomplete="off"
          v-model="state.citySearch"
          @keyup.enter="getWeather()"
        />
      </form>
      <p v-if="!state.visible" class="text-center my-3">No city found</p>
      <div
        v-else class="card rounded my-3 shadow-lg back-card overflow-hidden"
      >
        <!-- weather animation container -->
        <div>
          <div v-if="state.sunny" icon="sunny" data-label="Sunny">
            <span class="sun"></span>
          </div>

          <div v-if="state.snowy" icon="snowy" data-label="Chilly">
            <ul>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
            </ul>
          </div>

          <div v-if="state.stormy" icon="stormy" data-label="Soggy">
            <span class="cloud"></span>
            <ul>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
              <li></li>
            </ul>
          </div>
          <div v-if="state.cloudy" icon="cloudy" data-label="Perfect">
            <span class="cloud"></span>
            <span class="cloud"></span>
          </div>
          <div v-if="state.nightmoon"  icon="nightmoon" data-label="Cool!">
            <span class="moon"></span>
            <span class="meteor"></span>
          </div>
        </div>

        <!-- Top of card starts here -->
        <div class="card-top text-center" style="margin-bottom: 15rem">
          <div class="city-name my-3">
            <p>{{ state.cityName }}</p>
            <span>...</span>
            <p class="">{{ state.country }}</p>
          </div>
        </div>
        <!-- top of card ends here -->

        <!--card middle body, card body used cos I want to just update the innerHTML -->
        <div class="card-body" style="z-index: 9999999;">
          <!-- card middle starts here -->
          <div class="card-mid">
            <div class="row">
              <div class="col-12 text-center temp">
                <span>{{state.temperature}}&deg;C</span>
                <p class="my-4">{{ state.description }}</p>
              </div>
            </div>
            <div class="row">
              <div class="col d-flex justify-content-between px-5 mx-5">
                <p>
                  <img src="./assets/down.svg" alt="" />{{ state.lowTemp }}
                  &deg;C
                </p>
                <p>
                  <img src="./assets/up.svg" alt="" />{{ state.highTemp }}
                  &deg;C
                </p>
              </div>
            </div>
          </div>
          <!-- card middle ends here -->

          <!-- card bottom starts here -->
          <div class="card-bottom px-5 py-4 row">
            <div class="col text-center">
              <p>{{state.feelsLike}}&deg;C</p>
              <span>Feels like</span>
            </div>
            <div class="col text-center">
              <p>{{state.humidity}}%</p>
              <span>humidity</span>
            </div>
          </div>

          <!-- card bottom ends here -->
        </div>
      </div>
    </div>
  </div>
</template>
<script setup>
import {reactive, onMounted} from "vue"
const state = reactive({
  visible: false,
  snowy:false,
  stormy: false,
  cloudy: false,
  sunny: false,
  nightmoon : false,
  isDay: true,
  citySearch: "Purbalingga",
  cityName: "",
  country: "",
  temperature: 0,
  description: "",
  lowTemp: 0,
  highTemp: 0,
  feelsLike: 0,
  humidity: 0
})

const changeData = (result) =>{ 
  if (result.cod === 200) {
    
    state.cityName = result.name
    state.country = result.sys.country
    state.description = result.weather[0].description
    state.humidity = Math.round(result.main.humidity)
    state.temperature = Math.round(result.main.temp)
    state.lowTemp = Math.round(result.main.temp_max)
    state.highTemp = Math.round(result.main.temp_min)
    state.feelsLike = Math.round(result.main.feels_like)
    

    const timeOfDay = result.weather[0].icon
    if (timeOfDay.includes("d")) {
      state.isDay = true
    } else {
      state.isDay = false
    }

    if(result.weather[0].main.includes("Clear") && state.isDay) {

      state.snowy=false
      state.stormy=false
      state.cloudy=false
      state.sunny = true
      state.nightmoon=false

    } else if(result.weather[0].main.includes("Clouds") && !state.isDay) {
      
      state.snowy=false
      state.stormy=false
      state.cloudy=false
      state.sunny=false
      state.nightmoon = true

    }else if (result.weather[0].main.includes("Clouds")) {
      
      state.snowy=false
      state.stormy=false
      state.cloudy=true
      state.sunny=false
      state.nightmoon=false

    } else if(result.weather[0].main.includes("Snow")) {

      state.snowy = true
      state.stormy=false
      state.cloudy=false
      state.sunny=false
      state.nightmoon=false
    
    } else if(result.weather[0].main.includes("Thunderstorm")) {
      
      state.snowy=false
      state.stormy = true
      state.cloudy=false
      state.sunny=false
      state.nightmoon=false

    }

    state.visible = true
  }else{
    state.visible = false
  }
}

const getWeather = async () => {
  // event.preventDefault()
  const response = await fetch(`https://api.openweathermap.org/data/2.5/weather?q=${state.citySearch}&units=metric&appid=eef18f70a09482a342c53291f43445c9 `)
  let result = await response.json()
  changeData(result)
}

onMounted(async () => {
  const response = await fetch(`https://api.openweathermap.org/data/2.5/weather?q=${state.citySearch}&units=metric&appid=eef18f70a09482a342c53291f43445c9 `)
  let result = await response.json()
  console.log(result);
  changeData(result)
})




</script>