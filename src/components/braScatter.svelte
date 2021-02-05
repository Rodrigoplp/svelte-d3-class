<script>
	import { scaleLinear, scaleLog, scaleSqrt } from 'd3-scale';
  import { axisBottom, axisLeft } from 'd3-axis';
	import { extent } from 'd3-array';
	import { select } from 'd3-selection';

	export let data;
  
  const height = 400;
  const margin = { top: 30, right: 20, bottom: 30, left: 30 };
  
  let width;
  let hozg;
  let vertg;
  let xg;
  let yg;
  let yTransform;
  let xTransform;
  
  $: xScale = scaleLog()
		.domain(extent(data, d => +d.gdp / +d.population))
		.range([margin.left, width - margin.right]);
  
  $: yScale = scaleLinear()
		.domain(extent(data, d => +d.life_expectancy))
		.range([height - margin.bottom, margin.top]);
  
  $: radiusScale = scaleSqrt()
		.domain(extent(data, d => +d.population))
		.range([2, 50]);
  
  $: if (data) {
    // Axis
    xTransform = `translate(0, ${height - margin.bottom})`;
    select(hozg).call(axisBottom(xScale).tickSizeOuter(0));
    select(hozg).selectAll(".tick text").style("font-size", "12px");
    select(hozg).call(g => g.append("text")
      .attr("x", width)
      .attr("y", margin.bottom - 2)
      .attr("fill", "currentColor")
      .attr("text-anchor", "end")
      .style("font-size", "12px")
      .text("GDP per poopulation →"))


    yTransform = `translate(${margin.left}, 0)`;
    select(vertg).call(axisLeft(yScale).ticks(5).tickSize(2));
    select(vertg).selectAll(".tick text").style("font-size", "12px").text(d => d >= 1000 ? d / 1000 + " K" : d);
    select(vertg).call(g => g.append("text")
      .attr("x", -margin.left)
      .attr("y", 10)
      .attr("fill", "currentColor")
      .attr("text-anchor", "start")
      .style("font-size", "12px")
      .text("↑ Life expectancy (years)"))

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
		const radius = select(node).attr('r');
		return {
			duration,
			tick: (t) => select(node).attr('r', t * radius)
		};
	}
</script>

<div class='scatter-plot' bind:clientWidth={width}>
	{#if width}
		<svg width={width} height={height}>
      <g class='x-axis' bind:this={hozg} transform={xTransform} />
      <g class='y-axis' bind:this={vertg} transform={yTransform} />
      <g class="x axis-grid" bind:this={xg} transform={xTransform} />
      <g class="y axis-grid" bind:this={yg} transform={yTransform} />

			{#each data as d}
				<circle
					cx={xScale(+d.gdp / +d.population)}
					cy={yScale(+d.life_expectancy)}
					r={radiusScale(+d.population)}
					fill={d.country === "Brazil" ? "red" : "none"}
          stroke="green"
					in:reveal={{ duration: 1000 }}
				/>
			{/each}
		</svg>
	{/if}

  <p>Source: World Bank</p>
</div>

<style>
	circle {
		opacity: 0.7;
	}

  p {
    text-align: right;
  }
</style>
