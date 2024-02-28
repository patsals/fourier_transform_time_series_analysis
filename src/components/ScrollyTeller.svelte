<script>
  import Scroller from '@sveltejs/svelte-scroller';
  import Introduction from './Introduction.svelte';
  
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
          <div>
            <h1>Section {sectionIndex + 1}</h1>
            <p>This is some content within Section {sectionIndex + 1}.</p>
          </div>
        {:else if sectionIndex === 2}
        <div>
          <h1>Section {sectionIndex + 1}</h1>
          <p>This is some content within Section {sectionIndex + 1}.</p>
        </div>
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

  .background {
    width: 100%;
    height: 100vh;
    position: relative;
    outline: green solid 3px;
  }

  .foreground {
    width: 50%;
    margin: 0 auto;
    height: auto;
    position: relative;
    outline: red solid 3px;
  }

  .progress-bars {
    position: absolute;
    background: rgba(170, 51, 120, 0.2);
    visibility: visible;
  }

  section {
    height: 100vh;
    background-color: rgba(0, 0, 0, 0.2);
    outline: magenta solid 3px;
    text-align: center;
    max-width: 1000px;
    color: black;
    padding: 1em;
    margin: 0 0 2em 0;
    opacity: 0;
    transition: opacity 0.9s ease-in-out;
  }

  /* Use fade-in class to animate opacity on scroll */
  .fade-in {
    opacity: 1;
  }
</style>
