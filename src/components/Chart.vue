<script lang="ts">
import * as d3 from 'd3'
import { onMounted } from 'vue'

export default {
  props: {
    data: {
      type: Array,
      required: true
    }
  },

  setup(props) {
    onMounted(() => {
      plotChart(props.data as any, '#my_dataviz')
    })
  }
}

interface D3Data {date: string, value: string}

const plotChart = (d3Data: D3Data[], el: string) => {
  const mappedData = d3Data.map(({date, value}) => ({ date: d3.timeParse("%Y-%m-%d")(date), value }))
  // set the dimensions and margins of the graph
const margin = {top: 10, right: 30, bottom: 30, left: 60},
    width = 800 - margin.left - margin.right,
    height = 400 - margin.top - margin.bottom;

// append the svg object to the body of the page
const svg = d3.select(el)
  .append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
  .append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`)
    .data(mappedData)


    // Add X axis --> it is a date format
    const x = d3.scaleTime()
      .domain(d3.extent(mappedData, function(d) { return d.date; }))
      .range([ 0, width ]);
    svg.append("g")
      .attr("transform", `translate(0, ${height})`)
      .call(d3.axisBottom(x));

    // Add Y axis
    const y = d3.scaleLinear()
      .domain([0, d3.max(mappedData, function(d) { return +d.value; })])
      .range([ height, 0 ]);
    svg.append("g")
      .call(d3.axisLeft(y));

    // Add the line
    svg.append("path")
      .datum(mappedData)
      .attr("fill", "none")
      .attr("stroke", "steelblue")
      .attr("stroke-width", 1.5)
      .attr("d", d3.line()
        .x(function(d) { return x(d.date) })
        .y(function(d) { return y(d.value) })
        )

    }
</script>

<template>
  <div id="my_dataviz"></div>
</template>
