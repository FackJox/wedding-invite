<script>
	import { onMount, onDestroy } from 'svelte';
	import { browser } from '$app/environment';

	export let params;
	export let imageSrc;

	let pane;
	let container;

	onMount(async () => {
		if (!browser) return;

		const { Pane } = await import('tweakpane');

		pane = new Pane({
			container,
			title: 'Film Effects'
		});

		// Color adjustments
		const colorFolder = pane.addFolder({ title: 'Color' });
		colorFolder.addBinding(params, 'saturation', { min: 0, max: 3, step: 0.05, label: 'Saturation' });
		colorFolder.addBinding(params, 'contrast', { min: 0.5, max: 2, step: 0.05, label: 'Contrast' });

		// Film Grain - retrowave analog aesthetic
		const grainFolder = pane.addFolder({ title: 'Film Grain' });
		grainFolder.addBinding(params, 'grainIntensity', { min: 0, max: 0.4, step: 0.01, label: 'Intensity' });
		grainFolder.addBinding(params, 'grainSize', { min: 0.5, max: 4, step: 0.1, label: 'Size' });
		grainFolder.addBinding(params, 'grainSpeed', { min: 1, max: 24, step: 1, label: 'Speed' });
		grainFolder.addBinding(params, 'grainChroma', { min: 0, max: 1, step: 0.05, label: 'Chroma' });

		// Black & White Speckles - film dust/dirt
		const speckleFolder = pane.addFolder({ title: 'Speckles' });
		speckleFolder.addBinding(params, 'speckleIntensity', { min: 0, max: 1, step: 0.01, label: 'Intensity' });
		speckleFolder.addBinding(params, 'speckleSize', { min: 0.1, max: 8, step: 0.1, label: 'Size' });
		speckleFolder.addBinding(params, 'speckleDensity', { min: 0.5, max: 0.999, step: 0.001, label: 'Density' });
		speckleFolder.addBinding(params, 'speckleSpeed', { min: 1, max: 30, step: 1, label: 'Speed' });

		// Blur
		pane.addBlade({ view: 'separator' });
		pane.addBinding(params, 'blur', { min: 0, max: 10, step: 0.5, label: 'Blur' });

		// Export button
		pane.addBlade({ view: 'separator' });
		pane.addButton({ title: 'Copy Params to Console' }).on('click', () => {
			console.log('Current shader params:');
			console.log(JSON.stringify(params, null, 2));
		});
	});

	onDestroy(() => {
		if (pane) {
			pane.dispose();
		}
	});
</script>

<div bind:this={container} class="shader-gui"></div>

<style>
	.shader-gui {
		position: fixed;
		top: 10px;
		right: 10px;
		z-index: 9999;
		max-height: 90vh;
		overflow-y: auto;
	}

	.shader-gui :global(.tp-dfwv) {
		width: 240px !important;
	}
</style>
