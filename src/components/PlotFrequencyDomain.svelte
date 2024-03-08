<script>
  import * as d3 from 'd3';
  export let dataStoreFFTMagnitudes;
  export let dataStoreWaves;
  export let dataStoreWaveComponents;

  let time = [];
  let signal = [];
  let waves = [];
  let waveComponents = [];

  dataStoreFFTMagnitudes.subscribe(data => {
      time = data.map(d => d.time);
      signal = data.map(d => d.signal);

  });

  dataStoreWaves.subscribe(data => {
        waves = data.map(d => d);
    });
  
    dataStoreWaveComponents.subscribe(data => {
      waveComponents = data.map(d => d);
  });
  

  // Define width, height, etc.
  let width = 900;
  let height = 500;
  let marginTop = 20;
  let marginRight = 10;
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
    .domain([d3.min(signal), d3.max(signal) + 1])
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
  let minDistance = 20;
  let nearestDataPoint = null;

  dataStoreFFTMagnitudes.subscribe(data => {
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
    <line stroke="currentColor" x1={marginLeft - 6} x2={width}/>

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
      ↑ Signal Amplitudes
    </text>
    <text fill="currentColor" text-anchor="start" x={width-250} y={height-40}>
      Signal Frequencies →
    </text>
  </g>

  <!-- Line Path with Hover Effects -->
  <path
    fill="none"
    stroke="steelblue"
    stroke-width="2"
    d={line($dataStoreFFTMagnitudes)}
    aria-hidden="true"
    >
 
  </path>

  <!-- Tooltip -->
  {#if tooltip}
    <g transform={`translate(${tooltip.x},${tooltip.y - 20})`} role="tooltip" aria-live="assertive">
      <rect width="100" height="40" fill="white" stroke="steelblue" stroke-width="1" />
      <text x="10" y="15" fill="steelblue">{tooltip.content}</text>
    </g>
  {/if}

  <!-- Display waves and arrows -->
  <text fill="currentColor" text-anchor="start" x={width-200} y={30} style="text-decoration: underline; font-weight: bold;">
    Added waves:
  </text>
  {#each waveComponents as wc, index }
    <text y={index * 20 + 50} x={width - 230} fill="currentColor">
      {wc[0] + ' wave with amp:' + wc[1] + '; freq:' + wc[2]}
    </text>

    {#if waveComponents[index]}
      <line
        x1={width - 230}
        y1={index * 20 + 50}
        x2={xScale(wc[2])}
        y2={yScale(wc[1])}
        stroke="red"
        marker-end="url(#arrowhead)"
        stroke-dasharray="5 5"
        >

  </line>
    {/if}
  {/each}


</svg>
</main>