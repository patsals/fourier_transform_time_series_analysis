<!-- https://svelte.dev/repl/3ad4a548a4c442b69f20c25021ac8fbf?version=4.1.0 -->
<script>
  import * as d3 from "d3";

  export let data;
  export let width = 928;
  export let height = 500;
  export let marginTop = 20;
  export let marginRight = 30;
  export let marginBottom = 30;
  export let marginLeft = 40;

  const xScale = d3.scaleUtc(
    d3.extent(data, (d) => new Date(d.date)),
    [marginLeft, width - marginRight]
  );

  const yScale = d3.scaleLinear(
    [0, d3.max(data, (d) => d.close)],
    [height - marginBottom, marginTop]
  );

  const line = d3
    .line()
    .x((d) => xScale(new Date(d.date)))
    .y((d) => yScale(d.close));

  let tooltip = null;

  function showTooltip(event) {
    const [x, y] = d3.pointer(event);
    const nearestDataPoint = findNearestDataPoint(x, y);
    tooltip = {
      visible: true,
      x: x,
      y: y,
      content: `${nearestDataPoint.close.toFixed(2)}`,
    };
  }

  function hideTooltip() {
    tooltip = null;
  }

  function findNearestDataPoint(x, y) {
    let minDistance = 5;
    let nearestDataPoint = null;

    data.forEach((d) => {
      const xPos = xScale(new Date(d.date));
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
  viewBox="0 0 {width} {height}"
  style:max-width="100%"
  style:height="auto"
  role="img"
  on:mousemove={showTooltip}
  on:mouseleave={hideTooltip}
>
  <!-- X-Axis -->
  <g transform="translate(0,{height - marginBottom})">
    <line stroke="currentColor" x1={marginLeft - 6} x2={width} />

    {#each xScale.ticks() as tick}
      <!-- X-Axis Ticks -->
      <line
        stroke="currentColor"
        x1={xScale(tick)}
        x2={xScale(tick)}
        y1={0}
        y2={6}
      />

      <!-- X-Axis Tick Labels -->
      <text fill="currentColor" text-anchor="middle" x={xScale(tick)} y={22}>
        {tick.getFullYear()}
      </text>
    {/each}
  </g>

  <!-- Y-Axis and Grid Lines -->
  <g transform="translate({marginLeft},0)">
    {#each yScale.ticks() as tick}
      {#if tick !== 0}
        <!-- 
          Grid Lines. 
          Note: First line is skipped since the x-axis is already present at 0. 
        -->
        <line
          stroke="currentColor"
          stroke-opacity="0.1"
          x1={0}
          x2={width - marginLeft}
          y1={yScale(tick)}
          y2={yScale(tick)}
        />

        <!-- 
          Y-Axis Ticks. 
          Note: First tick is skipped since the x-axis already acts as a tick. 
        -->
        <line
          stroke="currentColor"
          x1={0}
          x2={-6}
          y1={yScale(tick)}
          y2={yScale(tick)}
        />
      {/if}

      <!-- Y-Axis Tick Labels -->
      <text
        fill="currentColor"
        text-anchor="end"
        dominant-baseline="middle"
        x={-9}
        y={yScale(tick)}
      >
        {tick}
      </text>
    {/each}

    <!-- Y-Axis Label -->
    <text fill="currentColor" text-anchor="start" x={-marginLeft} y={15}>
      ↑ Daily close ($)
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
