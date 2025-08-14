<template>
  <div id="app" :class="{
    'freezing': typeof weather.main != 'undefined' && weather.main.temp <= 0,
    'cold': typeof weather.main != 'undefined' && weather.main.temp > 0 && weather.main.temp <= 10,
    'semi-cold': typeof weather.main != 'undefined' && weather.main.temp > 10 && weather.main.temp <= 15,
    'sl-cold': typeof weather.main != 'undefined' && weather.main.temp > 15 && weather.main.temp <= 18,
    'midtemp': typeof weather.main != 'undefined' && weather.main.temp > 18 && weather.main.temp <= 22,
    'sl-warm': typeof weather.main != 'undefined' && weather.main.temp > 22 && weather.main.temp <= 25,
    'semi-warm': typeof weather.main != 'undefined' && weather.main.temp > 25 && weather.main.temp <= 28,
    'warm': typeof weather.main != 'undefined' && weather.main.temp > 28 && weather.main.temp <= 32,
    'hot': typeof weather.main != 'undefined' && weather.main.temp > 32,
    'undef': typeof weather.main == 'undefined'
  }">
    <main>
      <div class="search-box">
        <input type="text" class="search-bar" placeholder="Search...  (City name, Country code)" v-model="query"
          @keypress="fetchWeather" />
        <div class="search-error-msg" v-if="weather.error == 'Failed to fetch weather data'">Couldn't find weather data
          for this location...</div>
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
      weather: {}
    }
  },
  methods: {
    fetchWeather(e) {
      if (e.key == "Enter") {
        fetch(`${this.url_base}weather?q=${this.query}`)
          .then(res => {
            return res.json();
          }).then(this.setResults);
      }
    },
    setResults(results) {
      this.weather = results;
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
    }
  }
}
</script>



<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: 'montserrat', sans-serif;
}

#app {
  background-size: cover;
  background-position: bottom;
  transition: 0.4s;
}

main {
  min-height: 100vh;
  padding: 25px;
  display: grid;
  grid-template-rows: min-content;
  align-items: start;
  justify-content: stretch;
  background-image: linear-gradient(to bottom, rgba(0, 0, 0, 0.15), rgba(0, 0, 0, 0.50));
}

.search-box {
  width: 100%;
}

.search-box .search-bar {
  display: block;
  width: 100%;
  padding: 15px;
  color: #000;
  font-size: 20px;

  appearance: none;
  border: none;
  outline: none;
  background: none;

  background-color: rgba(255, 255, 255, 0.5);
  border-radius: 0px 16px 0px 16px;
  transition: 0.4s;
}

.search-box .search-bar:focus {
  box-shadow: 0px 0px 16px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.75);
  border-radius: 16px 0px 16px 0px;
}

.search-error-msg {
  padding: 1px 3px;
  margin: 0px 12px;
  width: fit-content;
  color: rgba(0, 0, 0, 0.8);
  font-size: 16px;
  background-color: rgba(255, 200, 200, 0.4);
  border-radius: 0 0 7px 7px;
  transition: 0.4s;
}

.search-box:focus-within .search-error-msg {
  margin: 0;
}

.results-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  align-self: stretch;
  margin: 25px 0 50px;
}

.location-box .location {
  color: #fff;
  font-size: 38px;
  font-weight: 500;
  text-align: center;
  text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
  margin-bottom: 5px;
}

.location-box .date {
  color: #fff;
  font-size: 24px;
  font-weight: 300;
  font-style: italic;
  text-align: center;
  text-shadow: 1px 2px rgba(0, 0, 0, 0.25);
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
  width: 90%;
  max-width: 866px;
}

.weather-extras {
  display: flex;
  justify-content: center;
  width: 90%;
  max-width: 866px;
  flex-wrap: wrap;
}

#app.freezing {
  background-image: url('./assets/backgrounds/freezing.jpg');
}

#app.cold {
  background-image: url('./assets/backgrounds/cold.jpg');
}

#app.semi-cold {
  background-image: url('./assets/backgrounds/semi-cold.jpg');
}

#app.sl-cold {
  background-image: url('./assets/backgrounds/sl-cold.jpg');
}

#app.midtemp {
  background-image: url('./assets/backgrounds/midtemp.png');
}

#app.sl-warm {
  background-image: url('./assets/backgrounds/sl-warm.jpg');
}

#app.semi-warm {
  background-image: url('./assets/backgrounds/semi-warm.jpg');
}

#app.warm {
  background-image: url('./assets/backgrounds/warm.jpg');
}

#app.hot {
  background-image: url('./assets/backgrounds/hot.jpg');
}

#app.undef {
  background-image: url('./assets/backgrounds/undef.png');
}
</style>