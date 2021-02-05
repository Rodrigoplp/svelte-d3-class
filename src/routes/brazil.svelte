<svelte:head>
  <title>Brazil</title>
</svelte:head>

<script>
  import { onMount } from "svelte";
  import { json, csv } from "d3-fetch";
  import BrazilLineChart from "../components/braLine.svelte";
  import BrazilScatterChart from "../components/braScatter.svelte";

  let gdpData;
  let gdpWldData;
  let expData;
  let worldData;

  // Load data
  onMount(async () => {
    gdpData = await json("gdp_bra.json");
    gdpWldData = await json("gdp_wld.json");
    expData = await json("exp_bra.json");
    worldData = await csv("world_bank.csv");
  })
</script>

<h1>Brazil</h1>

{#if gdpData && gdpWldData && worldData}
  <BrazilLineChart data={gdpData} wData={gdpWldData} eData={expData} />
  <BrazilScatterChart data={worldData} />
{/if}

<style>
  h1 {
    text-align: center;
  }
</style>
