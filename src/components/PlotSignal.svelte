<script>
  import * as d3 from 'd3';
  import { onMount, afterUpdate  } from 'svelte';
  import { writable } from 'svelte/store';

  export let dataStoreSignal;

  let time = [];
  let signal = [];

  dataStoreSignal.subscribe(data => {
      // Update local time and signal arrays
      time = data.map(d => d.time);
      signal = data.map(d => d.signal);

});
  
  // Define width, height, etc.
  let width = 900;
  let height = 500;
  let marginTop = 20;
  let marginRight = 80;
  let marginBottom = 30;
  let marginLeft = 80;

  let xScale;
  let yScale;

  let tooltip = null;

// Function to initialize scales
function initializeScales() {
  xScale = d3.scaleLinear()
    .domain([d3.min(time), d3.max(time)])
    .range([marginLeft, width - marginRight]);

  yScale = d3.scaleLinear()
    .domain([d3.min(signal) - 1, d3.max(signal) + 1])
    .range([height - marginBottom, marginTop]);
}

// Initialize scales when component is created or when data changes
$: {
    if (signal.length > 0) {
        initializeScales();
    }
}
const line = d3
  .line()
  .x((d) => xScale(d.time))
  .y((d) => yScale(d.signal));

function showTooltip(event) {
  const [x, y] = d3.pointer(event);
  const nearestDataPoint = findNearestDataPoint(x, y);

  // Add null check before accessing properties
  if (nearestDataPoint) {
    tooltip = {
      visible: true,
      x: x,
      y: y,
      content: `${nearestDataPoint.time.toFixed(2)}, ${nearestDataPoint.signal.toFixed(2)}`,
    };
  }
}

function hideTooltip() {
  tooltip = null;
}

function findNearestDataPoint(x, y) {
  let minDistance = 5;
  let nearestDataPoint = null;

  dataStoreSignal.subscribe(data => {
    data.forEach((d) => {
      const xPos = xScale(d.time);
      const yPos = yScale(d.signal);
      const distance = Math.sqrt((x - xPos) ** 2 + (y - yPos) ** 2);

      if (distance < minDistance) {
        minDistance = distance;
        nearestDataPoint = d;
      }
    });
  });

  return nearestDataPoint;
}
</script>

<main>

<svg
  {width}
  {height}
  viewBox={`0 0 ${width} ${height}`}
  style:max-width="100%"
  style:height="auto"
  role="img"
  on:mousemove={showTooltip}
  on:mouseleave={hideTooltip}
>
  <!-- X-Axis -->
  <g transform={`translate(0,${height - marginBottom})`}>
    <line stroke="currentColor" x1={marginLeft - 6} x2={width} />

    {#each xScale.ticks() as tick}
      <line
        stroke="currentColor"
        x1={xScale(tick)}
        x2={xScale(tick)}
        y1={0}
        y2={6}
      />

      <text fill="currentColor" text-anchor="middle" x={xScale(tick)} y={22}>
        {tick.toFixed(2)}
      </text>
    {/each}
  </g>

  <!-- Y-Axis and Grid Lines -->
  <g transform={`translate(${marginLeft},0)`}>
    {#each yScale.ticks() as tick}
      {#if tick !== 0}
        <line
          stroke="currentColor"
          stroke-opacity="0.1"
          x1={0}
          x2={width - marginLeft}
          y1={yScale(tick)}
          y2={yScale(tick)}
        />

        <line
          stroke="currentColor"
          x1={0}
          x2={-6}
          y1={yScale(tick)}
          y2={yScale(tick)}
        />
      {/if}

      <text
        fill="currentColor"
        text-anchor="end"
        dominant-baseline="middle"
        x={-9}
        y={yScale(tick)}
      >
        {tick.toFixed(2)}
      </text>
    {/each}

    <text fill="currentColor" text-anchor="start" x={-marginLeft} y={15}>
      ↑ Signal Values
    </text>
  </g>

  <!-- Line Path with Hover Effects -->
  <path
    fill="none"
    stroke="steelblue"
    stroke-width="2"
    d={line($dataStoreSignal)}
    aria-hidden="true"
    stroke-dasharray="50000"
    stroke-dashoffset="50000"
    >
    <animate
      attributeName="stroke-dashoffset"
      dur="5s" 
      values="50000;0"  
      repeatCount="indefinite"
    />

  </path>

  <!-- Tooltip -->
  {#if tooltip}
    <g transform={`translate(${tooltip.x},${tooltip.y - 20})`} role="tooltip" aria-live="assertive">
      <rect width="100" height="40" fill="white" stroke="steelblue" stroke-width="1" />
      <text x="10" y="15" fill="steelblue">{tooltip.content}</text>
    </g>
  {/if}
</svg>
</main>