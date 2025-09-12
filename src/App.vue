<template>
  <div id="app" :class="appBgImage">
    <main>
      <div class="search-box">
        <div class="search-bar">
          <input type="text" class="search-input" placeholder="Search...  (City name, Country code)" v-model="query"
            @keypress.enter="fetchWeather" />
          <div class="btn-area">
            <button class="search-button" @click="fetchWeather" @mouseenter="forceShowSearch = true"
              @mouseleave="forceShowSearch = false">
              <div class="loading-circle" v-if="isLoadingSearch && !forceShowSearch"></div>
              <img v-else class="search-icon" src="./assets/icons/search-icon-2.png" alt="🔍" />
            </button>
            <button class="geolocation-button" @click="getCurrentLocation" :disabled="isLoadingLocation">
              <div class="loading-circle" v-if="isLoadingLocation"></div>
              <img v-else class="geolocation-icon" src="./assets/icons/locate-icon-3.png" alt="📍" />
            </button>
          </div>
        </div>
        <div class="search-error-msg" v-if="weather.error == 'Failed to fetch weather data'">Couldn't find weather data
          for this location...</div>
        <div class="geolocation-error-msg" v-if="geoLocationError">{{ geoLocationError }}</div>
      </div>

      <div class="results-container" v-if="typeof weather.main != 'undefined'">
        <div class="location-box">
          <div class="location">{{ weather.name }}, {{ weather.sys.country }}</div>
          <div class="date">{{ dateBuilder() }} {{ dateBuilder("localtime") }} {{ dateBuilder("local_tz") }}</div>
        </div>

        <div class="weather-box">
          <div class="temp">{{ Math.round(weather.main.temp) }}°C</div>
          <div class="weather">{{ weather.weather[0].main }}</div>
        </div>

        <div class="weather-spacer"></div>

        <div class="weather-extras">
          <ExtraWidget title="Feels Like" :val="Math.round(weather.main.feels_like)" unit="°C" />
          <ExtraWidget title="Humidity" :val="weather.main.humidity" unit="%" />
          <ExtraWidget title="Wind Speed" :val="weather.wind.speed" unit=" m/s" />
          <ExtraWidget title="Wind Direction" :val="weather.wind.deg" unit="°" />
          <ExtraWidget title="Remote Time" :val="dateBuilder('remote_time')" />
          <ExtraWidget title="Remote Timezone" :val="dateBuilder('remote_tz')" />
          <ExtraWidget title="Time Difference" :val="dateBuilder('time_diff')" />
        </div>
      </div>
      <div class="err-no-data-cont" v-if="weather.error == 'Failed to fetch weather data'">

      </div>
    </main>
  </div>
</template>



<script>
import ExtraWidget from './components/ExtraWidget.vue';

export default {
  name: 'App',
  components: {
    ExtraWidget
  },
  data() {
    return {
      url_base: 'https://weatherapp-backend-fawn.vercel.app/api/',
      query: '',
      weather: {},
      isLoadingSearch: false,
      forceShowSearch: false,
      isLoadingLocation: false,
      geoLocationError: null,
      appBgImage: 'undef'
    }
  },
  mounted() {
    this.getCurrentLocation();
  },
  methods: {
    fetchWeather() {
      this.isLoadingSearch = true;
      fetch(`${this.url_base}weather?q=${this.query}`)
        .then(res => {
          return res.json();
        }).then(this.setResults)
        .catch(error => {
          console.error('Error fetching weather by search:', error);
          this.isLoadingSearch = false;
        });
    },
    fetchWeatherByCoords(lat, lon) {
      fetch(`${this.url_base}weather?lat=${lat}&lon=${lon}`)
        .then(res => {
          return res.json();
        }).then(this.setResults)
        .catch(error => {
          console.error('Error fetching weather by coordinates:', error);
          this.isLoadingLocation = false;
          this.geoLocationError = 'Failed to fetch weather data for your location.';
        });
    },
    setResults(results) {
      this.weather = results;
      this.geoLocationError = null;
      this.isLoadingSearch = false;
      this.isLoadingLocation = false;
      this.setAppBgImage();
    },
    getCurrentLocation() {
      if (!navigator.geolocation) {
        this.geoLocationError = 'Geolocation is not supported by this browser.';
        console.error('Geolocation is not supported by this browser.');
        return;
      }
      this.isLoadingLocation = true;
      this.geoLocationError = null;

      navigator.geolocation.getCurrentPosition(
        (position) => {
          const lat = position.coords.latitude;
          const lon = position.coords.longitude;
          console.log(`Geolocation API returned position: lat: ${lat} | lon: ${lon}`);
          this.fetchWeatherByCoords(lat, lon);
        },
        (error) => {
          this.isLoadingLocation = false;
          switch (error.code) {
            case error.PERMISSION_DENIED:
              this.geoLocationError = 'Please allow location access, or enter a city manually.';
              break;
            case error.POSITION_UNAVAILABLE:
              this.geoLocationError = 'Location information is unavailable.';
              break;
            case error.TIMEOUT:
              this.geoLocationError = 'Location request timed out.';
              break;
            default:
              this.geoLocationError = 'An unknown error occurred while retrieving location.';
              break;
          }
          console.error(`Geolocation API error: ${error.message}`);
        },
        {
          enableHighAccuracy: true,
          timeout: 10000,
          maximumAge: 300000  // Cache position for 5 minutes
        }
      );
    },
    dateBuilder(query) {
      let d = new Date();

      switch (query) {
        case "localtime":
          return this.formatTime(d.getHours(), d.getMinutes());

        case "local_tz":
          return this.formatTimeZone(-d.getTimezoneOffset());

        case "remote_time":
          return this.formatTime(d.getHours(), d.getMinutes(), this.dateBuilder("tz_diff"));

        case "remote_tz":
          return this.formatTimeZone(this.weather.timezone / 60);

        case "tz_diff":
          // wtf, d.getTimezoneOffset returns an inverted value (UTC -3 -> returns +180)
          return this.weather.timezone / 60 + d.getTimezoneOffset();

        case "time_diff":
          if (this.dateBuilder("tz_diff") < 0) {
            return `-${this.formatTimeDiff(this.dateBuilder("tz_diff"))}`;
          } else {
            return `+${this.formatTimeDiff(this.dateBuilder("tz_diff"))}`;
          }

        default:
          return this.currentDate();
      }
    },
    currentDate() {
      let months = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];
      let weekdays = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];
      let d = new Date();

      let weekday = weekdays[d.getDay()];
      let date = d.getDate();
      let month = months[d.getMonth()];
      let year = d.getFullYear();

      return `${weekday}, ${date} ${month} ${year}`;
    },
    formatTime(hours, minutes, addMinutes = 0) {
      return `${`${(Math.floor((hours * 60 + minutes + addMinutes) / 60)) % 24}`.padStart(2, "0")}:${`${(minutes + addMinutes) % 60}`.padStart(2, "0")}`;
    },
    formatTimeDiff(minutes) {
      return minutes % 60 == 0 ?
        `${Math.floor(Math.abs(minutes) / 60)}h` :
        `${`${Math.floor(Math.abs(minutes) / 60)}`.padStart(2, "0")}:${`${Math.abs(minutes) % 60}`.padStart(2, "0")}`;
    },
    formatTimeZone(tzMins) {
      return tzMins == 0 ?
        "UTC" : tzMins > 0 ?
          `UTC +${this.formatTimeDiff(tzMins)}` : `UTC -${this.formatTimeDiff(tzMins)}`;
    },
    setAppBgImage() {
      if (typeof this.weather.main == 'undefined') {
        this.appBgImage = 'undef';
        return;
      }
      
      let temp = this.weather.main.temp;

      this.appBgImage = temp <= -12 ? 'cold-5' :
      temp <= 0 ? 'cold-4' :
      temp <= 10 ? 'cold-3' :
      temp <= 15 ? 'cold-2' :
      temp <= 19 ? 'cold-1' :
      temp <= 22 ? 'midtemp' :
      temp <= 25 ? 'warm-1' :
      temp <= 28 ? 'warm-2' :
      temp <= 32 ? 'warm-3' :
      temp <= 36 ? 'warm-4' :
      'warm-5'
    }
  }
}
</script>



<style>
@media (max-width: 620px) {
  .extra-widget {
    font-size: 20px !important;
  }

  .extra-widget .value {
    font-size: 40px !important;
  }
}

@media (max-width: 526px) {
  .weather-extras {
    max-width: 430px !important;
  }

  .extra-widget {
    padding: 2px 7px 0px !important;
    margin: 6px !important;
    font-size: 15px !important;
    text-shadow: 1px 2px rgba(0, 0, 0, 0.25) !important;
    box-shadow: 2px 4px rgba(0, 0, 0, 0.25) !important;
  }

  .extra-widget .title {
    position: relative !important;
    top: 1px !important;
  }

  .extra-widget .value {
    font-size: 30px !important;
    text-shadow: 2px 4px rgba(0, 0, 0, 0.25) !important;
    top: -2px !important;
  }
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: 'Montserrat Variable', sans-serif;
}

#app {
  background-size: cover;
  background-position: center;
  transition: 0.4s;
}

main {
  display: grid;
  min-height: 100vh;
  padding: 25px;
  grid-template-rows: min-content;
  align-items: start;
  justify-content: stretch;
  background-image: linear-gradient(to bottom, rgba(0, 0, 0, 0.15), rgba(0, 0, 0, 0.50));
}

.search-box {
  display: grid;
  width: 100%;
  grid-template-columns: 1fr 1fr;
  grid-template-rows: max-content 22px;
}

.search-box .search-bar {
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
  width: 100%;
  grid-column-start: 1;
  grid-column-end: 3;

  background-color: rgba(255, 255, 255, 0.5);
  border-radius: 0px 16px 0px 16px;
  transition: 0.4s;
}

.search-input {
  display: inline-block;
  width: 100%;
  padding: 15px 10px 15px 15px;
  color: #000;
  font-size: 20px;

  appearance: none;
  border: none;
  outline: none;
  background: none;
}

.search-box .search-bar:focus-within {
  box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.75);
  border-radius: 16px 0px 16px 0px;
}

.search-bar .btn-area {
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
  justify-content: space-between;
  padding: 0 15px 0 0;
  flex: 1 1 100px;
  max-width: 100px;
  min-width: 83px;
}

.search-bar .btn-area button {
  width: 32px;
  min-width: 32px;
  max-width: 32px;
  height: 32px;
  min-height: 32px;
  max-height: 32px;
  margin: auto 0;
  border: none;
  background: none;
  cursor: pointer;
  opacity: 0.5;
  font-size: 24px;
}

.search-icon,
.geolocation-icon,
.search-button .loading-circle,
.geolocation-button .loading-circle {
  height: 100%;
  width: 100%;
}

.search-icon,
.search-button .loading-circle,
.geolocation-button .loading-circle {
  scale: 90%;
}

.loading-circle {
  border: 4px solid #000;
  border-top: 4px solid rgba(0, 0, 0, 0.5);
  border-radius: 50%;
  animation: spin 2s linear infinite;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }

  100% {
    transform: rotate(360deg);
  }
}

.search-error-msg,
.geolocation-error-msg {
  padding: 1px 3px;
  width: fit-content;
  color: rgba(0, 0, 0, 0.85);
  font-size: 16px;
  background-color: rgba(255, 200, 200, 0.5);
  border-radius: 0 0 7px 7px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-top: none;
  transition: 0.4s;
}

.search-error-msg {
  grid-column-start: 1;
  grid-column-end: 2;
  margin: 0px 13px;
}

.search-box:focus-within .search-error-msg {
  margin: 0;
}

.geolocation-error-msg {
  justify-self: end;
  grid-column-start: 2;
  grid-column-end: 3;
  margin: 0;
}

.search-box:focus-within .geolocation-error-msg {
  margin: 0px 13px;
}

.results-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  align-self: stretch;
  margin: 3px 0 50px;
}

.location-box .location {
  color: #fff;
  font-size: 38px;
  font-weight: 500;
  text-align: center;
  text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
  margin: 0 auto 5px;
  /* padding: 0 7px;
  background-color: rgba(0, 0, 0, 0.1);
  width: fit-content;
  border-radius: 9px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.22); */
}

.location-box .date {
  color: #fff;
  font-size: 24px;
  font-weight: 300;
  font-style: italic;
  text-align: center;
  text-shadow: 1px 2px rgba(0, 0, 0, 0.25);
  margin: 0 auto;
  /* padding: 0 7px;
  background-color: rgba(0, 0, 0, 0.18);
  width: fit-content;
  border-radius: 9px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.4); */
}

.weather-box {
  text-align: center;
}

.weather-box .temp {
  display: inline-block;
  margin: 18px 0px;
  padding: 10px 25px;

  color: #fff;
  font-size: 102px;
  font-weight: 900;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);

  background-color: rgba(255, 255, 255, 0.25);
  border-radius: 16px;
  box-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}

.weather-box .weather {
  color: #fff;
  font-size: 48px;
  font-weight: 700;
  font-style: italic;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}

.weather-spacer {
  height: 0px;
  border-bottom: 2px solid rgba(255, 255, 255, 0.25);
  box-shadow: 1px 1px rgba(0, 0, 0, 0.25);
  margin: 18px auto 10px;
  width: 95%;
  max-width: 866px;
}

.weather-extras {
  display: flex;
  justify-content: center;
  width: 95%;
  max-width: 866px;
  flex-wrap: wrap;
}

#app.cold-5 {
  background-image: url('./assets/backgrounds/cold-5.jpg');
}

#app.cold-4 {
  background-image: url('./assets/backgrounds/cold-4.jpg');
}

#app.cold-3 {
  background-image: url('./assets/backgrounds/cold-3.jpg');
}

#app.cold-2 {
  background-image: url('./assets/backgrounds/cold-2.jpg');
}

#app.cold-1 {
  background-image: url('./assets/backgrounds/cold-1.jpg');
}

#app.midtemp {
  background-image: url('./assets/backgrounds/midtemp.jpg');
}

#app.warm-1 {
  background-image: url('./assets/backgrounds/warm-1.jpg');
}

#app.warm-2 {
  background-image: url('./assets/backgrounds/warm-2.jpg');
}

#app.warm-3 {
  background-image: url('./assets/backgrounds/warm-3.jpg');
}

#app.warm-4 {
  background-image: url('./assets/backgrounds/warm-4.jpg');
}

#app.warm-5 {
  background-image: url('./assets/backgrounds/warm-5.jpg');
}

#app.undef {
  background-image: url('./assets/backgrounds/undef.jpg');
}
</style>