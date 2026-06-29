<script>
	import { onMount } from 'svelte';
	import * as d3 from "d3";
	import rates from '$lib/data/rates.csv';
	import single_federal_tax_amount from '$lib/data/single_federal_tax_amount.json';
	import single_tax_amount from '$lib/data/single_tax_amount.json';
	import married_federal_tax_amount from '$lib/data/married_federal_tax_amount.json';
	import married_tax_amount from '$lib/data/married_tax_amount.json';

	const cities = new Map(rates.map(element => [element.city_id, [element.canton, element.city, element.canton_rate, element.commune_rate, element.splitting_rate]]))
	const projection = d3.geoAlbers().center([0, 46.8]).rotate([-8.2, 0]).parallels([45, 48]).scale(17000).translate([525,350]);
	const path = d3.geoPath().projection(projection);

	export let lakes;
	export let municipalities;
	export let taxable_income;
	export let marital_status;

	let selected;
	let hovered;
	let religion = 'other'
	let allTaxes = [];
	let tax_table;
	let federal_tax_table;

	onMount(async () => {
		drawLegend();
		updateList();
	})

	function calculateTaxes(d) {
		if (marital_status == 'married') {
			federal_tax_table = married_federal_tax_amount
    	tax_table = married_tax_amount
    } else {
    	tax_table = single_tax_amount
    	federal_tax_table = single_federal_tax_amount
    }
	    let index = taxable_income/500
	    let federal_tax = federal_tax_table['tax_amount'][index]

	    let city_id = String(d)
	    if (cities.get(String(city_id)) == undefined) {return parseInt(100)}

	    let canton = cities.get(String(city_id))[0]
	    let city = cities.get(String(city_id))[1]

	    let cantonal_rate = cities.get(String(city_id))[2]
	    let cantonal_tax = tax_table[canton][index] * cantonal_rate/100

	    let municipal_rate = cities.get(String(city_id))[3]
	    let municipal_tax = tax_table[canton][index] * municipal_rate/100

	    let total_taxes = federal_tax + cantonal_tax + municipal_tax
	    let total_tax_rate = total_taxes/taxable_income

	    allTaxes.push([canton, 
	                    city, 
	                    city_id, 
	                    parseInt(federal_tax), 
	                    parseInt(cantonal_tax),
	                    parseInt(municipal_tax),
	                    parseFloat(total_tax_rate*100).toFixed(2), 
	                    parseInt(total_taxes)])

	    return parseInt(total_tax_rate*100)
	}


	const updateMapColor = function(d) {
		if (cities.get(String(d.id)) === undefined) {
		    return d3.interpolateGreys(0.5);
		}
		else {
		    let value = calculateTaxes(d.id)
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
    d3.select(".tooltip").html(function(){
        if (cities.get(String(d.id)) != undefined) {
          let taxes = allTaxes.filter(function(arr) {return arr[2] == d.id})[0]

          let title = cities.get(String(d.id))[1] + " (" + taxes[0] + ")<br>"
          let federal_tax = "Federal Tax: " + taxes[3] + "CHF <br>"
          let cantonal_tax = "Cantonal Tax: " + taxes[4] + "CHF <br>"
          let municipal_tax = "Municipal Tax: " + taxes[5] + "CHF <br>"
          let total_tax = "Total: " + taxes[7] + "CHF " + taxes[6] + "% of your income"

          return title + federal_tax + cantonal_tax + municipal_tax + total_tax
        } else {
          return "Not Available at this moment"
        }
      })
  }

  const updateList = function(d) {
    let canton = "ZH"
    if (d !== undefined){
      canton = cities.get(String(d.id))[0]
    }

    d3.select(".h10_canton").selectAll("*").remove()
    d3.select(".l10_canton").selectAll("*").remove()
    d3.select(".h10_country").selectAll("*").remove()
    d3.select(".l10_country").selectAll("*").remove()

    let h10_canton = (allTaxes.filter(function(d) {return d[0] == canton})).sort(function(a,b) {return b[6] - a[6]}).slice(0,10)
    let l10_canton = (allTaxes.filter(function(d) {return d[0] == canton})).sort(function(a,b) {return a[6] - b[6]}).slice(0,10)
    let h10_country = (allTaxes.sort(function(a,b) {return b[6] - a[6]})).slice(0,10)
    let l10_country = (allTaxes.sort(function(a,b) {return a[6] - b[6]})).slice(0,10)

    for (let item in h10_canton) {
      d3.select(".h10_canton")
        .append("tr").selectAll("td")
        .data([parseInt(item) + 1, h10_canton[item][1], h10_canton[item][0], h10_canton[item][7] + " CHF", h10_canton[item][6] + "%"])
        .enter()
        .append("td")
        .text(function(d){return d})
    }

    for (let item in l10_canton) {
      d3.select(".l10_canton")
        .append("tr").selectAll("td")
        .data([parseInt(item) + 1, l10_canton[item][1], l10_canton[item][0], l10_canton[item][7] + " CHF", l10_canton[item][6] + "%"])
        .enter()
        .append("td")
        .text(function(d){return d})
    }

    for (let item in h10_country) {
      d3.select(".h10_country")
        .append("tr").selectAll("td")
        .data([parseInt(item) + 1, h10_country[item][1], h10_country[item][0], h10_country[item][7] + " CHF", h10_country[item][6] + "%"])
        .enter()
        .append("td")
        .text(function(d){return d})
    }

    for (let item in l10_country) {
      d3.select(".l10_country")
        .append("tr").selectAll("td")
        .data([parseInt(item) + 1, l10_country[item][1], l10_country[item][0], l10_country[item][7] + " CHF", l10_country[item][6] + "%"])
        .enter()
        .append("td")
        .text(function(d){return d})
    }
  }

	export const onCalculate = function(d) {
		allTaxes = [];
		d3.selectAll(".municipality").style("fill", function() {
			return updateMapColor(this)})
		updateList();
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
	<div class="tooltip">Move your mouse over the map! Click on any of the municipality to get more details.</div>
	<svg width="1090" height="700">
		<g class="municipalities">
			{#each municipalities as feature, i}
				<path id={feature.id} d={path(feature)} class="municipality" fill={updateMapColor(feature)} on:mouseover={showTooltip(feature)} on:click={updateList(feature)}/>
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
		<p>
			Disclaimer: The information found here are only an estimation and your actual taxation may differ. This is not to be used for actual tax claims to the Confederation.
		</p>
	</div>
	<table class="styled-table styled-map-1 left-table">
		<caption>Highest Tax Rate within same Canton</caption>
	    <thead>
	        <tr>
	            <th>#</th>
	            <th>City</th>
	            <th>Canton</th>
	            <th>Total Taxes</th>
	            <th>% of income</th>
	        </tr>
	    </thead>
	    <tbody class="h10_canton">
	    </tbody>
	</table>
	<table class="styled-table styled-map-1 right-table">
		<caption>Lowest Tax Rate within same Canton</caption>
	    <thead>
	        <tr>
	            <th>#</th>
	            <th>City</th>
	            <th>Canton</th>
	            <th>Total Taxes</th>
	            <th>% of income</th>
	        </tr>
	    </thead>
	    <tbody class="l10_canton">
	    </tbody>
	</table>
	<table class="styled-table styled-map-1 left-table">
		<caption>Highest Tax Rate within Confederation</caption>
	    <thead>
	        <tr>
	            <th>#</th>
	            <th>City</th>
	            <th>Canton</th>
	            <th>Total Taxes</th>
	            <th>% of income</th>
	        </tr>
	    </thead>
	    <tbody class="h10_country">
	    </tbody>
	</table>
	<table class="styled-table styled-map-1 right-table">
		<caption>Lowest Tax Rate within Confederation</caption>
	    <thead>
	        <tr>
	            <th>#</th>
	            <th>City</th>
	            <th>Canton</th>
	            <th>Total Taxes</th>
	            <th>% of income</th>
	        </tr>
	    </thead>
	    <tbody class="l10_country">
	    </tbody>
	</table>
</div>