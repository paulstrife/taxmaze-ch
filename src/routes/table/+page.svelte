<script>
  import { onMount } from 'svelte';
  import rates from '$lib/data/rates.csv';
  import single_federal_tax_amount from '$lib/data/single_federal_tax_amount.json';
  import single_tax_amount from '$lib/data/single_tax_amount.json';
  import married_federal_tax_amount from '$lib/data/married_federal_tax_amount.json';
  import married_tax_amount from '$lib/data/married_tax_amount.json';

  const cities = new Map(rates.map(element => [element.city_id, [element.canton, element.city, element.canton_rate, element.commune_rate, element.splitting_rate]]))

  export let taxable_income = 100000;
  export let marital_status = 'single';

  let allTaxes = [];
  let tax_table;
  let federal_tax_table;
  let municipalities_list = [];
  let selected;
  let table_order = 0;

  let cantons = ['All','SO','JU','NW','ZG','OW','AI','GE','LU','VD','GL','ZH','TI','SZ','GR','VS','NE','AR','FR','SG','AG','BE','BS','BL','UR','SH','TG']

  onMount(async () => {
    cities.forEach(function(key, value){
      calculateTaxes(value)
    })
      updateList("All");
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

  const updateList = function() {
    if (table_order == 0) {
      allTaxes.sort(function(a,b) {return a[6] - b[6]})
    } else {
      allTaxes.sort(function(a,b) {return b[6] - a[6]})
    }
    if (selected == "All") {
      municipalities_list = allTaxes
    } else {
      municipalities_list = allTaxes.filter(function(d) {return d[0] == selected})
    }
  }

  export const onCalculate = function() {
    allTaxes = [];
    cities.forEach(function(key, value){
      calculateTaxes(value)
    })
      updateList();
  }
</script>

<svelte:head>
  <title>TaxMaze - Table View</title>
</svelte:head>

<main>
  <div class="tables">
    <div class="disclaimer">
      <p>
        Disclaimer: The information found here are only an estimation and your actual taxation may differ. This is not to be used for actual tax claims to the Confederation.
      </p>
    </div>
    <div class="input-table-view">
      <form style="margin-block-end:0;" on:submit|preventDefault={onCalculate}>
        <label for="taxable-income">Taxable Income:</label>
        <input type="number" min="500" max="1000000" step="500" placeholder="CHF" bind:value={taxable_income} class="input-text-round taxable-income">
        <label for="canton">Canton</label>
        <select bind:value={selected} on:change={() => updateList()}>
          {#each cantons as canton}
            <option value={canton}>{canton}</option>
          {/each}
        </select>
        <label for="marital-status">Marital Status:</label>
        <input type="radio" name="marital-status" bind:group={marital_status} value="single" checked>
        <label for="single">Single</label>
        <input type="radio" name="marital-status" bind:group={marital_status} value="married">
        <label for="married">Married</label>
        <button type="submit" class="submitIncome">Calculate</button>
      </form>
    </div>
    <table class="styled-table styled-table-view" style="width: 100%;">
  <thead>
<tr>
    <th>#</th>
    <th>City</th>
    <th>Canton</th>
    <th on:click={() => {table_order === 0 ? table_order = 1 : table_order = 0; updateList()}}>Federal Tax ≡</th>
    <th on:click={() => {table_order === 0 ? table_order = 1 : table_order = 0; updateList()}}>Cantonal Tax ≡</th>
    <th on:click={() => {table_order === 0 ? table_order = 1 : table_order = 0; updateList()}}>Municipal Tax ≡</th>
    <th on:click={() => {table_order === 0 ? table_order = 1 : table_order = 0; updateList()}}>Total Taxes ≡</th>
    <th on:click={() => {table_order === 0 ? table_order = 1 : table_order = 0; updateList()}}>% of income ≡</th>
</tr>
  </thead>
  <tbody class="municipalities_list">
    {#each municipalities_list as municipality, i}
    <tr>
      <td>{i + 1}</td>
      <td>{municipality[1]}</td>
      <td>{municipality[0]}</td>
      <td>{municipality[3] + " CHF"}</td>
      <td>{municipality[4] + " CHF"}</td>
      <td>{municipality[5] + " CHF"}</td>
      <td>{municipality[7] + " CHF"}</td>
      <td>{municipality[6] + "%"}</td>
      </tr>
    {/each}
  </tbody>
    </table>
</main> 