<script>
  import { onMount } from 'svelte';
  import * as topojson from 'topojson-client';
  import * as d3 from "d3";

  import swiss from '$lib/data/geodata/ch-combined.json';


  import Map_swiss from './Map_swiss.svelte';
  import Map_b_or_l from './Map_b_or_l.svelte';
  import Map_difference from './Map_difference.svelte';

  const ch = swiss;
  let cantons = topojson.feature(ch, ch.objects.cantons).features;
  let lakes = topojson.feature(ch, ch.objects.lakes).features;
  let municipalities = topojson.feature(ch, ch.objects.municipalities).features;
  let taxable_income = 100000;
  let marital_status = 'single';
  let activeTab;
  let onCalculate;
  let map1;
  let map2;
  let map3;

  onMount(async () => {
    activeTab = "map1"
  })
</script>

<svelte:head>
  <title>TaxMaze</title>
</svelte:head>

<main>
  <nav class="tab">
    <button class="tablinks" on:click={() => {activeTab = 'map1'; map1.onCalculate()}} class:active={activeTab === 'map1'}>Ordinary Taxation<br><small>(e.g. Swiss citizens/C permit)</small></button>
    <button class="tablinks" on:click={() => {activeTab = 'map2'; map2.onCalculate()}} class:active={activeTab === 'map2'}>Source Taxation<br><small>(e.g. B or L permit holders)</small></button>
    <button class="tablinks" on:click={() => {activeTab = 'map3'}} class:active={activeTab === 'map3'}>What's the difference?<br><br></button>
  </nav>
  <section class="map-container">
    <div class="map {activeTab === 'map1' ? 'active' : ''}"><Map_swiss bind:municipalities bind:lakes bind:taxable_income bind:marital_status bind:this={map1}/></div>
    <div class="map {activeTab === 'map2' ? 'active' : ''}"><Map_b_or_l bind:cantons bind:lakes bind:taxable_income bind:marital_status bind:this={map2}/></div>
    <div class="map {activeTab === 'map3' ? 'active' : ''}"><Map_difference/></div>
  </section>
</main>