<template>
    <div class="temperature-chart"></div>
</template>

<script>
import * as d3 from "d3";
import { ref, watch, onMounted } from 'vue';

export default {
    name: 'TemperatureChart',
    props: {
        forecastDay: {
            type: Array,
            required: true
        }
    },
    setup(props) {
        const temperatureChartData = ref([]);
        const options = { hour: '2-digit', minute: '2-digit' };

        onMounted(() => {
            drawChart();
        });

        onUnmounted(() => {
            d3.select(".temperature-chart svg").remove();
        });

        // Watch for changes in forecast (initialisation) and update the temperatureChartData
        watch(() => props.forecastDay, (newForecast) => {
            if (newForecast) {
                temperatureChartData.value = newForecast.map(item => ({
                    time: new Date(item.dt * 1000).toLocaleTimeString([], options),
                    temperature: Math.round(item.main.temp - 273.15)
                }));
                drawChart();
            }
        }, { immediate: true });

        // defenetly not self made 🙏
        function drawChart() {
            // Remove existing chart
            d3.select(".temperature-chart svg").remove();

            const margin = { top: 20, right: 30, bottom: 50, left: 60 };
            const width = 800 - margin.left - margin.right;
            const height = 400 - margin.top - margin.bottom;

            // Append SVG
            const svg = d3.select(".temperature-chart")
                .append("svg")
                .attr("width", width + margin.left + margin.right)
                .attr("height", height + margin.top + margin.bottom)
                .append("g")
                .attr("transform", `translate(${margin.left},${margin.top})`);

            // Parse time strings into Date objects
            const parseTime = d3.timeParse("%H:%M");
            temperatureChartData.value.forEach(d => {
                d.time = parseTime(d.time);
            });

            // Define scales
            const xScale = d3.scaleTime()
                .domain(d3.extent(temperatureChartData.value, d => d.time))
                .range([0, width]);

            const maxTemperature = d3.max(temperatureChartData.value, d => d.temperature);
            const minTemperature = d3.min(temperatureChartData.value, d => d.temperature);
            const yScale = d3.scaleLinear()
            .domain([Math.min(0, minTemperature), Math.max(30, maxTemperature)])
            .range([height, 0]);

            // Define gradient
            const gradient = svg.append("defs")
                .append("linearGradient")
                .attr("id", "temperature-gradient")
                .attr("gradientUnits", "userSpaceOnUse") // use the SVG coordinate system
                .attr("x1", 0).attr("y1", yScale(0)) // start of the gradient
                .attr("x2", 0).attr("y2", yScale(45)); // end of the gradient
            gradient.append("stop").attr("offset", "0%").attr("stop-color", "blue");
            gradient.append("stop").attr("offset", "100%").attr("stop-color", "red");

            // Define area generator
            const area = d3.area()
                .x(d => xScale(d.time))
                .y0(height)
                .y1(d => yScale(d.temperature));

            // Append axes
            svg.append("g")
                .attr("transform", `translate(0,${height})`)
                .call(d3.axisBottom(xScale));

            // Append area path
            svg.append("path")
                .datum(temperatureChartData.value)
                .attr("fill", "url(#temperature-gradient)")
                .attr("opacity", 0.7)
                .attr("d", area);

            // Append text for data point values
            svg.selectAll(".text")
                .data(temperatureChartData.value)
                .enter().append("text")
                .attr("x", d => xScale(d.time))
                .attr("y", d => yScale(d.temperature) - 10)
                .text(d => d.temperature + "°")
                .attr("class", "chart-point-label");
        }
    }
}
</script>

<style scoped>
.chart-point-label {
    text-anchor: middle;
}
</style>