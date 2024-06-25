<script>
	import { onMount } from 'svelte';
	import * as d3 from 'd3';
	import { geoPath, geoMercator } from 'd3-geo';
	import { feature } from 'topojson-client';
	import mapDataJson from '../../public/data/map.json';
  
	export let matchedCities = [];
  
	let svg;
	let g;
	let projection;
	let path;
	let mapData;
  
	onMount(() => {
	  const width = 800; // Base width for projection
	  const height = 600; // Base height for projection
  
	  svg = d3.select('#map')
		.attr('viewBox', `0 0 ${width} ${height}`)
		.attr('preserveAspectRatio', 'xMidYMid meet');
  
	  const topoObjectKey = Object.keys(mapDataJson.objects)[0];
	  mapData = feature(mapDataJson, mapDataJson.objects[topoObjectKey]).features;
  
	  projection = geoMercator()
		.center([4.5, 50.5])
		.scale(10000)
		.translate([width / 2, height / 2]);
  
	  path = geoPath().projection(projection);
  
	  g = svg.append('g');
  
	  g.selectAll('path')
		.data(mapData)
		.enter()
		.append('path')
		.attr('d', path)
		.attr('class', 'region')
		.style('fill', '#ccc')
		.style('stroke', '#333')
		.style('stroke-width', '0.5px');
  
	  highlightCities();
	});
  
	$: if (matchedCities.length) {
	  highlightCities();
	}
  
	function highlightCities() {
	  g.selectAll('path')
		.style('fill', d => (matchedCities.some(c => c.lowerCaseName === d.properties.name_fr.toLowerCase()) ? 'orange' : '#ccc'));
	}
</script>
  
<svg id="map"></svg>
  
<style>
	.region {
	  transition: fill 0.2s;
	}
	#map {
	  width: 100%; /* Ensure map takes full width */
	  height: 100%; /* Maintain aspect ratio */
	  display: block;
	}
</style>
