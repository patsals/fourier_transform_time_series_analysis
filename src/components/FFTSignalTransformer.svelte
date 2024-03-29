<script>
  import * as d3 from 'd3';
  import { writable } from 'svelte/store';
  import fft from 'fft-js';

  let counter = 1;

  const numPoints = 1024;
  const numSignals = 5;
  const interval = (40 * Math.PI) / numPoints;

  let time = [];
  let signal = [];
  let signalComponents = [];

  let magnitudes = [];
  let FFTresult = [];
  
  // const f = new FFT(numPoints);

  // const out = f.createComplexArray();

  // initialize our time array
  for (let i = 0; i <= numPoints; i++) {
    time.push(i * interval);
  }

  // initialize our signal components
  for (let i = 0; i < numPoints; i++) {
    let row = [];
    for (let j = 0; j < numSignals; j++) {
      let x = i * interval;
      // Set the first column to the desired value, and others to 0
      row.push(j === 0 ? Math.cos(x) : 0);
    }
    signalComponents.push(row);
  }
  updateSignal();

  function updateSignal() {
    for (let i = 0; i < signalComponents.length; i++) {
      signal[i] = signalComponents[i].reduce((acc, val) => acc + val, 0);
    }
    FFTresult = fft.fft(signal)
    magnitudes = FFTresult.map((value) => Math.sqrt(value[0] ** 2 + value[1] ** 2)/numPoints);
  }

  function increment() {
    if (counter < numSignals) {
      counter += 1;
      let sin_or_cos = Math.random();
      let amp = Math.random() * 3;
      let v_shift = Math.random() * 0;
      let h_shift = Math.random();
      let ang_freq = Math.random();
      for (let i = 0; i < signalComponents.length; i++) {
        let x = i * interval;
        if (sin_or_cos > 0.5) {
          signalComponents[i][counter - 1] = amp * Math.cos(ang_freq * (x - h_shift)) + v_shift;
        } else {
          signalComponents[i][counter - 1] = amp * Math.sin(ang_freq * (x - h_shift)) + v_shift;
        }
      }
      updateSignal();
    } else {
      console.log('Cannot have more than ' + numSignals);
    }
  }

  function decrement() {
    if (counter > 1) {
      for (let i = 0; i < signalComponents.length; i++) {
        signalComponents[i][counter - 1] = 0;
      }
      counter -= 1;
      updateSignal();
    } else {
      console.log('At least 1 wave must be present');
    }
  }

  // Create a writable store to hold the data
  let dataStoreSignal = writable([]);
  let dataStoreFFTMagnitudes = writable([]);

  $: {
    // Recreate the data array whenever signal or time changes
    dataStoreSignal.set(time.map((t, index) => ({ time: t, signal: signal[index] })));
    dataStoreFFTMagnitudes.set(time.slice(0, time.length / 2).map((t, index) => ({ time: t, signal: 2*magnitudes[index] })));
  }





  // Define width, height, etc.
  let width = 928;
  let height = 500;
  let marginTop = 20;
  let marginRight = 0;
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
      .domain([d3.min(signal) - 5, d3.max(signal) + 5])
      .range([height - marginBottom, marginTop]);
  }

  // Initialize scales when component is created or when data changes
  $: {
    initializeScales();
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
        content: `${nearestDataPoint.signal.toFixed(2)}`,
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
  <p>Number of Random Waves: {counter} <button on:click={increment}>+</button> <button on:click={decrement}>-</button></p>

  
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
    />

    <!-- Tooltip -->
    {#if tooltip}
      <g transform={`translate(${tooltip.x},${tooltip.y - 20})`} role="tooltip" aria-live="assertive">
        <rect width="100" height="40" fill="white" stroke="steelblue" stroke-width="1" />
        <text x="10" y="15" fill="steelblue">{tooltip.content}</text>
      </g>
    {/if}
  </svg>


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
      d={line($dataStoreFFTMagnitudes)}
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
