<script>
    import { onMount } from 'svelte';
    import { writable } from 'svelte/store';

    import FFT from 'fft.js';
    import PlotArray from './plotArray.svelte';
    
    // const f = new FFT(16);

    // const realInput = [1,2,3,4,1,2,3,4,1,2,3,4,1,2,3,4];

    // const out = f.createComplexArray();

    // f.realTransform(out, realInput);

    let counter = 1;

    const numPoints = 1024;
    const numColumns = 10;
    const interval = (6 * Math.PI) / numPoints;

    let time = [];
    let signal = [];
    let signalComponents = [];
  

    // initialize our time array
    for (let i = 0; i <= numPoints; i++) {
        time.push(i * interval);
    }

    // initialize our signal components
    for (let i = 0; i < numPoints; i++) {
        let row = [];
        let coeff = Math.random();
        for (let j = 0; j < numColumns; j++) {
            // Set the first column to the desired value, and others to 0
            row.push(j === 0 ? Math.cos(i * interval) : 0);
           // row.push(j === 0 ? 1: 0)
        }
        signalComponents.push(row);
    }
    updateSignal();
    

    function updateSignal(){
        for (let i = 0; i < signalComponents.length; i++){
            signal[i] = signalComponents[i].reduce((acc, val) => acc + val, 0);
        }
    }

    function increment() {
    if (counter < 10) {
        counter += 1;
        let temp = Math.random();
        let coeff = Math.random();
        for (let i = 0; i < signalComponents.length; i++){
            if (temp > 0.5){
                signalComponents[i][counter - 1] = coeff * Math.cos(i * interval);
            }else{
                signalComponents[i][counter - 1] = coeff * Math.sin(i * interval);
            }
                
        }
        updateSignal();
      } else {
        console.log('Cannot have more than 10');
      }
    //   console.log(signal[10]);
    };
  

    function decrement() {
        if (counter > 1){
            for (let i = 0; i < signalComponents.length; i++){
                signalComponents[i][counter - 1] = 0;
            }
            counter -= 1;
            updateSignal();
        } else {
            console.log('At least 1 wave must be present');
        }
        // console.log(signal[10]);
    };
    




    let a = 1;
  </script>
  
  <main>
    <h1>Svelte User Input</h1>
  
    <!-- <label for="userInput">Enter Something:</label>
    <input type="text" id="userInput" bind:value={userInput} on:input={handleInputChange} />
  
    <button on:click={saveUserInput}>Save Input</button> -->
  
    <h1>Add waves</h1>
    
    <p>Number of Random Cosine Waves: {counter}</p>

    <button on:click={increment}>+</button>
    <button on:click={decrement}>-</button>

    
    <PlotArray {time} {signal}/>

    <p>{signal}</p>
  </main>
  