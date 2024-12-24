<script>
    import { onMount, createEventDispatcher } from 'svelte';
    
    const dispatch = createEventDispatcher();
    let container;
    let morphTo;
  
    onMount(async () => {
  console.log('Face: onMount called');
  const { setupSVGMorph } = await import('./SVGMorph.js');
  console.log('Face: SVGMorph imported');
  morphTo = await setupSVGMorph(container);
  console.log('Face: setupSVGMorph completed');
  dispatch('faceLoaded'); 
  console.log('Face: faceLoaded event dispatched');
  dispatch('morphReady', { morphTo });
});

    export function handleMorph(newState) {
        if (morphTo) {
            morphTo(newState);
        }
    }
</script>
  
  <!-- Container with fixed dimensions -->
  <div bind:this={container} >
    <!-- 
      You can place a fallback <svg> or simply leave the container empty; 
      the 'setupSVGMorph' function will inject an SVG for you.
    -->
  </div>
  
  <!-- <div>
    
  </div> -->
  
  <style>
    div {
      display: flex;
      flex-direction: column;
      align-items: center;
    }
  
    button {
      margin: 5px;
    }
  
   
  </style>
  