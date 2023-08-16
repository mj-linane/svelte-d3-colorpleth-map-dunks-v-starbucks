<script lang="ts">
  import ChoroplethMap from "./components/ChoroplethMap.svelte";

  let datasets = [
    {
      label: "Dunkin' Donuts",
      url: "/data/dunkin_donuts_locations_counts.csv",
    },
    { label: "Starbucks", url: "/data/starbucks_locations_counts.csv" },
  ];

  let colors = ["#ea4f19", "#b15222", "#75542a", "#3c5733", "#005739"];
</script>

<section>
  <h1>
    Brewed Maps: A Choropleth Blend of Dunkin' & Starbucks Across the States
  </h1>
  <p>
    Sip your way through the U.S. with this D3.js choropleth map, showcasing the
    ratio of Dunkin' Donuts to Starbucks locations. A tasteful blend of data
    visualization and coffee culture, served in a Svelte cup.
  </p>
</section>
<main>
  <ChoroplethMap {datasets} {colors} />
</main>

<style>
  main {
    font-family: "Helvetica";
    display: flex;
    justify-content: center;
  }

  section {
    max-width: 800px;
    margin: 0 auto;
    padding: 0 1rem;
    font-family: "Helvetica";
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    text-align: center;
  }
</style>
