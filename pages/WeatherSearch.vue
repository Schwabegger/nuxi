<template>
  <div class="d-flex flex-column align-items-center justify-content-center vh-100">
    <div class="image-container">
    </div>
    <div class="search-container">
      <div class="search-bar-container" :class="{ 'dark-mode': $colorMode.value === 'dark' }">
        <input type="text" class="form-control search-bar" v-model="query" @input="handleInput"
          placeholder="Search for a location">
        <button class="search-button" @click="getWeatherForLocation" :disabled="locations.length !== 1">
          <Icon class="search-bar-icon" name="ph:paper-plane-tilt-fill" />
        </button>
      </div>
      <div class="dropdown mt-2 w-50" v-show="query && locations.length">
        <div v-for="(location, index) in locations" :key="index" class="dropdown-item"
          :class="{ 'dark-mode': $colorMode.value === 'dark' }" @click="selectLocation(location)">
          {{ location }}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'WeatherSearch',
  data() {
    return {
      query: '',
      locations: [],
      timeoutId: null
    }
  },
  methods: {
    handleInput() {
      clearTimeout(this.timeoutId);
      if (this.query.length >= 3) {
        this.timeoutId = setTimeout(this.fetchLocations, 200);
      } else {
        this.locations = [];
      }
    },
    fetchLocations() {
      fetch(`https://localhost:7003/WeatherForecast/GetAutocomplete?search=${this.query}`)
        .then(response => response.json())
        .then(data => {
          this.locations = data;
        })
        .catch(error => {
          console.error('Error fetching locations:', error);
        });
    },
    selectLocation(location) {
      this.query = location;
      this.getWeatherForLocation();
    },
    getWeatherForLocation() {
      if (!this.query) {
        return;
      }
      // navigate to WeatherView
      console.log('Navigating to weather view for location:', this.query);
      this.$router.push(`/weather?location=${this.query}`);
    }
  }
}
</script>

<style scoped>
.vh-100 {
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}

.search-container {
  position: relative;
  width: 100%;
}

.search-bar-container {
  position: relative;
  width: 100%;
}

.search-bar-icon {
  height: 50%;
  width: 50%;
  color: white;
}

.search-bar {
  height: 50px;
  width: 100%;
  font-size: 1.5rem;
  border-radius: 25px;
  padding-right: 50px;
  padding: 5px;
  border: solid 1px #ddd;
}

.search-bar-container.dark-mode {
  color: black !important;
}


.search-button {
  position: absolute;
  right: 0px;
  top: 0px;
  height: 100%;
  width: 55px;
  border-radius: 0 25px 25px 0;
  border: none;
  background: #aaa;
  cursor: pointer;
  padding: 0;
  margin: 0;
}

.image-container {
  position: relative;
  width: 300px;
  height: 300px;
  margin-bottom: 50px;
  background-image: url('../assets/logo.jpeg');
  background-size: cover;
  -webkit-mask-image: radial-gradient(ellipse at center, black 50%, transparent 70%);
  /* animation: animate-gradient 1s infinite; */
}

.dropdown {
  position: absolute;
  width: 100%;
  border: 1px solid #dfe1e5;
  box-shadow: none;
  border-radius: 0 0 24px 24px;
  overflow: auto;
  max-height: calc(3 * 40px);
  z-index: 1;
}

.dropdown-item {
  padding: 10px 20px;
  cursor: pointer;
}

.dropdown-item:hover {
  background-color: #eee;
}

.dropdown-item.dark-mode:hover {
  color: white;
  background-color: #333;
}

/* XD */
@keyframes animate-gradient {
  0% {
    -webkit-mask-image: radial-gradient(ellipse at center, black 30%, transparent 90%);
  }

  10% {
    -webkit-mask-image: radial-gradient(ellipse at center, black 30%, transparent 85%);
  }

  20% {
    -webkit-mask-image: radial-gradient(ellipse at center, black 30%, transparent 80%);
  }

  30% {
    -webkit-mask-image: radial-gradient(ellipse at center, black 30%, transparent 75%);
  }

  40% {
    -webkit-mask-image: radial-gradient(ellipse at center, black 30%, transparent 70%);
  }

  50% {
    -webkit-mask-image: radial-gradient(ellipse at center, black 30%, transparent 65%);
  }

  60% {
    -webkit-mask-image: radial-gradient(ellipse at center, black 30%, transparent 70%);
  }

  70% {
    -webkit-mask-image: radial-gradient(ellipse at center, black 30%, transparent 75%);
  }

  80% {
    -webkit-mask-image: radial-gradient(ellipse at center, black 30%, transparent 80%);
  }

  90% {
    -webkit-mask-image: radial-gradient(ellipse at center, black 30%, transparent 85%);
  }

  100% {
    -webkit-mask-image: radial-gradient(ellipse at center, black 30%, transparent 90%);
  }
}
</style>