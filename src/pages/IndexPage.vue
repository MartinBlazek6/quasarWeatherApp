<template>
  <q-page class="flex column" :class="bgClass">
    <q-inner-loading :showing="searching" color="white" />
    <section class="col q-pt-lg q-px-md">
      <q-input
        v-model="searchText"
        borderless
        dark
        placeholder="Search"
        @keyup.enter="getWeatherBySearch"
      >
        <template v-slot:before>
          <q-icon name="my_location" @click="getLocation" />
        </template>
        <template v-slot:append>
          <q-btn round dense flat icon="search" @click="getWeatherBySearch" />
        </template>
      </q-input>
    </section>
    <section class="col q-px-md">
      <q-select
        v-model="forecastDays"
        :options="forecastOptions"
        dark
        label="Forecast Days"
        outlined
        emit-value
        map-options
        @input="getWeatherBySearch"
      />
    </section>
    <template v-if="weatherData">
      <section v-for="(day, index) in weatherData.list" :key="index" class="col text-white text-center">
        <div class="text-h4 text-weight-light">{{ weatherData.city.name }}</div>
        <div class="text-h6 text-weight-light">
          {{ day.weather[0].main }}
        </div>
        <div class="text-h1 text-weight-thin q-my-lg relative-position">
          <span>{{ day.temp.day }}</span>
          <span class="text-h4 relative-position degree">&deg;C</span>
        </div>
        <section class="col text-center">
          <q-img :src="weatherImage(day.weather[0].icon)" alt="Weather Image" width="100px" />
        </section>
      </section>
    </template>
    <template v-else>
      <section class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">Quasar<br />Weather</div>
        <q-btn class="col" flat v-on:click="getLocation">
          <q-icon left size="3em" name="my_location" />
          <div>Find My Location</div>
        </q-btn>
      </section>
    </template>
    <section class="col skyline"></section>
  </q-page>
</template>

<script setup lang="ts">
import { computed, inject, ref } from 'vue';
import { useQuasar, getCssVar } from 'quasar';

const $q = useQuasar();
const axios = inject('axios');

const searching = ref(false);
const searchText = ref('');
const weatherData = ref(null);
const forecastDays = ref(1);
const forecastOptions = [
  { label: '1 Day', value: 1 },
  { label: '3 Days', value: 3 },
  { label: '5 Days', value: 5 },
];

const temp = computed(() => {
  return Math.round(weatherData.value.list[0].temp.day);
});

const weatherImage = (icon) => {
  const weatherApiBaseUrl = 'http://openweathermap.org/img/wn';
  return `${weatherApiBaseUrl}/${icon}@2x.png`;
};

const bgClass = computed(() => {
  if (!weatherData.value) return null;

  if (weatherData.value.list[0].weather[0].icon.endsWith('n')) {
    return 'bg-night';
  } else {
    return 'bg-day';
  }
});

const getLocation = async () => {
  searching.value = true;
  try {
    if ($q.platform.is.electron) {
      // Electron specific code
    } else {
      navigator.geolocation.getCurrentPosition((position) => {
        const lat = position.coords.latitude;
        const lon = position.coords.longitude;
        getWeatherByCoords(lat, lon);
        searching.value = false;
      });
    }
  } catch (error) {}
};

const getWeatherBySearch = async () => {
  try {
    const endpoint = `https://api.openweathermap.org/data/2.5/forecast/daily?q=${searchText.value}&cnt=${forecastDays.value}&appid=33aa634c216259f797f35e862f073b40&units=metric`;
    const response = await axios.get(endpoint);
    weatherData.value = response.data;
    console.log('weatherData', weatherData.value);
  } catch (error) {
    console.error('getWeatherBySearch', error.message);
    weatherData.value = null;
  }
};

const getWeatherByCoords = async (lat, lon) => {
  try {
    const endpoint = `https://api.openweathermap.org/data/2.5/forecast/daily?lat=${lat}&lon=${lon}&cnt=${forecastDays.value}&appid=33aa634c216259f797f35e862f073b40&units=metric`;
    const response = await axios.get(endpoint);
    weatherData.value = response.data;
    console.log('weatherData', weatherData.value);
  } catch (error) {
    console.error('getWeatherByCoords', error.message);
    weatherData.value = null;
  }
};

const primary = getCssVar('primary');
const secondary = getCssVar('secondary');
</script>

<style lang="scss">
.q-page {
  background: v-bind(primary);
  background: -webkit-linear-gradient(
      to bottom,
      v-bind(secondary),
      v-bind(primary)
  );
  background: linear-gradient(to bottom, v-bind(secondary), v-bind(primary));

  &.bg-night {
    background: #0f2027;
    background: -webkit-linear-gradient(to bottom, #0f2027, #203a43, #2c5364);
    background: linear-gradient(to bottom, #0f2027, #203a43, #2c5364);
  }
}
.degree {
  top: -44px;
}
.skyline {
  flex: 0 0 100px;
  background: url(/skyline.png);
  background-size: contain;
  background-position: bottom;
}
</style>
