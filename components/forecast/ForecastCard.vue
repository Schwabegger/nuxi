<template>
  <div class="card" :class="{ 'expanded': isExplanded }">
    <div class="day">{{ day }}</div>
    <div class="forecast">
      <img class="icon-weather" :src="forecastIconUrl" />
      <div class="temp">
        <div>{{ maxTemp }}°</div>
        <div>{{ minTemp }}°</div>
      </div>
    </div>
    <div v-if="isExplanded" class="info">
      <p class="desc">{{ forecast[0].weather[0].description }}</p>
      <p class="rain">{{ }}</p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ForecastCard',
  props: {
    forecast: {
      type: Array,
      required: true
    },
    isExplanded: {
      type: Boolean,
      default: false
    }
  },
  computed: {
    forecastIconUrl() {
      if (!this.forecast[0]) {
        return '';
      }
      return `http://openweathermap.org/img/w/${this.forecast[0].weather[0].icon}.png`;
    },
    day() {
      if (!this.forecast[0]) {
        return '';
      }
      let forecastDate = new Date(this.forecast[0].dt * 1000);
      if (forecastDate.toDateString() === new Date().toDateString()) {
        return 'Today';
      }
      return new Date(this.forecast[0].dt * 1000).toLocaleDateString('en-US', { weekday: 'short' }) + '. ' + new Date(this.forecast[0].dt * 1000).toLocaleDateString('en-US', { day: 'numeric' }) + '.';
    },
    maxTemp() {
      if (!this.forecast || !this.forecast[0]) {
        return '';
      }
      let maxTemp = this.forecast[0].main.temp_max;
      this.forecast.forEach(item => {
        if (item.main.temp_max > maxTemp) {
          maxTemp = item.main.temp_max;
        }
      });
      return this.kelvinToCelsius(maxTemp);
    },
    minTemp() {
      if (!this.forecast || !this.forecast[0]) {
        return '';
      }
      let minTemp = this.forecast[0].main.temp_min;
      this.forecast.forEach(item => {
        if (item.main.temp_min < minTemp) {
          minTemp = item.main.temp_min;
        }
      });
      return this.kelvinToCelsius(minTemp);
    },
    rain() {
      if (!this.forecast || !this.forecast[0]) {
        return '';
      }
      let rain = 0;
      this.forecast.forEach(item => {
        if (item.rain) {
          rain += item.rain['3h'];
        }
      });
      return `Rain: ${rain}mm`;
    }
  },
  methods: {
    kelvinToCelsius(kelvin) {
      return Math.round(kelvin - 273.15);
    }
  }
}
</script>

<style scoped>
.card {
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 1);
  padding: 20px;
  margin: 3px;
  width: 200px;
}

.day {
  font-size: 20px;
  font-weight: 600;
  text-align: left;
}

.temp {
  font-weight: 600;
  text-align: left;
}

.temp div {
  margin: 10px 0;
}

.forecast {
  margin-top: 10px;
  display: flex;
  align-items: center;
  font-size: 20px;
}

.icon-weather {
  width: 70px;
  height: 70px;
  margin-right: 10px;
}
</style>