<script>
	import { onMount } from 'svelte';
	let posterContainer;
	let scaleFactor = 1;
	const REFERENCE_WIDTH = 420;
	const REFERENCE_HEIGHT = 594;

	function updateScale() {
		if (!posterContainer) return;
		const { width, height } = posterContainer.getBoundingClientRect();
		scaleFactor = width / REFERENCE_WIDTH;
	}

	let fontLoaded = false;

	onMount(() => {
		const font = new FontFace('Zuume', 'url(/zuume.woff2)');
		font.load().then(() => {
			document.fonts.add(font);
			fontLoaded = true;
		});

		updateScale();
		window.addEventListener('resize', updateScale);
		return () => window.removeEventListener('resize', updateScale);
	});
</script>

<div class="poster" bind:this={posterContainer}>
	<div class="poster-content" style="transform: scale({scaleFactor});">
        <div class="banner">
            <h1 class:font-loaded={fontLoaded}>SMILE LIKE IT'S A WEDDING</h1>
          </div>


	</div>
</div>

<style>
	.poster {
		aspect-ratio: 420 / 594; /* Let it shrink or grow based on screen size */
		max-width: 95vw;
		max-height: 95vh;
		background-color: #FEFA99;
		box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
		position: relative;
		overflow: hidden;
	}

	.poster-content {
		width: 420px;
		height: 594px;
		transform-origin: top left;
		border: 1px dashed #ccc;
	}


    .banner {
      width: 569px;
      height: 108px;
      background-color: #343233;
      display: flex;
      justify-content: center;
      align-items: center;
      position: absolute;
      top: 0;
      left: 50%;
      transform: translateX(-50%);
    }
  
    h1 {
      color: #FEFA99;
      font-size: 88px;
      letter-spacing: -4%;
      margin: 0;
      padding: 0;
      text-align: center;
      opacity: 0;
      transition: opacity 0.3s ease;
    }
  
    .font-loaded {
      font-family: 'Zuume', sans-serif;
      opacity: 1;
    }
</style>
