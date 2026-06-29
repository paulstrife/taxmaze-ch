<script>
	import { onMount } from 'svelte';
	import * as d3 from "d3";
	import swiss from '$lib/data/geodata/ch-combined.json';
	import single_source_tax_amount from '$lib/data/single_source_tax_amount.json';
	import married_source_tax_amount from '$lib/data/married_source_tax_amount.json';

	const projection = d3.geoAlbers().center([0, 46.8]).rotate([-8.2, 0]).parallels([45, 48]).scale(17000).translate([525,350]);
	const path = d3.geoPath().projection(projection);
;
	export let cantons;
	export let lakes;
	export let taxable_income;
	export let marital_status;

	let selected;
	let hovered;
	let tax_table;
	let allTaxes = [];
	let cantons_dict = {11: ['SO', 'Solothurn'], 26: ['JU','Jura'], 7: ['NW','Nidwalden'], 9: ['ZG', 'Zug'], 
                      6: ['OW', 'Obwalden'], 16: ['AI', 'Appenzell Innerrhoden'], 25: ['GE', 'Geneva'],
                      3: ['LU', 'Luzern'], 22: ['VD', 'Vaud'], 8: ['GL', 'Glarus'], 1: ['ZH', 'Zürich'], 
                      21: ['TI', 'Ticino'], 5: ['SZ', 'Schwyz'], 18: ['GR', 'Graubünden'], 23: ['VS', 'Valas'],
                      24: ['NE', 'Neuchâtel'], 15: ['AR', 'Appenzell Ausserrhoden'], 10: ['FR', 'Freiburg'], 17: ['SG', 'Sankt Gallen'], 
                      19: ['AG', 'Aargau'], 2: ['BE', 'Bern'], 12: ['BS', 'Basel Stadt'], 13: ['BL', 'Basel Land'],
                      4: ['UR', 'Uri'], 14: ['SH', 'Schaffhausen'], 20: ['TG', 'Thurgau']}


	onMount(async () => {
		drawLegend();
		updateList();
	})

	function calculateTaxes(d) {
    if (marital_status == 'married') {
      tax_table = married_source_tax_amount
    } else {
      tax_table = single_source_tax_amount
    }

    let index = taxable_income/500
    let canton_id = d.id
    let canton = cantons_dict[canton_id][0]

    let total_taxes = tax_table[canton][index]
    let total_tax_rate = total_taxes/taxable_income

    allTaxes.push([canton,canton_id,parseInt(total_taxes),parseFloat(total_tax_rate*100).toFixed(2)])

    return parseInt(total_tax_rate*100)
  }

  const updateMapColor = function(d) {
  	if (cantons_dict[d.id][0] === undefined) {
      return d3.interpolateGreys(0.5);
    }
    else {
      let value = calculateTaxes(d);
      return d3.interpolateGreens(value/40);
    }
  }

	const drawLegend = function()  {
		const legend = d3.selectAll("svg").append("g");
		legend.style("transform", "translate(42%, 95%)")

		var linearScale = d3.scaleLinear().domain([0, 100]).range([0, 500]);
		var sequentialScale = d3.scaleSequential().interpolator(d3.interpolateGreens).domain([0, 40]);
		const axis = d3.axisBottom(d3.scaleLinear().domain([0,40]).range([0,199]))
		      .tickFormat((d,i) => ["0%","5%","10%","15%","20%","25%","30%","35%","40%>"][i]); 

		var data_range = d3.range(0, 40, 2);

		legend
		.selectAll('rect')
		.data(data_range)
		.join('rect')
		.attr('width', 10)
		.attr("height", 10)
		.attr('x', function(d) {return linearScale(d)})
		.style('fill', function(d) {return sequentialScale(d)});

		legend.append("g")
		  .attr("transform", "translate(0.5,15)")
		  .call(axis)
		  .select(".domain")
		  .attr("visibility", "hidden");
	 }

  const showTooltip = function(d) {
    d3.select(".b_or_l").html(function(){
        let taxes = allTaxes.filter(function(arr) {return arr[1] == d.id})[0]
        let title = cantons_dict[d.id][1] + " (" + taxes[0] + ") <br>"
        let total_tax = "Total: " + taxes[2] + "CHF " + taxes[3] + "% of your income"

        return title + total_tax
      })
    d3.select(this).raise()
  }

  const updateList = function() {
    d3.select(".h10_borl").selectAll("*").remove()
    d3.select(".l10_borl").selectAll("*").remove()

    let h10_canton = allTaxes.sort(function(a,b) {return b[3] - a[3]}).slice(0,13)
    let l10_canton = allTaxes.sort(function(a,b) {return a[3] - b[3]}).slice(0,13)

    for (let item in h10_canton) {
      d3.select(".h10_borl")
        .append("tr").selectAll("td")
        .data([parseInt(item) + 1, h10_canton[item][0], h10_canton[item][2] + " CHF", h10_canton[item][3] + "%"])
        .enter()
        .append("td")
        .text(function(d){return d})
    }

    for (let item in l10_canton) {
      d3.select(".l10_borl")
        .append("tr").selectAll("td")
        .data([parseInt(item) + 1, l10_canton[item][0], l10_canton[item][2] + " CHF", l10_canton[item][3] + "%"])
        .enter()
        .append("td")
        .text(function(d){return d})
    }
  }

	export const onCalculate = function(d) {
		allTaxes = [];
		d3.selectAll(".canton").style("fill", function() {
			return updateMapColor(this)})
		updateList()
	}
</script>

<div class="input">
	<form style="margin-block-end:0;" on:submit|preventDefault={onCalculate}>
		<label for="taxable-income">Taxable Income:</label><br>
		<input type="number" min="500" max="1000000" step="500" placeholder="CHF" bind:value={taxable_income} class="input-text-round taxable-income"><br>
		<label for="marital-status">Marital Status:</label><br>
		<input type="radio" name="marital-status" bind:group={marital_status} value="single" checked>
		<label for="single">Single</label>
		<input type="radio" name="marital-status" bind:group={marital_status} value="married">
		<label for="married">Married</label><br>
		<button type="submit" class="submitIncome">Calculate</button>
	</form>
</div>

<div class="tooltip b_or_l">Move your mouse over the map! Click on any of the municipality to get more details.</div>

<svg width="1090" height="700">
	<g class="cantons">
		{#each cantons as feature, i}
			<path id={feature.id} d={path(feature)} class="canton" fill={updateMapColor(feature)} on:mouseover={showTooltip(feature)}/>
		{/each}
	</g>
		
	<g class="lakes">
		{#each lakes as feature, i}
			<path d={path(feature)} class="lake" />
		{/each}
	</g>
</svg>

  <div class="tables">
    <div class="disclaimer">
      <p>Disclaimer: The information found here are only an estimation and your actual taxation may differ. This is not to be used for actual tax claims to the Confederation.</p>
    </div>
    <table class="styled-table styled-table-map-2 left-table">
      <caption>Highest Tax Rate within Confederation</caption>
      <thead>
        <tr><th>#</th><th>Canton</th><th>Total Taxes</th><th>% of income</th></tr>
      </thead>
      <tbody class="h10_borl h10_canton"></tbody>
    </table>
    <table class="styled-table styled-table-map-2 right-table">
      <caption>Lowest Tax Rate within Confederation</caption>
        <thead>
          <tr>
            <th>#</th><th>Canton</th><th>Total Taxes</th><th>% of income</th>
          </tr>
        </thead>
        <tbody class="l10_borl l10_canton"></tbody>
    </table>
  </div>