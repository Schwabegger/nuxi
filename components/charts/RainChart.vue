<template>
    <div class="rain-chart"></div>
</template>

<script>
import * as d3 from "d3";
import { ref, watch, onMounted } from 'vue';

export default {
    name: 'RainChart',
    props: {
        forecastDay: {
            type: Array,
            required: true
        }
    },
    setup(props) {
        const rainChartData = ref([]);
        const options = { hour: '2-digit', minute: '2-digit' };

        onMounted(() => {
            drawChart();
        });

        onUnmounted(() => {
            d3.select(".rain-chart svg").remove();
        });

        // Watch for changes in forecast (initialisation) and update the rainChartData
        watch(() => props.forecastDay, (newForecast) => {
            if (newForecast) {
                rainChartData.value = newForecast.map(item => ({
                    time: new Date(item.dt * 1000).toLocaleTimeString([], options),
                    rain: item.rain ? item.rain['3h'] : 0
                }));
                drawChart();
            }
        }, { immediate: true });

        function drawChart() {
            // Remove existing chart
            d3.select(".rain-chart svg").remove();

            const margin = { top: 20, right: 30, bottom: 50, left: 60 };
            const width = 800 - margin.left - margin.right;
            const height = 400 - margin.top - margin.bottom;

            // Append SVG
            const svg = d3.select(".rain-chart")
                .append("svg")
                .attr("width", width + margin.left + margin.right)
                .attr("height", height + margin.top + margin.bottom)
                .append("g")
                .attr("transform", `translate(${margin.left},${margin.top})`);

            // Parse time strings into Date objects
            const parseTime = d3.timeParse("%H:%M");
            rainChartData.value.forEach(d => {
                d.time = parseTime(d.time);
            });

            // X scale
            const x = d3.scaleTime()
                .domain(d3.extent(rainChartData.value, d => d.time))
                .range([0, width]);

            // Y scale
            const y = d3.scaleLinear()
                .domain([0, d3.max(rainChartData.value, d => d.rain)])
                .range([height, 0]);

            // Line
            const line = d3.line()
                .x(d => x(d.time))
                .y(d => y(d.rain));

            // Append line
            svg.append("path")
                .datum(rainChartData.value)
                .attr("fill", "none")
                .attr("stroke", "steelblue")
                .attr("stroke-width", 1.5)
                .attr("d", line);

            // Append X axis
            svg.append("g")
                .attr("transform", `translate(0,${height})`)
                .call(d3.axisBottom(x));

            // Append Y axis
            svg.append("g")
                .call(d3.axisLeft(y));
        }
    }
}
</script>

<style scoped>
.rain-chart {
    margin-top: 20px;
}
</style>