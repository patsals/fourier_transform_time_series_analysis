<script>
    import { onMount } from 'svelte';
    import { writable } from 'svelte/store';

    import FFT from 'fft.js';
    
    const f = new FFT(16);

    const realInput = [1,2,3,4,1,2,3,4,1,2,3,4,1,2,3,4];

    const out = f.createComplexArray();

    f.realTransform(out, realInput);

    

    let userInput = ''; // Variable to store user input

    function handleInputChange(event) {
        userInput = event.target.value;
    }

    function saveUserInput() {
        // You can perform further actions with the user input here
        console.log('User Input:', userInput);
    }

    let counter = 1;

    function increment() {
    if (counter < 3) {
        counter += 1;
        for (let i = 0; i < twoDArray[0].length; i++){
            twoDArray[i][counter - 1] = 1;
        }
      } else {
        console.log('Cannot have more than 10');
      }
      console.log(twoDArray);
    };
  

    function decrement() {
        if (counter > 1){
            for (let i = 0; i < twoDArray[0].length; i++){
                twoDArray[i][counter - 1] = 0;
            }
            counter -= 1;
        } else {
            console.log('At least 1 wave must be present');
        }
        console.log(twoDArray);
    };
    

    let twoDArray = [
        [1, 0, 0],
        [1, 0, 0],
        [1, 0, 0]
    ];


  </script>
  
  <main>
    <h1>Svelte User Input</h1>
  
    <label for="userInput">Enter Something:</label>
    <input type="text" id="userInput" bind:value={userInput} on:input={handleInputChange} />
  
    <button on:click={saveUserInput}>Save Input</button>
  
    <h1>Svelte Counter</h1>
    
    <p>Number of Random Cosine Waves: {counter}</p>

    <button on:click={increment}>+</button>
    <button on:click={decrement}>-</button>
  </main>
  