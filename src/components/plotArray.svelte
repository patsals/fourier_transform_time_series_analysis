<script>
    import * as d3 from 'd3';
  
    export let time = []; // Assuming these are initialized with some data
    export let signal = []; // Assuming these are initialized with some data
  
    let data = time.map((t, index) => ({ date: t, close: signal[index] })); // Create data array of objects
  
    let width = 928;
    let height = 500;
    let marginTop = 20;
    let marginRight = 30;
    let marginBottom = 30;
    let marginLeft = 40;
  
    let xScale;
    let yScale;
  
    let tooltip = null;
  
    // Function to initialize scales
    function initializeScales() {
      xScale = d3.scaleLinear()
        .domain([d3.min(time), d3.max(time)])
        .range([marginLeft, width - marginRight]);
  
      yScale = d3.scaleLinear()
        .domain([d3.min(signal), d3.max(signal)])
        .range([height - marginBottom, marginTop]);
    }
  
    // Initialize scales when component is created
    $: initializeScales();
  
    const line = d3
      .line()
      .x((d) => xScale(d.date))
      .y((d) => yScale(d.close));
  
    function showTooltip(event) {
      const [x, y] = d3.pointer(event);
      const nearestDataPoint = findNearestDataPoint(x, y);
  
      // Add null check before accessing properties
      if (nearestDataPoint) {
        tooltip = {
          visible: true,
          x: x,
          y: y,
          content: `${nearestDataPoint.close.toFixed(2)}`,
        };
      }
    }
  
    function hideTooltip() {
      tooltip = null;
    }
  
    function findNearestDataPoint(x, y) {
      let minDistance = 5;
      let nearestDataPoint = null;
  
      data.forEach((d) => {
        const xPos = xScale(d.date);
        const yPos = yScale(d.close);
        const distance = Math.sqrt((x - xPos) ** 2 + (y - yPos) ** 2);
  
        if (distance < minDistance) {
          minDistance = distance;
          nearestDataPoint = d;
        }
      });
  
      return nearestDataPoint;
    }
  </script>
  
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
      d={line(data)}
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
  