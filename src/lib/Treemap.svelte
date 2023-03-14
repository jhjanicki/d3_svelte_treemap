<script>
  import data from "../data/data.json";
  import * as d3 from "d3";
  const length = 750;

  //legend data (if desired) and color scale
  const legendData = [
    { category: "Critically Endangered", color: "#b30000" },
    { category: "Endangered", color: "#e34a33" },
    { category: "Vulnerable", color: "#fc8d59" },
    { category: "Near Threatened", color: "#fdbb84" },
    { category: "Least Concern", color: "#fdd49e" },
    { category: "Data Deficient", color: "#d9d9d9" },
  ];
  const categories = legendData.map((d) => d.category);
  const colors = legendData.map((d) => d.color);
  const colorScale = d3.scaleOrdinal().domain(categories).range(colors);

  // sort data based on the categories array so the order would go from most to least threatened
  data.sort(
    (a, b) =>
      categories.indexOf(a.redlistCategory) -
      categories.indexOf(b.redlistCategory)
  );

  //convert data to hierarchical format
  const groupingFn = [(d) => d.redlistCategory]; //can group by multiple attributes here
  const rollupData = d3.rollup(data, (v) => v.length, ...groupingFn);
  const childrenAccessorFn = ([key, value]) => value.size && Array.from(value);
  const hierarchyData = d3
    .hierarchy(rollupData, childrenAccessorFn)
    .sum(([key, value]) => value);

  // Layout + data prep
  const treemap = d3
    .treemap()
    .paddingInner(2)
    .paddingOuter(1)
    .paddingTop(1)
    .round(true)
    .size([length, length])
    .tile(d3.treemapResquarify.ratio(2));

  const leaves = treemap(hierarchyData)
    .leaves()
    .sort((a, b) => b.data[0] - a.data[0]);
</script>

<svg width={length} height={length}>
  {#each leaves as d, i}
    <rect
      transform={`translate(${d.x0},${d.y0})`}
      width={d.x1 - d.x0}
      height={d.y1 - d.y0}
      stroke-width="0.5"
      fill={colorScale(d.data[0])}
    />
    <text x={d.x0 + 3} y={d.y0 + 15}>{`${d.data[0]}: ${d.value}`}</text>
  {/each}
</svg>

<style>
  text {
    font-weight: 700;
    font-size: 14px;
    paint-order: stroke;
    stroke: white;
    stroke-width: 2px;
    stroke-linecap: butt;
    stroke-linejoin: miter;
  }
</style>
