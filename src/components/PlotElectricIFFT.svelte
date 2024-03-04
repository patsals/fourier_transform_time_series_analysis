<script>
    import * as d3 from 'd3';
    import { format } from 'date-fns';

    export let dataStoreIFFT;

    let width = 900;
    let height = 300;
    let marginTop = 30;
    let marginRight = 80;
    let marginBottom = 30;
    let marginLeft = 80;
  
    let xScale;
    let yScale;
  
    let tooltip = null;
  
    let Date_time = [];
    let Voltage = [];

    dataStoreIFFT.subscribe(data => {
        // Update local time and signal arrays
        Date_time = data.map(d => d.Date_time);
        Voltage = data.map(d => d.Voltage);

    });


  // Function to initialize scales
  function initializeScales() {
    xScale = d3.scaleTime()
        .domain([d3.min(Date_time), d3.max(Date_time)]) // extent returns [min, max] of the dates
        .range([marginLeft, width - marginRight]);
            
    yScale = d3.scaleLinear(
        [d3.min(Voltage)-5, d3.max(Voltage)+5],
        [height - marginBottom, marginTop]
    );
  }


  $: {
        if (Voltage.length > 0) {
            initializeScales();
        }

    }
  
    const line = d3
    .line()
    .x((d) => xScale(d.Date_time))
    .y((d) => yScale(d.Voltage));

  function showTooltip(event) {
    const [x, y] = d3.pointer(event);
    const nearestDataPoint = findNearestDataPoint(x, y);
  
    // Add null check before accessing properties
    if (nearestDataPoint) {
      tooltip = {
        visible: true,
        x: x,
        y: y,
        content: `${nearestDataPoint.Voltage}, ${nearestDataPoint.Voltage}`,
      };
    }
  }
  
  function hideTooltip() {
    tooltip = null;
  }
  
  function findNearestDataPoint(x, y) {
    let minDistance = 5;
    let nearestDataPoint = null;
  
    dataStoreIFFT.subscribe(data => {
        data.forEach((d) => {
        const xPos = xScale(d.Date_time);
        const yPos = yScale(d.Voltage);
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
        {format(tick, 'M/d')}
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
        ↑ Voltage Values
      </text>
    </g>
  
    <!-- Line Path with Hover Effects -->
    <path
      fill="none"
      stroke="teal"
      stroke-width="2"
      d={line($dataStoreIFFT)}
      aria-hidden="true"
    />
  
    <!-- Tooltip -->
    {#if tooltip}
      <g transform={`translate(${tooltip.x},${tooltip.y - 20})`} role="tooltip" aria-live="assertive">
        <rect width="100" height="40" fill="white" stroke="steelblue" stroke-width="1" />
        <text x="10" y="15" fill="steelblue">{tooltip.content}</text>
      </g>
    {/if}
  </svg>
  </main>