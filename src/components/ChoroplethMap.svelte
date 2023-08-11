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
  let statesFeatures = [];

  // ADJUST COLOR BASED ON LOCATION RATIO
  let scale = scaleLinear<string, string>()
    .domain([0, 1])
    .range(["#FFFFFF", "#FFFFFF"]);

  let ratios = new Map();

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
    // rollup() method to calculate the total count for each state
    ratios = rollup(
      data,
      ([v1, v2]) => v1.count / v2.count, // ratio of the two counts, first being Dunks, second being Starbucks. The higher the ratio over 1, the more Dunks there are in that state. The lower the ratio under 1, the more Starbucks there are in that state. If the ratio is 1, there are equal amounts of Dunks and Starbucks in that state.
      (d) => d.state
    );

    // extent() method to calculate the minimum and maximum ratio values. Get the maximum location ratio from ratios via D3’s extent() method. Since the method only accepts an array as an argument, you will need to first convert the map to an array via Array.from().
    const [, max] = extent(Array.from(ratios.values()), (ratio) => ratio);

    // Then, set scale to a linear scale that maps the ratios to colors passed into the colors prop. The max value corresponds to the first color in the colors list, an orange color.

    scale = scaleLinear<string, string>()
      .domain([max, (max - 1) / 2, 1, 0.5, 0]) // 0.5 is the middle value, so it will be the middle color in the colors list, which is a brown color. 1 is the last value, so it will be the last color in the colors list, which is a green color. 0 is the first value, so it will be the first color in the colors list, which is a white color.
      .range(colors);
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
    {#each statesFeatures as feature}
      <path
        fill={scale(ratios.get(feature.properties.name))}
        d={path(feature)}
      />
    {/each}
  </g>
</svg>
