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
        // Remove any existing pins
        g.selectAll('.pin').remove();

        g.selectAll('path')
            .style('fill', '#ccc');

        g.selectAll('path')
            .filter(d => matchedCities.some(c => c.lowerCaseName === d.properties.name_fr.toLowerCase()))
            .each(function(d) {
                const centroid = path.centroid(d);
                g.append('svg')
                    .attr('class', 'pin')
                    .attr('x', centroid[0] - 12) // Adjust x position to center the pin
                    .attr('y', centroid[1] - 24) // Adjust y position to center the pin
                    .attr('width', 24)
                    .attr('height', 24)
                    .html(`
                        <svg viewBox="0 0 24 24" width="24" height="24">
                            <path fill="orange" d="M12 2C8.13 2 5 5.13 5 9c0 3.54 3.42 8.71 6.22 12.32.4.54 1.16.54 1.56 0C15.58 17.71 19 12.54 19 9c0-3.87-3.13-7-7-7m0 9.5c-1.38 0-2.5-1.12-2.5-2.5s1.12-2.5 2.5-2.5 2.5 1.12 2.5 2.5-1.12 2.5-2.5 2.5z"></path>
                        </svg>
                    `);
            });
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
    .pin {
        pointer-events: none; /* Ensure pins don't interfere with map interactions */
    }
</style>
