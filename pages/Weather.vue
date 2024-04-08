<template>
    <div class="weather-container">
        <h1 class="location">{{ location }}</h1>
        <div class="toggle-container">
            <label class="switch">
                <input type="checkbox" @change="toggleAutoUpdate">
                <span class="slider round"></span>
            </label>
            <label>Auto-update weather data</label>
        </div>
        <WeatherCard v-if="currentWeather" :currentWeather="currentWeather" :iconUrl="iconUrl"
            :sunriseTime="sunriseTime" :sunsetTime="sunsetTime" :lastUpdated="lastUpdated" :population="population" />
        <br>
        <div class="forecast-list">
            <h1>5 day forecast</h1>
            <div class="carousel-container">
                <button v-if="showLeftScrollButton" @click="scrollLeft" class="carousel-button carousel-button-left">
                    <Icon name="material-symbols:play-arrow-rounded" :style="{ transform: `scaleX(-1)` }" />
                </button>
                <div class="carousel" @scroll="onScroll">
                    <ForecastCard v-for="(day, index) in forecastDays" :key="index" :forecast="day"
                        :class="{ 'selected': index === selectedCardIndex }" @click="selectCard(index)" />
                </div>
                <button v-if="showRightScrollButton" @click="scrollRight" class="carousel-button carousel-button-right">
                    <Icon name="material-symbols:play-arrow-rounded" />
                </button>
            </div>
        </div>
        <div class="graphs">
            <div class="graph-options">
                <div class="tab-list">
                    <button v-for="(graph, index) in graphs" :key="index"
                        :class="{ active: selectedGraph === graph.value }" @click="selectedGraph = graph.value">
                        {{ graph.name }}
                    </button>
                </div>
                <div v-for="(graph, index) in graphs" :key="index">
                    <div v-if="selectedGraph === graph.value">
                        <TemperatureChart v-if="graph.value === 'temperature'" :forecastDay="forecastDays[selectedCardIndex]" />
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
import { ref, onMounted } from 'vue';
import { useRoute } from 'vue-router';
import WeatherCard from '@/components/weather/WeatherCard.vue';
import ForecastCard from '@/components/forecast/ForecastCard.vue';
import TemperatureChart from '@/components/charts/TemperatureChart.vue';

export default {
    name: 'WeatherView',
    components: {
        WeatherCard,
        ForecastCard,
        TemperatureChart
    },
    // methods is used to define methods in the component that can be called from the template
    methods: {
        scrollLeft() {
            const carousel = document.querySelector('.carousel');
            carousel.scrollLeft -= 200;
        },
        scrollRight() {
            const carousel = document.querySelector('.carousel');
            carousel.scrollLeft += 200;
        },
        onScroll() {
            const carousel = document.querySelector('.carousel');
            if (carousel.scrollLeft < 30)
                carousel.scrollLeft = 0;
            // else if (carousel.scrollWidth - carousel.scrollLeft <= 500)
            //     carousel.scrollLeft = carousel.scrollWidth;
            this.showLeftScrollButton = carousel.scrollLeft > 0;
            this.showRightScrollButton = carousel.scrollLeft + carousel.clientWidth < carousel.scrollWidth;
        },
        selectCard(index) {
            this.selectedCardIndex = index;
        }
    },
    // computed is used to create a computed property reactively based on other properties
    computed: {
        iconUrl() {
            if (this.currentWeather != null) {
                return `http://openweathermap.org/img/w/${this.currentWeather.weather[0].icon}.png`;
            }
            return '';
        },
        sunriseTime() {
            if (this.currentWeather != null) {
                return new Date(this.currentWeather.sys.sunrise * 1000).toLocaleTimeString([], this.options);
            }
            return '';
        },
        sunsetTime() {
            if (this.currentWeather != null) {
                return new Date(this.currentWeather.sys.sunset * 1000).toLocaleTimeString([], this.options);
            }
            return '';
        }
    },
    // setup is a new lifecycle hook in Vue 3, replaces the data function, used to set up the component's reactive state
    setup() {
        // ref is used to create a reactive reference to a value
        const weatherForecast = ref([]);
        const route = useRoute();
        const location = ref(route.query.location);
        const currentWeather = ref(null);
        const options = { hour: '2-digit', minute: '2-digit' };
        const autoUpdate = ref(false);
        const lastUpdated = ref('');
        const population = ref(0);
        const forecastDays = ref({});
        lastUpdated.value = new Date().toLocaleTimeString([], options);
        var updateInterval;
        const showLeftScrollButton = ref(false);
        const showRightScrollButton = ref(true);
        const selectedCardIndex = ref(0);
        const selectedGraph = ref('temperature');
        const graphs = [
            { name: 'Temperature', value: 'temperature' },
            { name: 'Rain', value: 'rain' },
            { name: 'Humidity', value: 'humidity' },
            { name: 'Wind', value: 'wind' }
        ];

        // drop the last part of the location string
        const locationParts = location.value.split(' ');
        locationParts.pop();
        const locationWithoutLastPart = locationParts.join(' ');

        // encodeURIComponent is used to encode the location string to be used in the URL
        const encodedLocation = encodeURIComponent(locationWithoutLastPart);

        // onMounted is a new lifecycle hook in Vue 3 that is called when the component is mounted to the DOM
        onMounted(() => {
            // fetch the current weather from the API
            fetchCurrentWeather();
            // fetch the weather forecast from the API
            fetchForecast();

            if (autoUpdate.value) {
                updateInterval = setInterval(() => {
                    fetchCurrentWeather();
                    fetchForecast();
                    lastUpdated.value = new Date().toLocaleTimeString([], options);
                }, 60000);
            }
        });

        onUnmounted(() => {
            if (autoUpdate.value) {
                clearInterval(updateInterval);
            }
        });

        function fetchCurrentWeather() {
            fetch(`https://localhost:7003/WeatherForecast/GetCurrentWeather?city=${encodedLocation}`)
                .then(response => response.json())
                .then(data => {
                    data.Celsius = convertFromKelvinToCelsius(data.main.temp);
                    data.CelsiusMax = convertFromKelvinToCelsius(data.main.temp_max);
                    data.CelsiusMin = convertFromKelvinToCelsius(data.main.temp_min);
                    data.CelsiusFeelsLike = convertFromKelvinToCelsius(data.main.feels_like);
                    data.weather[0].description = data.weather[0].description.charAt(0).toUpperCase() + data.weather[0].description.slice(1)
                    currentWeather.value = data;
                })
                .catch(error => {
                    console.error('Error fetching current weather:', error);
                });
        }

        function fetchForecast() {
            fetch(`https://localhost:7003/WeatherForecast/GetWeatherForecast?city=${encodedLocation}`)
                .then(response => response.json())
                .then(data => {
                    data.list.forEach(item => {
                        item.CelsiusMax = convertFromKelvinToCelsius(item.main.temp_max);
                        item.CelsiusMin = convertFromKelvinToCelsius(item.main.temp_min);
                        item.CelsiusFeelsLike = convertFromKelvinToCelsius(item.main.feels_like);
                        item.weather[0].description = item.weather[0].description.charAt(0).toUpperCase() + item.weather[0].description.slice(1)
                    });
                    population.value = data.city.population;
                    forecastDays.value = splitForecastIntoDays(data);
                    // weatherForecast.value = splitForecastIntoDays(data);
                })
                .catch(error => {
                    console.error('Error fetching forecast:', error);
                });
        }

        function splitForecastIntoDays(forecast) {
            const days = {};
            let i = 0;
            days[i] = [];

            forecast.list.forEach(item => {
                const date = new Date(item.dt * 1000).toLocaleDateString('en-US');

                if (days[i][0] && toDate(days[i][0].dt) !== date) {
                    i++;
                    days[i] = [];
                }

                days[i].push(item);
            });

            return days;
        }

        function toDate(date) {
            return new Date(date * 1000).toLocaleDateString('en-US');
        }

        function convertFromKelvinToCelsius(kelvin) {
            return Math.round(kelvin - 273.15);
        }

        function convertFromKelvinToFahrenheit(kelvin) {
            return Math.round((kelvin - 273.15) * 9 / 5 + 32);
        }

        function update() {
            fetchCurrentWeather();
            fetchForecast();
            lastUpdated.value = new Date().toLocaleTimeString([], options);
        }

        function toggleAutoUpdate() {
            autoUpdate.value = !autoUpdate.value;

            if (autoUpdate.value) {
                update();
                updateInterval = setInterval(() => {
                    update();
                }, 60000);
            } else {
                clearInterval(updateInterval);
            }
        }

        return {
            location,
            weatherForecast,
            currentWeather,
            options,
            lastUpdated,
            autoUpdate,
            population,
            forecastDays,
            toggleAutoUpdate,
            showLeftScrollButton,
            showRightScrollButton,
            selectedCardIndex,
            selectedGraph,
            graphs
        };
    }
}
</script>

<style scoped>
.tab-list {
    display: flex;
    gap: 10px;
    justify-content: center;
}

.tab-list button {
    cursor: pointer;
    border-radius: 5px;
    padding: 5px;
}

.tab-list .active {
    color: #fff;
    background-color: rgba(0, 0, 0, 0.5);
}

.graphs {
    margin-top: 2em;
}

.selected {
    border: 2px solid #000;
    /* Or any other style you want */
}

.carousel-container {
    position: relative;
    left: 0;
    height: fit-content;
}

.carousel-button {
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    background: rgba(255, 255, 255, 1);
    color: rgba(0, 0, 0, 1);
    border-radius: 5px;
    text-align: center;
    height: 30px;
    width: 18px;
    cursor: pointer;
}

.carousel-button:hover {
    background: rgba(255, 255, 255, 0.7);
}

.carousel-button-left {
    left: 0px;
}

.carousel-button-right {
    right: 0px;
}

.carousel {
    display: flex;
    overflow-x: auto;
    scrollbar-width: none;
    scroll-behavior: smooth;
    scrollbar-color: rgba(155, 155, 155, 0.7) transparent;
    left: 0;
    gap: 10px;
}

.carousel .card {
    flex: 0 0 auto;
    scroll-snap-align: start;
    /* width: 200px; */
}

.toggle-container {
    width: 20px;
    height: 20px;
    text-align: left;
    margin-bottom: 1em;
    display: inline;
    margin-left: 0;
}

.weather-container {
    width: 60%;
    margin: 0 auto;
    text-align: center;
}

.location {
    font-size: 2em;
    margin-bottom: 1em;
}

.switch {
    position: relative;
    display: inline-block;
    width: 40px;
    height: 20px;
}

/* Hide default HTML checkbox */
.switch input {
    opacity: 0;
    width: 0;
    height: 0;
}

/* The slider */
.slider {
    position: absolute;
    cursor: pointer;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: #ccc;
    -webkit-transition: .4s;
    transition: .4s;
}

/* Circle */
.slider:before {
    position: absolute;
    content: "";
    height: 16px;
    width: 16px;
    left: 2px;
    bottom: 2px;
    background-color: white;
    -webkit-transition: .4s;
    transition: .4s;
}

input:checked+.slider {
    background-color: #2196F3;
}

input:focus+.slider {
    box-shadow: 0 0 1px #2196F3;
}

input:checked+.slider:before {
    -webkit-transform: translateX(20px);
    -ms-transform: translateX(20px);
    transform: translateX(20px);
}

/* Rounded sliders */
.slider.round {
    border-radius: 20px;
}

.slider.round:before {
    border-radius: 50%;
}
</style>