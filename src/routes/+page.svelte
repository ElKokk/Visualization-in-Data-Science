<script>
  import { onMount } from 'svelte';
  import Map from './Map.svelte';
  import CarDetails from './CarDetails.svelte';
  

  let gpsCoordinates;
  let businesses;
  let carStops;
  let selectedCar = '';
  let showDetails = false;
  let isOverviewPage = true;
  

  let carIds = [];

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

    gpsCoordinates = await fetchGpscoordinates();
    carIds = Array.from(new Set(gpsCoordinates.map((d) => d.car_id)));
    businesses = await fetchbusinesses();
    carStops = await fetchcarstops();
  });
</script>


{#if !showDetails}
  <h2 class="display-20">Eleftherios Kokkinis - UHasselt - 2262635</h2>
  {#if isOverviewPage}
  <h3 class="display-9">Overview</h3>
  {:else}
  <h2 class="display-5">Details Page</h2>
  {/if}
  <p class="my-3">Select car to highlight:
    <select class="form-select-sm bg-primary text-light" bind:value={selectedCar}>
      <option value="">Select a car</option>
      {#each carIds as carId}
        <option value={carId}>Car {carId}</option>
      {/each}
    </select>
  </p>

  {#if selectedCar}
    <p class="my-3">Go to <button class="btn btn-primary" on:click={() => (showDetails = true)}>details</button> for car {selectedCar}</p>
  {/if}
{/if}

{#if showDetails}
  <CarDetails
  carId={selectedCar}
  carIds={carIds}
  data={gpsCoordinates.filter(item => item.car_id == selectedCar)}
  businesses={businesses}
  on:closeDetails={() => (showDetails = false)}
  on:changeCar={(e) => {
  showDetails = false;
  selectedCar = e.detail.carId;
  showDetails = true;
  }}
  />
{:else}
  <Map key={selectedCar} selectedCar={selectedCar} detailsPage={false} />
{/if}