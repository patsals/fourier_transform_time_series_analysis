<script>
  import Scroller from '@sveltejs/svelte-scroller';
  import Introduction from './Introduction.svelte';
  import Fftvisualizer from './fftvisualizer.svelte';
  import FftDataAnalysis from './FFTDataAnalysis.svelte';


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
  <div class="background" slot="background">
    <div class="progress-bars">
      <p>current section: <strong>{index + 1}/{count}</strong></p>
      <progress value={count ? (index + 1) / count : 0} />

      <p>total progress</p>
      <progress value={index / (count - 1) || 0} />
    </div>
  </div>

  <div class="foreground" slot="foreground">
    {#each Array(4) as _, sectionIndex}
      <section class:fade-in={fadeIn[sectionIndex]}>
        {#if sectionIndex === 0}
          <Introduction />
        {:else if sectionIndex === 1}
          <Fftvisualizer />
          
        {:else if sectionIndex === 2}
          <FftDataAnalysis />
        {:else if sectionIndex === 3}
        <div>
          <h1>Section {sectionIndex + 1}</h1>
          <p>This is some content within Section {sectionIndex + 1}.</p>
        </div>
        {/if}
      </section>
    {/each}
  </div>
</Scroller>





<style>
  @import '../styles.css'; 

</style>
