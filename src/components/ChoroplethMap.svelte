<script lang="ts">
  import { extent, rollup } from "d3-array";
  import { csv, json } from "d3-fetch";
  import { geoPath } from "d3-geo";
  import { scaleLinear } from "d3-scale";
  import { type UsAtlas, mesh, feature } from "topojson";
  import { onMount } from "svelte";

  export let datasets = [];
  export let colors = [];

  // MAP RENDERING
  // scale and dimensions for the map
  let dimensions = {
    width: 975,
    height: 720,
    margin: {
      top: 24,
      right: 0,
      left: 0,
      bottom: 6,
    },
  };

  // generate the path geometry
  const path = geoPath();

  // DATA FETCHING

  let stateMesh = null;
  let statesFeatures = null;

  //  Fetching CSV Data with d3-fetch's d3.csv() Method in Svelte’s onMount Lifecycle Method

  // in a svelte component, we need to use the onMount lifecycle method to fetch data after the component has loaded.

  onMount(async () => {
    // parse count from csv as a number, map through both csvs
    // Map datasets to promises that return the parsed CSV data.
    // Run the promises with Promise.all.

    const counts = await Promise.all(
      datasets.map(
        async ({ url }) =>
          await csv(url, ({ state, count }) => ({
            state,
            count: +count,
          }))
      )
    );

    // combine the two arrays into one array called data
    const data = [...counts[0], ...counts[1]];

    // d3-fetch comes with a convenient method for fetching and parsing JSON files: json().
    const us: UsAtlas = await json("/data/us_topojson.json");

    // Convert state meshes to equivalent GeoJSON MultiLineString geometry objects via topojson‘s .mesh() method. The mesh method takes the TopoJSON object, the object to convert, and an optional comparator function. The comparator function is used to determine whether two arcs should be stitched together. In this case, we want to stitch together all arcs that are not equal to each other. This will create a MultiLineString geometry object for each state.
    stateMesh = mesh(us, us.objects.states, (a, b) => a !== b);

    statesFeatures = feature(us, us.objects.states).features;
  });
</script>

<svg
  width={dimensions.width}
  height={dimensions.height}
  viewBox={`0 0 ${dimensions.width} ${dimensions.height}`}
>
  <g>
    <path
      fill="none"
      stroke="white"
      stroke-linejoin="round"
      d={path(stateMesh)}
    />
    <!-- This is broken in the tutorial. The first render, it will display an error because the statesFeatures isn't loaded yet. We can add in the if statement to check if it is loaded so it doesn't fail on the first render. -->
    {#if statesFeatures && Array.isArray(statesFeatures)}
      {#each statesFeatures as feature}
        <path fill="green" d={path(feature)} />
      {/each}
    {/if}
  </g>
</svg>
