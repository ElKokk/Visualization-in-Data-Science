<script>
  export let carId;
  export let carIds;
  export let data;
  import { onMount } from 'svelte';
  import { createEventDispatcher } from "svelte";
  import Map from './Map.svelte';
  import * as d3 from 'd3';
  import 'bootstrap/dist/css/bootstrap.min.css';

  let sliderValue = 0;
  let timeIndicator;
  let filteredData = [];
  let selectedTime = '';
  let highlightedData = [];
  let carstops = [];


  onMount(async () => {
  const fetchcarstops = async () => {
    const res = await fetch("data/vast2021_carstops.json");
    const data = await res.json();
    return data;
  };
  carstops = await fetchcarstops();
})

$: {
  if (carId && carstops.length > 0) {
    createBarChart();
  }
}

let dayData = Array.from({length: 14}, (_, i) => i + 6);

function createBarChart() {
  
  const carStops = carstops.filter(stop => stop.car === carId);

  
  const chartData = d3.group(carStops, d => d.start.day);

  
  const sortedDays = Array.from(chartData.keys())
    .filter(day => day !== 20) 
    .sort((a, b) => a - b);

  
  const chartContainer = d3.select("#car-stop-chart");
  chartContainer.selectAll("*").remove();

  

  
  const margin = { top: 30, right: 20, bottom: 30, left: 30 };
  const width = 500 - margin.left - margin.right;
  
  const barHeight = carId === 31 ? 55 : carId === 101 || carId === 104 || carId === 105 || carId === 106 ? 35 : 20;
  const height = sortedDays.length * barHeight;

  
  const svg = chartContainer.append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
    .append("g")
    .attr("transform", `translate(${margin.left},${margin.top})`);

    svg.append("text")
      .attr("x", width / 2)
      .attr("y", height + margin.bottom)
      .style("text-anchor", "middle")
      .text("Hour");


  svg.append("text")
      .attr("transform", "rotate(-90)")
      .attr("y", 0 - margin.left - 5.5)
      .attr("x",0 - (height / 2))
      .attr("dy", "1em")
      .style("text-anchor", "middle")
      .text("Days");

  
  const y = d3.scaleBand()
  .range([0, height])
  .padding(0.1)
  .domain(dayData);

  
  const x = d3.scaleLinear()
    .range([0, width])
    .domain([0, 24 * 60 * 60]); 

  
  const color = d3.scaleOrdinal()
    .domain(["housing", "professional", "catering", "other"])
    .range(["#1f77b4","#d62728", "#2ca02c", "#ff7f0e"]);

  
  svg.append("g")
    .call(d3.axisLeft(y));

  let barElements = [];

  
  dayData.forEach((day, i) => {
  if (!chartData.has(day)) return;

  let dayStops = chartData.get(day).sort((a, b) => a.start.time - b.start.time);
  let previousEnd = 0;

  dayStops.forEach(stop => {
    const startX = x(previousEnd);
    const stopWidth = x(stop.end.time - stop.start.time);

    const rect = svg.append("rect")
      .attr("class", "bar")
      .attr("y", y(day))
      .attr("height", 20)
      .attr("x", startX)
      .attr("width", stopWidth)
      .attr("fill", color(stop.location.location_type));

    barElements.push({
      rect: rect,
      start: previousEnd,
      end: stop.end.time
    });

    previousEnd = stop.end.time;
  });
});

const allDays = Array.from({length: 14}, (_, i) => i + 6);


allDays.forEach((day, i) => {
  svg.append("rect")
    .attr("class", "background-bar")
    .attr("y", y(day))
    .attr("height", 20)
    .attr("x", 0)
    .attr("width", width)
    .attr("fill", "#eee"); 
});
dayData.forEach((day, i) => {
  if (!chartData.has(day)) {
    svg.append("text")
      .attr("class", "no-data-text")
      .attr("x", width / 2)
      .attr("y", y(day) + y.bandwidth() / 2)
      .text("No data");
  }
});





sortedDays.forEach((day, i) => {
  let dayStops = chartData.get(day).sort((a, b) => a.start.time - b.start.time);
  let previousEnd = 0;

  dayStops.forEach(stop => {
    const startX = x(previousEnd);
    const stopWidth = x(stop.end.time - stop.start.time);

    const rect = svg.append("rect")
      .attr("class", "bar")
      .attr("y", y(day))
      .attr("height", y.bandwidth())
      .attr("x", startX)
      .attr("width", stopWidth)
      .attr("fill", color(stop.location.location_type));

    
    rect.on('mouseover', function(event) {
      d3.select('#chartTooltip')
        .style('left', `${event.pageX + 10}px`)
        .style('top', `${event.pageY + 10}px`)
        .style('visibility', 'visible')
        .text(`Location: ${stop.location.name}', Type: ${stop.location.location_type} `);
    });

    rect.on('mouseout', function() {
      d3.select('#chartTooltip').style('visibility', 'hidden');
    });

    previousEnd = stop.end.time;
  });
});

const lineTimes = [0, 6 * 60 * 60, 12 * 60 * 60, 18 * 60 * 60, 24 * 60 * 60];

lineTimes.forEach(time => {
  svg.append('line')
    .attr('x1', x(time))
    .attr('y1', 0)
    .attr('x2', x(time))
    .attr('y2', height)
    .attr('stroke', 'black')
    .attr('stroke-width', 1);

  svg.append('text')
    .attr('x', x(time))
    .attr('y', height + 20)
    .attr('text-anchor', 'middle')
    .text(time / (60 * 60)); 
});


if (timeIndicator) {
      timeIndicator.remove();
    }

    timeIndicator = svg.append("line")
      .attr("class", "time-indicator")
      .attr("x1", 0)
      .attr("y1", 0)
      .attr("x2", 0)
      .attr("y2", height)
      .attr("stroke", "black")
      .attr("stroke-width", 2);


      if (timeIndicator) {

  const dayIndex = Math.floor(sliderValue / (24 * 60));
  const day = dayData[dayIndex];
  const secondOfDay = (sliderValue % (24 * 60)) * 60;

  
  timeIndicator.attr("y1", y(day)).attr("y2", y(day) + y.bandwidth());

  
  timeIndicator.attr("x1", x(secondOfDay)).attr("x2", x(secondOfDay));
}
  
  const day = Math.floor(sliderValue / (24 * 60));
  const timeOfDay = sliderValue % (24 * 60);
  const timeX = x(timeOfDay * 60);

  let barForTimeIndicator = null;
  for (let bar of barElements) {
    if (timeOfDay >= bar.start && timeOfDay <= bar.end) {
      barForTimeIndicator = bar;
      break;
    }
  }


let legend = svg.append('g')
  .attr('transform', 'translate(0, -30)');


let legendData = [
  { name: 'Housing', color: '#1f77b4' },
  { name: 'Professional', color: '#d62728' },
  { name: 'Catering', color: '#2ca02c' },
  { name: 'Other', color: '#ff7f0e' },
];


let legendItems = legend.selectAll('.legend-item')
  .data(legendData)
  .enter()
  .append('g')
  .attr('class', 'legend-item');


legendItems.append('rect')
  .attr('x', (d, i) => i * 120)
  .attr('width', 18)
  .attr('height', 18)
  .style('fill', d => d.color);


legendItems.append('text')
  .attr('x', (d, i) => i * 120 + 24)
  .attr('y', 9)
  .attr('dy', '.35em')
  .style('text-anchor', 'start')
  .style('font-size', '18px')
  .text(d => d.name);

}
$: {
  if (carId && sliderValue !== undefined) {
    createBarChart();
  }
}


  $: {
    filteredData = data.filter(d => d.cumulative_minute_total <= sliderValue);
    
    let day = Math.floor(sliderValue / (24 * 60)) + 6;
    let hour = Math.floor((sliderValue % (24 * 60)) / 60);
    let minute = sliderValue % 60;
    selectedTime = `Day ${day}, ${hour}:${minute < 10 ? '0' : ''}${minute}`;

    highlightedData = getPointsInRange(data, sliderValue);
  
  }

  $: {
  dispatch('changeSliderTime', { value: sliderValue });
}

  const dispatch = createEventDispatcher();


  function getPointsInRange(data, selectedMinute) {
    const lowerBound = selectedMinute - 15;
    const upperBound = selectedMinute + 15;
    const pointsInRange = data.filter(
      point => point.cumulative_minute_total >= lowerBound && point.cumulative_minute_total <= upperBound
    );
  
    console.log(pointsInRange);  // Add this line
    return pointsInRange;
}

  function navigateToCarOverview() {
    window.location.href = "/";
  }

  function nextCar() {
    const numericCarId = Number(carId);
    const currentIndex = carIds.indexOf(numericCarId);
    if (currentIndex < carIds.length - 1) {
      dispatch("changeCar", { carId: carIds[currentIndex + 1] });
    }
  }

  function previousCar() {
    const numericCarId = Number(carId);
    const currentIndex = carIds.indexOf(numericCarId);
    if (currentIndex > 0) {
      dispatch("changeCar", { carId: carIds[currentIndex - 1] });
    }
  }


</script>

<div id="legend"></div>

<div class="custom-tooltip" id="chartTooltip"></div>

<p><button class="btn custom-btn2" on:click={navigateToCarOverview}>Car overview</button></p>

<p>
  <button class="btn custom-btn" on:click={previousCar}>Previous Car</button>
  <button class="btn custom-btn" on:click={nextCar}>Next Car</button>
</p>

<h2 class="large-text">Eleftherios Kokkinis - UHasselt - 2262635</h2>
<h1 class="small-text">Details for car {carId}</h1>

<div>
  <input type="range" min="0" max="20160" bind:value={sliderValue} class="slider" />
  <span>Time: {selectedTime}</span>
</div>

<div class="flex-container">
    <div class="map-container">
      <Map key={carId} selectedCar={carId} mapSizeWidth={390} mapSizeHeight={300} detailsPage={true} highlightedData={highlightedData} sliderValue={sliderValue} />
    </div>
    <div class="chart-container">
      <div id="car-stop-chart"></div>
    </div>
</div>

<div class="bar" style="display: none;"></div>

<style>
  .large-text {
    font-size: 36px;
    color: black;
  }

  .small-text {
    font-size: 24px;
  }

  .slider {
    width: 30%;
  }

  .bar {
    fill: steelblue;
  }

  .custom-tooltip {
    position: absolute;
    text-align: center;
    width: 120px;
    height: 45px;
    padding: 5px;
    font: 12px sans-serif;
    background: rgba(246, 208, 156);
    border: 1px;
    border-radius: 8px;
    pointer-events: none;
    visibility: hidden;
}

.flex-container {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
}

.map-container, .chart-container {
    flex: 1;
    padding: 15px;
}

:global(.no-data-text) {
  fill: black;
  font-size: 20px;
  text-anchor: middle;
  alignment-baseline: middle;
}

.flex-container {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
}

.map-container,
.chart-container {
  flex: 1 2 0;
  margin-top: 30px;
  margin-right: 100px;
}

.chart-container {
    flex: 2;
    margin-top: 40px;
    margin-right: 250px;
}

#legend {
  position: absolute;
  top: 10px;
  right: 10px;
}

.custom-btn {
    background-color: #28a745;
    color: white;
    border-color: #ffffff;
    border-radius: 5px;
  }

  .custom-btn:hover {
    background-color: #0c4926;
    color: white;
    border-radius: 5px;
  }
    .custom-btn2 {
    background-color: #b51616;
    color: white;
    border-color: #080000;
    border-radius: 5px;
  }

  .custom-btn2:hover {
    background-color: #510808;
    color: white;
    border-radius: 5px;
  }
</style>