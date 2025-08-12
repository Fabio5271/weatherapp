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
      </div>

      <div class="results-container" v-if="typeof weather.main != 'undefined'">
        <div class="location-box">
          <div class="location">{{ weather.name }}, {{ weather.sys.country }}</div>
          <div class="date">{{ dateBuilder() }}</div>
        </div>

        <div class="weather-box">
          <div class="temp">{{ Math.round(weather.main.temp) }}°C</div>
          <div class="weather">{{ weather.weather[0].main }}</div>
        </div>

        <div class="weather-spacer"></div>

        <div class="weather-extras">
          <ExtraWidget title='Feels Like' :val="formattedData.feels_like" unit='°C' />
          <ExtraWidget title='Humidity' :val="weather.main.humidity" unit='%' />
        </div>
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
      formattedData: {}
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
      this.formattedData.feels_like = Math.round(results.main.feels_like);
      this.formattedData.localtime = this.dateBuilder("localtime");
    },
    dateBuilder(query) {
      let d = new Date();
      let months = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];
      let days = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];

      let day = days[d.getDay()];
      let date = d.getDate();
      let month = months[d.getMonth()];
      let year = d.getFullYear();
      let localtime = d.getHours() + ":" + d.getMinutes()

      switch (query) {
        case "localtime":
          return `${localtime}`;
      
        default:
          return `${day} ${date} ${month} ${year}`;
      }
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
  color: #313131;
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

.results-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  align-self: stretch;
  margin: 25px 0 75px;
}

.location-box .location {
  color: #fff;
  font-size: 38px;
  font-weight: 500;
  text-align: center;
  text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
  margin-bottom: 3px;
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

.weather-spacer {
  height: 0px;
  border-bottom: 2px solid rgba(255, 255, 255, 0.25);
  box-shadow: 1px 1px rgba(0, 0, 0, 0.25);
  margin: 20px auto 15px;
  width: 75%;
}

.weather-box .temp {
  display: inline-block;
  padding: 10px 25px;
  color: #fff;
  font-size: 102px;
  font-weight: 900;

  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.25);
  border-radius: 16px;
  margin: 25px 0px 25px;

  box-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}

.weather-box .weather {
  color: #fff;
  font-size: 48px;
  font-weight: 700;
  font-style: italic;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}

.weather-extras {
  display: flex;
  justify-content: space-around;
  width: 75%;
}
/* 
.weather-extras .extra-widget {
  display: inline-block;
  padding: 5px 12px;
  margin: 15px 15px;
  text-align: center;
  font-size: 24px;
  font-weight: 500;
  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
  color: #fff;
  background-color: rgba(255, 255, 255, 0.25);
  border-radius: 8px;
  box-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}

.weather-extras .extra-widget .value {
  font-size: 50px;
  font-weight: 900;
} */

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