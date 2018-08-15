<template>
  <div>

    <h2>Current Weather <span v-if="weatherData"> for {{ weatherData.name }}, {{weatherData.sys.country }}</span></h2>
    <message-container v-bind:messages="messages"></message-container>
    <p>
      <router-link to="/">Home</router-link> |
      <router-link v-bind:to="{ name: 'Forecast', params: { cityId: $route.params.cityId } }">View 5-Day Forecast</router-link>
    </p>
    <load-spinner v-if="showLoading"></load-spinner>
    <div v-if="weatherData">

      <weather-summary v-bind:weatherData="weatherData.weather"></weather-summary>

      <weather-data v-bind:weatherData="weatherData.main"></weather-data>

    </div>

  </div>
</template>

<script>
import { API } from "@/common/api";
import WeatherSummary from "@/components/WeatherSummary";
import WeatherData from "@/components/WeatherData";
import CubeSpinner from "@/components/CubeSpinner";
import MessageContainer from "@/components/MessageContainer";

export default {
  name: "CurrentWeather",
  components: {
    "weather-summary": WeatherSummary,
    "weather-data": WeatherData,
    "load-spinner": CubeSpinner,
    "message-container": MessageContainer
  },
  data() {
    return {
      weatherData: null,
      messages: [],
      query: "",
      showLoading: false
    };
  },
  created() {
    this.showLoading = true;

    let cacheLabel = "currentWeather_" + this.$route.params.cityId;
    let cacheExpiry = 15 * 60 * 1000; // Sets to 15 minutes in milliseconds

    if (this.$ls.get(cacheLabel)) {
      console.log("Cached value detected.");
      this.weatherData = this.$ls.get(cacheLabel);
      this.showLoading = false;
    } else {
      console.log("No cache available. Making API request.");
      API.get("weather", {
        params: {
          id: this.$route.params.cityId
        }
      })
        .then(response => {
          this.$ls.set(cacheLabel, response.data, cacheExpiry);
          this.showLoading = false;
          this.weatherData = response.data;
        })
        .catch(error => {
          this.showLoading = false;
          this.messages.push({
            type: "error",
            text: error.message
          });
        });
    }
  }
};
</script>

<style scoped>
.errors li {
  color: red;
  border: solid red 1px;
  padding: 5px;
}
h1,
h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  width: 300px;
  min-height: 300px;
  border: solid 1px #e8e8e8;
  padding: 10px;
}
a {
  color: #42b983;
}
</style>


