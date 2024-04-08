<template>
    <div class="weather-container">
        <div class="location">
            <h2>{{ location }}</h2>
        </div>
        <div class="current-weather">
            <img :src="iconUrl" alt="weather icon">
            <div class="temperature">{{ temperature }}°C</div>
            <div class="description">{{ description }}</div>
        </div>
    </div>
</template>

<script>
export default {
    data() {
        return {
            location: 'New York',
            temperature: 20,
            description: 'Sunny',
            iconUrl: 'http://openweathermap.org/img/w/10d.png'
        }
    }
}
</script>

<style scoped>
.weather-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 20px;
}

.location h2 {
    margin-bottom: 20px;
}

.current-weather {
    display: flex;
    align-items: center;
}

.current-weather img {
    width: 50px;
    height: 50px;
    margin-right: 20px;
}

.temperature {
    font-size: 2em;
    margin-right: 10px;
}

.description {
    font-size: 1.2em;
}
</style>