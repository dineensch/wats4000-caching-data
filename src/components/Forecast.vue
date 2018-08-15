<template>
  <div>

    <h2>Five Day Hourly Forecast <span v-if="weatherData"> for {{ weatherData.city.name }}, {{weatherData.city.country }}</span></h2>
    <message-container v-bind:messages="messages"></message-container>
    <p>
      <router-link to="/">Home</router-link> |
      <router-link v-bind:to="{ name: 'CurrentWeather', params: { cityId: $route.params.cityId } }">Current Weather <span v-if="weatherData"> for {{ weatherData.city.name }}, {{weatherData.city.country }}</span></router-link>
    </p>

    <ul v-if="weatherData" class="forecast">
      <transition-group name="fade" tag="div" appear>
        <li v-for="forecast in weatherData.list" v-bind:key="forecast.dt">
          <h3>{{ forecast.dt|formatDate }}</h3>
          <weather-summary v-bind:weatherData="forecast.weather"></weather-summary>
          <weather-data v-bind:weatherData="forecast.main"></weather-data>
        </li>
      </transition-group>
    </ul>
    <load-spinner v-if="showLoading"></load-spinner>
  </div>
</template>

<script>
import {API} from '@/common/api';
import WeatherSummary from '@/components/WeatherSummary';
import WeatherData from '@/components/WeatherData';
import CubeSpinner from '@/components/CubeSpinner';
import MessageContainer from '@/components/MessageContainer';


export default {
  name: 'Forecast',
  components: {
    'weather-summary': WeatherSummary,
    'weather-data': WeatherData,
    'message-container': MessageContainer,
    'load-spinner': CubeSpinner,

  },
  data () {
    return {
      weatherData: null,
      messages: [],
      query: '',
      showLoading: false,
      messages: [],
    
    }
  },
  created () {
    this.showLoading = true;

    let cacheLabel = 'forecast_' + this.$route.params.cityId;
    let cacheExpiry = 15 * 60 * 1000; // Sets to 15 minutes in milliseconds

    if (this.$ls.get(cacheLabel)) {
      console.log('Cache value detected.');
      this.weatherData = this.$ls.get(cacheLabel);
      this.showLoading = false;
    } else {
      console.log('No cache available. Making API request.');
      API.get('forecast', {
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
          type: 'error',
          text: error.message
        });
      });
    }
  },
  filters: {
    formatDate: function (timestamp){
      let date = new Date(timestamp * 1000);
      const months = ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'];
      const weekdays = ['Sunday', 'Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday'];
      //let weekday = date.getDay();
      let daynum = date.getDate();
      let month = date.getMonth();

      let hour = date.getHours();
      if (hour === 12) {
        hour = 'Noon';
      } else if (hour === 0) {
        hour = 'Midnight';
      } else if (hour > 12) {
        hour = hour - 12 + 'PM';
      } else if (hour < 12) {
        hour = hour + 'AM';
      }
      //let year = date.getFullYear();
      return `${ months[month] } ${ daynum } @ ${ hour }`;
    }
  }
}
</script>

<style scoped>
.fade-enter-active, .fade-leave-active {
  transition: opacity 1s
}
.fade-enter, .fade-leave-to {
  opacity: 0
}
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  width: 200px;
  min-height: 300px;
  border: solid 1px #e8e8e8;
  padding: 10px;
  margin: 5px;
}

a {
  color: #42b983;
}
</style>


