<script>
  import { createEventDispatcher } from 'svelte';
  import * as d3 from 'd3';
  import 'bootstrap/dist/css/bootstrap.min.css';

  import { onMount } from 'svelte';

  export let selectedCar;
  export let mapSizeWidth = 600;
  export let mapSizeHeight = 600;
  export let detailsPage = false;
  export let sliderValue;

  

  export let highlightedData = [];

  let data = [];
  let businesses = [];
  let businessTooltipData = {
  visible: false,
  name: "",
  x: 0,
  y: 0,
};

let coordinateTooltipData = {
  visible: false,
  longitude: "",
  latitude: "",
  x: 0,
  y: 0,
};

  onMount(async () => {
  const fetchGpscoordinates = async () => {
    const res = await fetch("data/vast2021_gps_coordinates.json");
    const data = await res.json();
    return data;
  };

  const fetchbusinesses = async () => {
    const res = await fetch("data/vast2021_businesses.json");
    const data = await res.json();
    return data;
  };

  const fetchcarstops = async () => {
    const res = await fetch("data/vast2021_carstops.json");
    const data = await res.json();
    return data;
  };

  data = await fetchGpscoordinates();
  businesses = await fetchbusinesses();
  carstops = await fetchcarstops();


  carstops.forEach(stop => {
    const carId = stop.car;
    const startTime = stop.start.day * 24 * 60 * 60 + stop.start.time;
    const endTime = stop.end.day * 24 * 60 * 60 + stop.end.time;

    
    data.forEach(d => {
      if (d.car_id === carId && d.cumulative_minute_total >= startTime && d.cumulative_minute_total <= endTime) {
        d.stop = true;
      }
    });
  });

  if (data && data.length && businesses && businesses.length) {
    createVisualization();
    createBusinessesVisualization();
  }


});

  const dispatch = createEventDispatcher();


$: {
  if (data.length && businesses.length) {
    highlightedData = data.filter(d => Math.abs(d.cumulative_minute_total - sliderValue) <= 15);
    createVisualization(selectedCar);
    createBusinessesVisualization();
    updateCarVisualization();
  }
}

  $: {
  if (highlightedData.length && businesses.length) {
    
    updateCarVisualization();
  }
}

  function getCombinedExtent(dataArray, businessArray, key) {
    const dataExtent = d3.extent(dataArray, (d) => d[key]);
    const businessExtent = d3.extent(businessArray, (d) => d[key]);
    return [Math.min(dataExtent[0], businessExtent[0]), Math.max(dataExtent[1], businessExtent[1])];
  }

  function createVisualization() {
  
  
  const visualizedData = detailsPage ? data.filter((d) => d.car_id === selectedCar) : data;


  d3.select("#gps-map").select("svg").remove();

let width, height, margin;

if (detailsPage) {
  
  margin = { top: 0, right: 0, bottom: 0, left: 0 };
  width = (mapSizeWidth - margin.left - margin.right);
  height = mapSizeHeight - margin.top - margin.bottom;
} else {
  margin = { top: 10, right: 30, bottom: 30, left: 60 };
  width = mapSizeWidth - margin.left - margin.right;
  height = mapSizeHeight - margin.top - margin.bottom;
}
  const svg = d3
    .select("#gps-map")
    .append("svg")
    .attr("width", width + margin.left + margin.right)
    .attr("height", height + margin.top + margin.bottom)
    .append("g")
    .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

  const xScale = d3
    .scaleLinear()
    .domain(getCombinedExtent(visualizedData, businesses, "long"))
    .range([0, width]);

  const yScale = d3
    .scaleLinear()
    .domain(getCombinedExtent(visualizedData, businesses, "lat"))
    .range([height, 0]);

    svg
      .selectAll("circle")
      .data(visualizedData)
      .enter()
      .append("circle")
      .attr("class", "gps-point")
      .attr("cx", (d) => xScale(d.long))
      .attr("cy", (d) => yScale(d.lat))
      .attr("r", 2)
      
      .attr("fill", (d) => {
    
    const isHighlighted = highlightedData.some(
      (highlightedPoint) =>
        highlightedPoint.car_id === d.car_id &&
        highlightedPoint.cumulative_minute_total === d.cumulative_minute_total
    );

    
    if (isHighlighted) {
      return "red";
    } else if (detailsPage) {
      
      return d.car_id === selectedCar ? "blue" : "grey";
    } else {
      return d.car_id === selectedCar ? "red" : "black";
    }
  })
      .attr("opacity", (d) => (d.car_id === selectedCar ? 1 : (detailsPage ? 0.03 : 0.03)))
      .on("mouseover", (event, d) => {
        const isHighlighted = highlightedData.some(
        (highlightedPoint) =>
          highlightedPoint.car_id === d.car_id &&
          highlightedPoint.cumulative_minute_total === d.cumulative_minute_total
      );
    
    
    if (isHighlighted) {
      coordinateTooltipData.visible = true;
      coordinateTooltipData.longitude = d.long;
      coordinateTooltipData.latitude = d.lat;
      coordinateTooltipData.day = d.day;
      coordinateTooltipData.hour = d.hour;
      coordinateTooltipData.minute = d.minute;
      coordinateTooltipData.x = event.pageX + 10;
      coordinateTooltipData.y = event.pageY - 30;


      d3.select(event.target)
          .attr("stroke", "#333")
          .attr("stroke-width", 2)
          .attr("opacity", 1);
    }
  })
  .on("mouseout", () => {
    coordinateTooltipData.visible = false;

    d3.select(event.target)
    .attr('stroke', 'none')
    .attr('stroke-width', 0); 
  });
      
  }


function createBusinessesVisualization() {
  if (detailsPage == false) {
  const svg = d3.select("#gps-map svg g");

  const margin = { top: 10, right: 30, bottom: 30, left: 60 };
  const width = mapSizeWidth - margin.left - margin.right;
  const height = mapSizeHeight - margin.top - margin.bottom;

  const xScale = d3
    .scaleLinear()
    .domain(getCombinedExtent(data, businesses, "long"))
    .range([0, width]);

  const yScale = d3
    .scaleLinear()
    .domain(getCombinedExtent(data, businesses, "lat"))
    .range([height, 0]);

  const colorScale = d3
    .scaleOrdinal()
    .domain(["catering", "domestic", "housing", "professional", "other"])
    .range(["#4daf62", "#08519c", "#4b97c9", "#970b13", "#ff7f00"]);


  function showTooltip(event, d) {
    businessTooltipData.visible = true;
    businessTooltipData.name = d.name;
    businessTooltipData.x = event.pageX + 10;
    businessTooltipData.y = event.pageY - 30;
}

  function hideTooltip() {
    businessTooltipData.visible = false;
  }

  svg
    .selectAll(".business")
    .data(businesses)
    .enter()
    .append("circle")
    .attr("class", "business")
    .attr("cx", (d) => xScale(d.long))
    .attr("cy", (d) => yScale(d.lat))
    .attr("r", 4)
    .attr("fill", (d) => colorScale(d.type))
    .attr("opacity", 0.7)
    .attr("class", (d) => "business " + d.type)
    .on("mouseover", (event, d) => {
        showTooltip(event, d);
        d3.select(event.target)
          .attr("stroke", "#333")
          .attr("stroke-width", 2)
          .attr("opacity", 1);
  })
    .on("mouseout", (event) => {
      hideTooltip();
      d3.select(event.target)
        .attr("stroke", "none")
        .attr("stroke-width", 0)
        .attr("opacity", 0.7);
});
return;
}
}
function updateCarVisualization() {
  d3.select("#gps-map svg g")
    .selectAll("circle.gps-point")
    .attr("fill", (d) => {
      const isHighlighted = highlightedData.some(
        (highlightedPoint) =>
          highlightedPoint.car_id === d.car_id &&
          highlightedPoint.cumulative_minute_total === d.cumulative_minute_total
      );

      if (isHighlighted) {
        return "red";
      } else if (detailsPage) {
        return d.car_id === selectedCar ? "#08519c" : "grey";
      } else {
        return d.car_id === selectedCar ? "red" : "black";
      }
    })
    .attr("r", (d) => {
      const isHighlighted = highlightedData.some(
        (highlightedPoint) =>
          highlightedPoint.car_id === d.car_id &&
          highlightedPoint.cumulative_minute_total === d.cumulative_minute_total
      );

    
      return isHighlighted ? 4 : 2;
    })
    .attr("opacity", (d) => {
  const isHighlighted = highlightedData.some(
    (highlightedPoint) =>
      highlightedPoint.car_id === d.car_id &&
      highlightedPoint.cumulative_minute_total === d.cumulative_minute_total
  );

  
  if (!detailsPage && d.car_id === selectedCar) {
    return 1;
  }
  
  else if (detailsPage && isHighlighted) {
    return 1;
  } 
  else if (detailsPage) {
    
    return 0.2; 
  }
  else if (!detailsPage) {
    return 0.03;
  }
  });
}
</script>

<div id="gps-map"></div>  

{#if businessTooltipData.visible}
<div
  class="custom-tooltip"
  style="left: {businessTooltipData.x}px; top: {businessTooltipData.y}px;"
>
  {businessTooltipData.name}
</div>
{/if}

{#if coordinateTooltipData.visible}
<div
  class="custom-tooltip"
  style="left: {coordinateTooltipData.x}px; top: {coordinateTooltipData.y}px;"
>
  Longitude: {coordinateTooltipData.longitude} <br> Latitude: {coordinateTooltipData.latitude} <br>

  Day: {coordinateTooltipData.day}, Time: {coordinateTooltipData.hour}:{coordinateTooltipData.minute < 10 ? '0' : ''}{coordinateTooltipData.minute} <br>
</div>
{/if}

<style>
.custom-tooltip {
  position: absolute;
  pointer-events: none;
  background-color: rgba(246, 208, 156, 0.8);
  border: 1px solid #000000; 
  border-radius: 4px;
  padding: 5px; 
  font-size: 14px; 
  font-family: Arial, sans-serif; 
  color: #000000; 
  white-space: normal;
}
</style>