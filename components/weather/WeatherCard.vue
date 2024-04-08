<template>
    <div v-if="currentWeather != null" class="weather-card">
        <!-- <p>{{ currentTime }}</p> -->
        <img :src="iconUrl" alt="weather icon" style="display: inline;">
        <p style="font-size: 60px; display: inline">{{ currentWeather.Celsius }}°C</p>
        <p style="font-weight: 600;">Feels like {{ currentWeather.CelsiusFeelsLike }}°C. {{
        currentWeather.weather[0].description }}.</p>
        <ul class="weather-items grid grid-cols-2 w-fit gap-x-20 gap-y-1">
            <li class="wind">
                <div class="wind-direction-icon-container">
                    <svg viewBox="0 0 1000 1000" class="icon-wind-direction"
                        :style="{ transform: `rotate(${currentWeather.wind.deg - 180}deg)` }">
                        <g fill="#ffffff">
                            <path
                                d="M510.5,749.6c-14.9-9.9-38.1-9.9-53.1,1.7l-262,207.3c-14.9,11.6-21.6,6.6-14.9-11.6L474,48.1c5-16.6,14.9-18.2,21.6,0l325,898.7c6.6,16.6-1.7,23.2-14.9,11.6L510.5,749.6z">
                            </path>
                            <path
                                d="M817.2,990c-8.3,0-16.6-3.3-26.5-9.9L497.2,769.5c-5-3.3-18.2-3.3-23.2,0L210.3,976.7c-19.9,16.6-41.5,14.9-51.4,0c-6.6-9.9-8.3-21.6-3.3-38.1L449.1,39.8C459,13.3,477.3,10,483.9,10c6.6,0,24.9,3.3,34.8,29.8l325,898.7c5,14.9,5,28.2-1.7,38.1C837.1,985,827.2,990,817.2,990z M485.6,716.4c14.9,0,28.2,5,39.8,11.6l255.4,182.4L485.6,92.9l-267,814.2l223.9-177.4C454.1,721.4,469,716.4,485.6,716.4z">
                            </path>
                        </g>
                    </svg>
                </div>
                <p class="wind-speed">{{ currentWeather.wind.speed }} m/s</p>
                <p style="margin-left: 5px;">({{ windDirectionLetter }})</p>
            </li>
            <li class="preassure">
                <div class="preassure-icon-container">
                    <svg width="15pt" height="15pt" viewBox="0 0 96 96" class="icon-pressure">
                        <g transform="translate(0,90) scale(0.100000,-0.100000)" fill="#ffffff">
                            <path d="M351 854 c-98 -35 -179 -108 -227 -202 -27 -53 -29 -65 -29 -172 0
                                -107 2 -119 29 -172 38 -75 104 -141 180 -181 58 -31 66 -32 176 -32 110 0
                                118 1 175 32 77 40 138 101 178 178 31 57 32 65 32 175 0 110 -1 118 -32 176
                                -40 76 -106 142 -181 179 -49 25 -71 29 -157 32 -73 2 -112 -1 -144 -13z m259
                                -80 c73 -34 126 -86 161 -159 24 -50 29 -73 29 -135 0 -62 -5 -85 -29 -135
                                -57 -119 -161 -185 -291 -185 -130 0 -234 66 -291 185 -24 50 -29 73 -29 135
                                0 130 66 234 185 291 82 40 184 41 265 3z">
                            </path>
                            <path d="M545 600 c-35 -35 -68 -60 -80 -60 -27 0 -45 -18 -45 -45 0 -33 -50
                            -75 -89 -75 -18 0 -41 -5 -53 -11 -20 -11 -20 -11 3 -35 12 -13 33 -24 46 -24
                            17 0 23 -6 23 -23 0 -13 10 -33 23 -45 30 -28 47 -13 47 43 0 32 6 47 28 68
                            15 15 37 27 48 27 26 0 44 18 44 44 0 12 26 47 60 81 l60 61 -28 27 -28 27
                            -59 -60z">
                            </path>
                        </g>
                    </svg>
                </div>
                <p class="preassure-value">{{ currentWeather.main.pressure }} hPa</p>
            </li>
            <li class="humidity">
                <Icon class="icon-humidity" name="material-symbols:humidity-percentage" size="25" color="lightblue" />
                <p style="margin-left: 5px;">{{ currentWeather.main.humidity }}%</p>
            </li>
            <li class="visibility">
                <Icon class="icon-visibility" name="material-symbols:visibility" size="25" />
                <p style="margin-left: 5px;">{{ currentWeather.visibility / 1000 }} km</p>
            </li>
            <li class="sunrise">
                <Icon class="icon-sunrise" name="solar:sunrise-bold" size="25" color="orange" />
                <p style="margin-left: 5px;">{{ sunriseTime }}</p>
            </li>
            <li class="sunset">
                <Icon class="icon-sunset" name="solar:sunset-bold" size="25" color="orangered" />
                <p style="margin-left: 5px;">{{ sunsetTime }}</p>
            </li>
            <li class="min-max-temp">
                <Icon class="icon-min-max-temp" name="solar:temperature-bold-duotone" size="28" />
                <p>{{ currentWeather.CelsiusMax }}°C / {{ currentWeather.CelsiusMin }}°C</p>
            </li>
        </ul>
        <div class="info">
            <p class="population">Population: {{ population }}</p>
            <p class="last-updated">Last updated: {{ lastUpdated }}</p>
        </div>
    </div>
    <div v-else>
        <p>No weather data available</p>
    </div>
</template>

<script>
export default {
    name: 'WeatherCard',
    props: {
        currentWeather: {
            type: Object,
            required: true
        },
        iconUrl: {
            type: String,
            required: true
        },
        sunriseTime: {
            type: String,
            required: true
        },
        sunsetTime: {
            type: String,
            required: true
        },
        lastUpdated: {
            type: String,
            required: true
        },
        population: {
            type: Number,
            required: true
        }
    },
    computed: {
        windDirectionLetter() {
            if (this.currentWeather.wind.deg >= 337.5 || this.currentWeather.wind.deg < 22.5) {
                return 'N';
            } else if (this.currentWeather.wind.deg >= 22.5 && this.currentWeather.wind.deg < 67.5) {
                return 'NE';
            } else if (this.currentWeather.wind.deg >= 67.5 && this.currentWeather.wind.deg < 112.5) {
                return 'E';
            } else if (this.currentWeather.wind.deg >= 112.5 && this.currentWeather.wind.deg < 157.5) {
                return 'SE';
            } else if (this.currentWeather.wind.deg >= 157.5 && this.currentWeather.wind.deg < 202.5) {
                return 'S';
            } else if (this.currentWeather.wind.deg >= 202.5 && this.currentWeather.wind.deg < 247.5) {
                return 'SW';
            } else if (this.currentWeather.wind.deg >= 247.5 && this.currentWeather.wind.deg < 292.5) {
                return 'W';
            } else {
                return 'NW';
            }
        }
    }
}
</script>

<style scoped>
.info {
    position: absolute;
    bottom: 0;
    right: 0;
    font-size: 11px;
    color: #ffffff;
    padding: 5px;
    background-color: rgba(0, 0, 0, 0.5);
    border-radius: 0 0 20px 0;
}

.wind,
.humidity,
.visibility,
.min-max-temp,
.sunset,
.sunrise {
    display: flex;
    align-items: center;
}

.weather-items {
    margin-top: 10px;
    font-weight: 600;
    font-size: larger;
}

.preassure-icon-container {
    display: inline-block;
    width: 15px;
    height: 15px;
}

.preassure-value {
    display: inline-block;
    margin-left: 13px;
}

.wind-direction-icon-container {
    display: inline-block;
    width: 15px;
    height: 15px;
}

.wind-speed {
    display: inline-block;
    margin-left: 5px;
}

.weather-card {
    position: relative;
    width: 612px;
    height: 304px;
    padding-left: 20px;
    text-align: left;
    font-size: 20px;
    border-radius: 20px;
    background-image: url('https://assets.msn.com/weathermapdata/1/static/images/webps/v1.0/partlysunny_day.webp');
}
</style>