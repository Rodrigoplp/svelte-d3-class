<script>
  import { scaleTime, scaleLinear } from 'd3-scale';
  import { extent, max, min } from 'd3-array';
  import { line } from 'd3-shape';
  import { axisBottom, axisLeft, axisRight } from 'd3-axis';
  import { select, selectAll } from 'd3-selection';

  export let data = [];
  export let wData = [];
  export let eData = [];

  const height = 400;
  const margin = { top: 30, right: 20, bottom: 30, left: 30 };
  let width;
  let hozg;
  let vertg;
  let vertg2;
  let xg;
  let yg;
  let xTransform;
  let yTransform;
  let yTransform2;

  $: xScale = scaleTime()
    .domain(extent(data, d => new Date(d.date)))
    .range([margin.left, width - margin.right]);

  $: yScale = scaleLinear()
    .domain([0, max(data, d => +d.value)])
    .range([height - margin.bottom, margin.top]);

  $: yScale2 = scaleLinear()
    .domain([min(eData, d => +d.value), max(eData, d => +d.value)])
    .range([height - margin.bottom, margin.top]);

  $: lineGenerator = line()
      .x(d => xScale(new Date(d.date)))
      .y(d => yScale(+d.value));

  $: lineGenerator2 = line()
      .x(d => xScale(new Date(d.date)))
      .y(d => yScale2(+d.value));

  $: if (data && wData && eData) {
    // Axis
    xTransform = `translate(0, ${height - margin.bottom})`;
    select(hozg).call(axisBottom(xScale).tickSizeOuter(0));
    select(hozg).selectAll(".tick text").style("font-size", "12px");

    yTransform = `translate(${margin.left}, 0)`;
    select(vertg).call(axisLeft(yScale).ticks(5).tickSize(2));
    select(vertg).selectAll(".tick text").style("font-size", "12px").text(d => d >= 1000 ? d / 1000 + " K" : d);

    yTransform2 = `translate(${width - margin.right}, 0)`;
    select(vertg2).call(axisRight(yScale2).ticks(5).tickSize(2));
    select(vertg2).selectAll(".tick text").style("font-size", "12px");

    // Gridlines
    // Horizontal grid
    select(xg).call(
      axisBottom(xScale)
        .tickFormat("")
        .ticks(5)
        .tickSize(margin.top + margin.bottom - height)
    );
    select(xg).select(".domain").remove();
    select(xg).selectAll(".tick").select("line").style("stroke", "lightgrey");

    // Vertical grid
    select(yg).call(
      axisLeft(yScale)
        .tickFormat("")
        .ticks(5)
        .tickSize(margin.left + margin.right - width)
    );
    select(yg).select(".domain").remove();
    select(yg).selectAll(".tick").select("line").style("stroke", "lightgrey");
  }

  const reveal = (node, { duration }) => {
    if (!xScale || !yScale) return;
    const length = node.getTotalLength();
    node.style.strokeDasharray = length;
    return {
      duration,
      css: (t, u) => `stroke-dashoffset: ${u * length}`
    };
  }
</script>

<div class='line-chart' bind:clientWidth={width}>
  <h2>GDP per capita</h2>

  {#if width}
    <svg width={width} height={height}>
      <g class='x-axis' bind:this={hozg} transform={xTransform} />
      <g class='y-axis' bind:this={vertg} transform={yTransform} />
      <g class='y-axis' bind:this={vertg2} transform={yTransform2} />
      <g class="x axis-grid" bind:this={xg} transform={xTransform} />
      <g class="y axis-grid" bind:this={yg} transform={yTransform} />

      <path
        d={lineGenerator2(eData)}
        stroke='green'
        stroke-width={1}
        stroke-linecap='round'
        fill='papayawhip'
        opacity=0.5
        in:reveal={{ duration: 2000 }}
      />

      <path
        d={lineGenerator(data)}
        stroke='blue'
        stroke-width={2}
        stroke-linecap='round'
        fill='none'
        in:reveal={{ duration: 2000 }}
      />

      <path
        d={lineGenerator(wData)}
        stroke='red'
        stroke-width={2}
        stroke-linecap='round'
        fill='none'
        in:reveal={{ duration: 2000 }}
      />

    </svg>

    <div class="legend">
      <span class="circle red"></span><span>GDP World</span>
      <span class="circle blue"></span><span>GDP Brazil</span>
      <span class="circle green"></span><span>Export balance Brazil</span>
    </div>
  {/if}

  <p>Source: World Bank</p>
</div>

<style>
  h2 {
    text-align: center;
  }

  p {
    text-align: right;
  }

  .legend {
    display: table;
    margin: auto;
  }

  span {
    margin: 0 5px;
  }

  .circle:before {
    content: ' \25CF';
    font-size: 14px;
  }

  .circle.red {
    color: red;
  }

  .circle.blue {
    color: blue;
  }

  .circle.green {
    color: papayawhip;
  }
</style>
