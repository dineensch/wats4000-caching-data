<template>
  <div>
    <favorite-cities v-bind:favoriteCities="favorites"></favorite-cities>
    <h2>City Search</h2>
    <message-container v-bind:messages="messages"></message-container>
    <form v-on:submit.prevent="getCities">
        <p>Enter city name: <input type="text" v-model="query" placeholder="Paris, TX"> <button type="submit">Go</button></p>
    </form>
    <load-spinner v-if="showLoading"></load-spinner>
    <ul class="cities" v-if="results && results.list.length > 0">
      <li v-for="city in results.list" v-bind:key="city.id">
        <h2>{{ city.name }}, {{ city.sys.country }}</h2>
        <p><router-link v-bind:to="{ name: 'CurrentWeather', params: { cityId: city.id } }">View Current Weather</router-link></p>
        <weather-summary v-bind:weatherData="city.weather"></weather-summary>
        <weather-data v-bind:weatherData="city.main"></weather-data>
        <p><button class="save" v-on:click="saveCity(city)">Save City to Favorites</button></p>
      </li>
    </ul>
  </div>
</template>

<script>
import { API } from "@/common/api";
import WeatherSummary from "@/components/WeatherSummary";
import WeatherData from "@/components/WeatherData";
import CubeSpinner from "@/components/CubeSpinner";
import MessageContainer from "@/components/MessageContainer";
import FavoriteCities from "@/components/FavoriteCities";

export default {
  name: "CitySearch",
  components: {
    "weather-summary": WeatherSummary,
    "weather-data": WeatherData,
    "load-spinner": CubeSpinner,
    "message-container": MessageContainer,
    "favorite-cities": FavoriteCities
  },
  data() {
    return {
      results: null,
      query: "",
      showLoading: false,
      messages: [],
      favorites: []
    };
  },
  created() {
    if (this.$ls.get("favoriteCities")) {
      this.favorites = this.$ls.get("favoriteCities");
    }
  },
  methods: {
    saveCity: function(city) {
      this.favorites.push(city);
      this.$ls.set("favoriteCities", this.favorites);
    },
    getCities: function() {
      this.results = null;
      this.showLoading = true;

      let cacheLabel = "citySearch_" + this.query;
      let cacheExpiry = 15 * 60 * 1000; // Sets to 15 minutes

      // if data is stored in cacheLabel:
      if (this.$ls.get(cacheLabel)) {
        console.log("Cached query detected.");
        this.results = this.$ls.get(cacheLabel);
        this.showLoading = false;
      } else {
        console.log("No cache available. Making API request.");
        API.get("find", {
          params: {
            q: this.query
          }
        })
          .then(response => {
            this.$ls.set(cacheLabel, response.data, cacheExpiry);
            console.log("New query has been cached as: " + cacheLabel);
            this.results = response.data;
            this.showLoading = false;
          })
          .catch(error => {
            this.messages.push({
              type: "error",
              text: error.message
            });
            this.showLoading = false;
          });
      } // end of else statement
    } // end of getCities function
  } // end of methods
}; // end of export
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
  margin: 5px;
}

a {
  color: #42b983;
}
</style>


