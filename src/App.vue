<template>
  <!-- <img alt="Vue logo" src="./assets/logo.png">
  <HelloWorld msg="Welcome to Your Vue.js App"/> -->
  <div id="app"
  :class="{'freezing': typeof weather.main != 'undefined' && weather.main.temp <= 0,
          'cold': typeof weather.main != 'undefined' && weather.main.temp > 0 && weather.main.temp <= 10,
          'semi-cold': typeof weather.main != 'undefined' && weather.main.temp > 10 && weather.main.temp <= 15,
          'sl-cold': typeof weather.main != 'undefined' && weather.main.temp > 15 && weather.main.temp <= 18,
          'midtemp': typeof weather.main != 'undefined' && weather.main.temp > 18 && weather.main.temp <= 22,
          'sl-warm': typeof weather.main != 'undefined' && weather.main.temp > 22 && weather.main.temp <= 25,
          'semi-warm': typeof weather.main != 'undefined' && weather.main.temp > 25 && weather.main.temp <= 28,
          'warm': typeof weather.main != 'undefined' && weather.main.temp > 28 && weather.main.temp <= 32,
          'hot': typeof weather.main != 'undefined' && weather.main.temp > 32,
          'undef': typeof weather.main == 'undefined'}">
    <main>
      <div class="search-box">
          <input
            type="text"
            class="search-bar"
            placeholder="Search...  (City name, Country code)"
            v-model="query"
            @keypress="fetchWeather"/>
      </div>

      <div class="weather-wrap" v-if="typeof weather.main != 'undefined'">
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
          <div class="feelslike"><span class="flt">Feels Like</span><br><span class="fltemp">{{ Math.round(weather.main.feels_like) }}°C</span></div>
          <div class="humidity"><span class="flt">Humidity</span><br><span class="humval">{{ weather.main.humidity }}%</span></div>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  data () {
    return {
      api_key: '8c10cdbaa002cf3e6cbaaf08ab64077a',
      url_base: 'https://api.openweathermap.org/data/2.5/',
      query: '',
      weather: {}
    }
  },
  methods: {
    fetchWeather (e) {
      if (e.key == "Enter") {
        fetch(`${this.url_base}weather?q=${this.query}&units=metric&APPID=${this.api_key}`)
        .then(res => {
          return res.json();
        }).then(this.setResults);
      }
    },
    setResults (results) {
      this.weather = results;
    },
    dateBuilder () {
      let d = new Date();
      let months = ["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"];
      let days = ["Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"];

      let day = days[d.getDay()];
      let date = d.getDate();
      let month = months[d.getMonth()];
      let year = d.getFullYear();

      return `${day} ${date} ${month} ${year}`;
    }
  }
  // components: {
  //   HelloWorld
  // }
}
</script>

<style>
/* #app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
} */
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

main {
  min-height: 100vh;
  padding: 25px;

  background-image: linear-gradient(to bottom, rgba(0, 0, 0, 0.15), rgba(0, 0, 0, 0.50));
}

.search-box {
  width: 100%;
  margin-bottom: 30px;
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

.location-box .location {
  color: #fff;
  font-size: 32px;
  font-weight: 500;
  text-align: center;
  text-shadow: 1px 3px rgba(0, 0, 0, 0.25);
  margin-bottom: 3px;
}

.location-box .date {
  color: #fff;
  font-size: 20px;
  font-weight: 300;
  font-style: italic;
  text-align: center;
}

.weather-box {
  text-align: center;
  padding-bottom:20px ;
}

.weather-spacer {
  height: 0px;
  border-bottom: 2px solid rgba(255, 255, 255, 0.25);
  /* border-radius: 1px; */
  box-shadow: 1px 1px rgba(0, 0, 0, 0.25);
  margin: 5px auto;
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
  justify-content: space-evenly;
  text-align: center;
  margin-top: 15px;
}

.weather-extras .feelslike, .weather-extras .humidity {
  display: inline-block;
  padding: 5px 12px;
  color: #fff;
  font-size: 24px;
  font-weight: 500;

  text-shadow: 3px 6px rgba(0, 0, 0, 0.25);
  background-color: rgba(255, 255, 255, 0.25);
  border-radius: 8px;
  margin:  15px 0px;

  box-shadow: 3px 6px rgba(0, 0, 0, 0.25);
}

.feelslike .fltemp, .humidity .humval {
  font-size: 50px;
  font-weight: 900;
}

</style>
