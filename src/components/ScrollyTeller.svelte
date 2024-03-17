<script>
  import Scroller from '@sveltejs/svelte-scroller';
  import Introduction from './Introduction.svelte';
  import Fftvisualizer from './fftvisualizer.svelte';
  import FftDataAnalysis from './FFTDataAnalysis.svelte';
  import Conclusion from './conclusion.svelte';


  let count, index, fadeIn;

  $: {
    fadeIn = Array.from({ length: count }, (_, i) => index === i);
  }
</script>

<Scroller
  top={0.0}
  bottom={1}
  threshold={0.5}
  bind:count
  bind:index
>
  <!-- <div class="background" slot="background">
    <div class="progress-bars">
      <p>current section: <strong>{index + 1}/{count}</strong></p>
      <progress value={count ? (index + 1) / count : 0} />

      <p>total progress</p>
      <progress value={index / (count - 1) || 0} />
    </div>
  </div> -->

  <div class="foreground" slot="foreground">
    {#each Array(7) as _, sectionIndex}
      <section class:fade-in={fadeIn[sectionIndex]}>
        {#if sectionIndex === 0}
          <Introduction />
          <h1> </h1>
        {:else if sectionIndex === 1}
          <h3>At its core, the Fourier Transform breaks down a signal or a function into its constituent frequencies, revealing underlying spectral information.</h3>
          <!-- <img src={"/src/FFT.png"} alt="something" style="width: 50%; height: 50%;"> -->
          <img src={"https://miro.medium.com/v2/resize:fit:4800/format:webp/0*3SPZL7OBkn4RhLkG.gif"} alt="something" style="width: 50%; height: 50%;">
          <h3>When represented mathematically, it comes primarily in two forms,</h3>
          <!-- <h4>The Continuous Fourier Transform:</h4> -->
          <img src={"https://i.stack.imgur.com/iHRkd.png"} alt="something" 
              style="width: 30%; height: 30%;">
            
        {:else if sectionIndex === 2}
          <h1> </h1>
          <h3> but its turns out, both computations can quickly become computationally inefficient...</h3>
          <h3>However, thanks to advancements in science, a much more efficient way of computing these equations has been discovered!</h3>
          <h1> </h1>
        {:else if sectionIndex === 3}
          <Fftvisualizer />
        {:else if sectionIndex === 4}
          <h1> </h1>
          <h3>But what can we do with it?</h3>
          <h1> </h1>
        {:else if sectionIndex === 5}
          <FftDataAnalysis />
          <h1> </h1>
        {:else if sectionIndex === 6}
        <h3>In conclusion,</h3>
          <Conclusion />
          <img src="https://www.explainxkcd.com/wiki/images/f/f0/fourier.jpg" alt="FFT Cat">
          <h1> </h1>   
        {/if}
      </section>
    {/each}
  </div>
</Scroller>





<style>
  @import '../styles.css'; 

</style>