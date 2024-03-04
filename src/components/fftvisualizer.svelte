<script>
    import * as d3 from 'd3';
    import {  writable } from 'svelte/store';
    import fft from 'fft-js';
    import PlotSignal from './PlotSignal.svelte';
    import PlotFrequencyDomain from './PlotFrequencyDomain.svelte';

    let counter = 1;
  
    //const numPoints = 1024;
    const numSignals = 5;
    //const interval = (40 * Math.PI) / numPoints;
  

    let fs = 20; //sampling frequency
    let tstep = 1 / fs; //sampling time interval
    //let tstep = (40 * Math.PI) / numPoints; //sampling time interval

    let f0 = 1; //signal frequency

    let numPoints = 256; //Math.floor(fs / f0) * ; //number of samples

    //let numPoints =  N ; // numcyles
    let interval = tstep;
    let fstep = f0 / numPoints; //frequency interval
    let frequency = [];

    let time = [];
    let signal = [];
    let signalComponents = [];
  
    let magnitudes = [];
    let FFTresult = [];
    
    let waves = ['cos(2 * pi * 1 * x)'];
  
    // initialize our time array
    for (let i = 0; i <= numPoints; i++) {
      time.push(i * interval);
    }

    // initialize our frequency array
    for (let i = 0; i <= numPoints; i++) {
      frequency.push(i * fstep * 2);
    }
  
    // initialize our signal components
    for (let i = 0; i < numPoints; i++) {
      let row = [];
      for (let j = 0; j < numSignals; j++) {
        let x = i * interval;
        // Set the first column to cos(2 * pi * frequency * timestep), and others to 0
        row.push(j === 0 ? Math.cos( 2 * Math.PI * f0 * x) : 0);
        //row.push(j === 0 ? Math.cos(x) : 0);

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
        let ang_freq = Math.random() * 3;
        let wave_added = '';
        for (let i = 0; i < signalComponents.length; i++) {
          let x = i * interval;
          if (sin_or_cos > 0.5) {
            //signalComponents[i][counter - 1] = amp * Math.cos(ang_freq * (x - h_shift)) + v_shift;
            signalComponents[i][counter - 1] = amp * Math.cos( 2 * Math.PI * ang_freq * (x - h_shift)) + v_shift;

            wave_added = amp.toFixed(2)+'cos('+ang_freq.toFixed(2)+'(x-'+h_shift.toFixed(2)+'))+'+v_shift.toFixed(2);
          } else {
            //signalComponents[i][counter - 1] = amp * Math.sin(ang_freq * (x - h_shift)) + v_shift;
            signalComponents[i][counter - 1] = amp * Math.sin( 2 * Math.PI * ang_freq * (x - h_shift)) + v_shift;

            wave_added = amp.toFixed(2)+'cos('+ang_freq.toFixed(2)+'(x-'+h_shift.toFixed(2)+'))+'+v_shift.toFixed(2);
          }
        }
        waves.push(wave_added);
        console.log(waves);
        //console.log(wave_added);
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
        waves.pop();
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
    dataStoreFFTMagnitudes.set(frequency.slice(0, frequency.length / 2).map((t, index) => ({ time: t*10, signal: 2*magnitudes[index] })));
  }

  </script>
  
  <main>
    <h2>Signal: </h2>
    <p>Number of Random Cosine Waves: {counter}<button on:click={increment}>+</button><button on:click={decrement}>-</button></p>
    <PlotSignal {dataStoreSignal}/>
    <h2>Frequency Domain</h2>
    <PlotFrequencyDomain {dataStoreFFTMagnitudes}/>
    <p>{waves}</p>
  </main>
  