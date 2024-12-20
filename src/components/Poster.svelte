<script>
	import { onMount } from 'svelte';
	let posterContainer;
	let scaleFactor = 1

	// Internal reference dimensions of your A4 design in px. // Adjust as desired, but these numbers match 210mm x 297mm // when considering a simplistic 1px = 1mm approach. 
    const REFERENCE_WIDTH = 420; const REFERENCE_HEIGHT = 594;

	function updateScale() {
		if (!posterContainer) return;
		// Measure how big the poster container currently is
		const { width, height } = posterContainer.getBoundingClientRect();

		// Figure out how much we need to scale our "content" to fill that container
		// based on the reference width we designed for originally.
		// We focus on width since aspect-ratio will keep the height correct.
		scaleFactor = width / REFERENCE_WIDTH;
	}
	// Update scale on initial mount and whenever window resizes
	onMount(() => {
		updateScale();
		window.addEventListener('resize', updateScale);
		return () => window.removeEventListener('resize', updateScale);
	});
</script>

<!-- Outer container that keeps the A4 ratio and is centered by the parent. We bind it so we can measure it and calculate scale. -->
<div class="poster" bind:this={posterContainer}>
	<!-- The .poster-content has a fixed reference width/height (simulating a 'true size' A4). We then apply a transform: scale(...) to ensure everything shrinks/grows uniformly. -->
	<div class="poster-content" style="transform: scale({scaleFactor});">
		<h1>Poster Title</h1>
		<p>This is a sample of text on the poster. The layout here will scale proportionally.</p>
		<!-- Add more poster elements here, laid out at the "true" reference size -->
	</div>
</div>

<style>
	/* The .poster uses aspect-ratio to maintain 210:297, effectively locking width/height to the A4 ratio. */
	.poster {
		aspect-ratio: 420 / 594; /* Let it shrink or grow based on screen size */
		max-width: 95vw;
		max-height: 95vh;
		background-color: #fff;
		box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
		position: relative;
		overflow: hidden;
	}

	/*Reference size for A4. Everything inside is placed as if it's 210 x 297 "absolute" space, then scaled by transform. 
   */
	.poster-content {
		width: 420px;
		height: 594px;
		transform-origin: top left;
		/* To show layout boundaries in a wireframe, uncomment below:
   border: 1px dashed #ccc;
*/
	}

	/* Basic formatting for your poster text */
	.poster-content > h1 {
		font-size: 24px;
		margin-bottom: 0.5rem;
	}
	.poster-content > p {
		font-size: 14px;
	}
</style>
