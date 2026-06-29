<script>
</script>

<svelte:head>
  <title>TaxMaze - About</title>
</svelte:head>

<p style="text-align: center;">
  This site was developed with HTML/CSS and JavaScript using the Svelte framework. Visualization is done with the <a href="https://d3js.org">D3.js</a> and <a href="https://github.com/topojson/topojson">TopoJSON</a> libraries.
  <br>
  Tax information were extracted from the <a href="https://swisstaxcalculator.estv.admin.ch/#/home">Federal Tax Administration</a> of Switzerland website.
</p>
<p style="text-align: center;">
  I hope you appreciated this site. For any inquiry, feedback, bug report please feel free to reach out to <a href="mailto:info@taxmaze.ch">info@taxmaze.ch</a> or open an issue at <a href="https://github.com/paulstrife/taxmaze-ch">GitHub</a>.
</p>
<h3 style="text-align: center;">Release Notes</h3>
<p style="text-align: center;">
  <strong>v3.3</strong>: Updated with 2023 tax rates for all cantons.<br>Updated map with 2024 topographic information.
</p>
<hr style="border-top: 1px dotted; width: 50%;">
<p style="text-align: center;">
  <strong>v3.2</strong>: Updated with 2023 tax rates for ordinary taxation (GE, BS and NE are still based on 2022 pending data to be released by the canton).<br>Updated with 2024 tax rates for source taxation. Updated to ordinary/source taxation naming.
  <br>Started partnership with <a href="https://mm-tax.ch/">M&M Tax</a> to offer tax advice.
  <br>Some municipalities are not showing up correctly pending release of 2024 topographic information.
</p>
<hr style="border-top: 1px dotted; width: 50%;">
<p style="text-align: center;">
  <strong>v3.1</strong>: Added back table view of taxes for Switzerland citizens/C permit holders.
</p>
<hr style="border-top: 1px dotted; width: 50%;">
<p style="text-align: center;">
  <strong>v3</strong>: Moved to Svelte framework. Added marital status option. Added map for B/L permit holders.
</p>
<hr style="border-top: 1px dotted; width: 50%;">
<p style="text-align: center;">
  <strong>v2</strong>: Fixed tax calculation for Basel-Landschaft, Fribourg and Valais cantons. Updated tax rates to 2022 data.
</p>
<hr style="border-top: 1px dotted; width: 50%;">
<p style="text-align: center;">
  <strong>v1</strong>: Initial release of TaxMaze.
</p>