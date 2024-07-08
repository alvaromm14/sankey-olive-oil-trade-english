<script>
	import * as d3 from 'd3'
  import { sankey, sankeyJustify, sankeyLinkHorizontal } from 'd3-sankey';
  import { fly } from "svelte/transition";
  import data from "/src/data/data.js";

  const margin = {
    top: 10,
    right: 5,
    bottom: 20,
    left: 5,
  };

  let width = 600;
  let height = 600;
  $: innerWidth = width - margin.left - margin.right;
  $: innerHeight = height - margin.top - margin.bottom;

  $: sankeyGen = sankey()
    .nodeAlign(sankeyJustify)
    .nodeWidth(10)
    .nodePadding(12)
    .nodeId(d => d.id)
    .size([innerWidth, innerHeight])
    .linkSort((a, b) => 
    d3.descending(a.value, b.value))
    .nodeSort((a, b) => 
    d3.descending(a.value, b.value));

	$: dataSankey = sankeyGen(data);
	  
	$: nodes = dataSankey.nodes;
	$: links = dataSankey.links;

  const path = sankeyLinkHorizontal()
    // @ts-ignore
    .source(d => [d.source.x1, d.y0])
    // @ts-ignore
    .target(d => [d.target.x0, d.y1]);

  let hoveredData = [];
  $: suma = Math.floor(hoveredData.reduce((sum, link) => sum + link.value, 0) / 10).toLocaleString('en-EN') + " tonnes";

  function handleMouseOverNode(node) {
    hoveredData = links.filter(link => link.source === node || link.target === node);
  }

  let tooltipWidth;
  let mouseX;
  let mouseY;
  document.addEventListener('mousemove', function(event) {
    mouseX = event.clientX;
    mouseY = event.clientY;
  });

</script>

<div class="chart-container" bind:clientWidth={width}>
  <div class="flujos">
    <p style="color: #497D0B">Origin</p><p style="color: #B3AD10">Destiny</p>
  </div>
  <svg {width} {height} transform="translate({margin.left}, {margin.top})">
    <defs>
      <linearGradient id="gradient" x1="0%" y1="0%" x2="100%" y2="0%">
        <stop offset="0%" style="stop-color:#497D0B; stop-opacity:1" />
        <stop offset="100%" style="stop-color:#CFC912; stop-opacity:1" />
      </linearGradient>
    </defs>
    <g>
      {#each links as d, i}
        <!-- svelte-ignore a11y-mouse-events-have-key-events -->
        <path
        d={path(d)}
        fill='none'
        stroke={i === 8 ? "salmon" : "url(#gradient)"}
        stroke-opacity={hoveredData.length > 0 ? (hoveredData.includes(d) ? 1 : 0.1) : 0.3}
        stroke-width={d.width}
        on:mouseover={() => {
          hoveredData = [d];
        }}
        on:mouseleave={() => {
          hoveredData = [];
        }}/>
      {/each}
    </g>
    <g class='rect-group'>
      {#each nodes as d, i}
        <!-- svelte-ignore a11y-mouse-events-have-key-events -->
        <rect
          x={d.x0}
          y={d.y0}
          height={d.y1 - d.y0}
          width={d.x1 - d.x0}
          fill={i < 9 ? "#497D0B" : "#CFC912"}
          on:mouseover={() => {
            handleMouseOverNode(d);
          }}
          on:mouseleave={() => {
            hoveredData = [];
          }}/>
        <text
          class="country-names"
          x={d.x0 < innerWidth / 4 ? d.x1 + 6 : d.x0 - 6}
          y={(d.y1 + d.y0) / 2}
          dy={5}
          style="text-anchor: {d.x0 < width / 4 ? 'start' : 'end'};
          font-weight: {hoveredData.some(link => link.source.id === d.id || link.target.id === d.id) ? 'bold' : 'normal'};
          font-size: {d.id === "MoroccoOrigen" || d.id === "TurkeyOrigen" || d.id === "ArgentinaOrigen" || d.id === "SyriaOrigen" ? '0.85rem' : 'initial'};">
          {i < 9 ? d.id.slice(0, -6) : d.id}
        </text>
      {/each}
    </g>
  </svg>
  {#if hoveredData.length > 0}
    <div
      transition:fly
      bind:clientWidth={tooltipWidth}
      class="tooltip"
      style="left: {tooltipWidth + mouseX > innerWidth ? mouseX - tooltipWidth - 5 : mouseX + 7}px; top: {mouseY + 12}px">
      <h1>{suma === "0 tonnes" ? "" : suma}</h1>
    </div>
  {/if}
</div>

<style>
  .flujos {
    display: flex;
    justify-content: space-between;
    font-weight: 600;
  }

  .tooltip {
        position: absolute;
        padding: 8px 6px;
        background: white;
        box-shadow: rgba(0, 0, 0, 0.15) 2px 3px 8px;
        border-radius: 3px;
        pointer-events: none;
        transition:
            top 10 ease,
            left 10ms ease;
    }

    .country-names {
      pointer-events: none;
    }

    .degradado-path {
    fill: url(#gradiente); /* Usar el gradiente como relleno */
    stroke: black; /* Color del borde */
}
</style>