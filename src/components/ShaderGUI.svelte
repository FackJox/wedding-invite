<script>
	import { onMount, onDestroy } from 'svelte';
	import { browser } from '$app/environment';

	export let params;
	export let imageSrc;

	// Disco ball CRT/ASCII effect params
	export let discoBallParams = null;

	let pane;
	let container;

	onMount(async () => {
		if (!browser) return;

		const { Pane } = await import('tweakpane');

		pane = new Pane({
			container,
			title: 'Effects'
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

		// Disco Ball CRT/ASCII Effect controls
		if (discoBallParams) {
			pane.addBlade({ view: 'separator' });

			const discoFolder = pane.addFolder({ title: 'Disco Ball', expanded: true });

			// ASCII settings - Tweakpane modifies object directly, DiscoBall reads every frame
			const asciiFolder = discoFolder.addFolder({ title: 'ASCII', expanded: true });
			asciiFolder.addBinding(discoBallParams, 'cellSize', { min: 2, max: 16, step: 1, label: 'Cell Size' });
			asciiFolder.addBinding(discoBallParams, 'asciiStyle', {
				label: 'Style',
				options: {
					'Standard': 0,
					'Dense': 1,
					'Minimal': 2,
					'Blocks': 3
				}
			});
			asciiFolder.addBinding(discoBallParams, 'colorMode', { label: 'Color Mode' });
			asciiFolder.addBinding(discoBallParams, 'invert', { label: 'Invert' });

			// CRT settings
			const crtFolder = discoFolder.addFolder({ title: 'CRT', expanded: false });
			crtFolder.addBinding(discoBallParams, 'scanlineIntensity', { min: 0, max: 1, step: 0.01, label: 'Scanlines' });
			crtFolder.addBinding(discoBallParams, 'scanlineCount', { min: 50, max: 400, step: 10, label: 'Line Count' });
			crtFolder.addBinding(discoBallParams, 'curvature', { min: 0, max: 0.2, step: 0.005, label: 'Curvature' });
			crtFolder.addBinding(discoBallParams, 'aberrationStrength', { min: 0, max: 0.02, step: 0.001, label: 'Chromatic Ab.' });

			// Post-processing
			const postFolder = discoFolder.addFolder({ title: 'Post FX', expanded: false });
			postFolder.addBinding(discoBallParams, 'vignetteIntensity', { min: 0, max: 1, step: 0.05, label: 'Vignette' });
			postFolder.addBinding(discoBallParams, 'vignetteRadius', { min: 0.5, max: 2, step: 0.1, label: 'Vig. Radius' });
			postFolder.addBinding(discoBallParams, 'bloomIntensity', { min: 0, max: 2, step: 0.05, label: 'Bloom' });
			postFolder.addBinding(discoBallParams, 'bloomMix', { min: 0, max: 0.5, step: 0.01, label: 'Bloom Mix' });
			postFolder.addBinding(discoBallParams, 'noiseIntensity', { min: 0, max: 0.1, step: 0.005, label: 'Noise' });

			// Adjustment
			const adjustFolder = discoFolder.addFolder({ title: 'Adjustments', expanded: false });
			adjustFolder.addBinding(discoBallParams, 'brightnessAdjust', { min: -0.5, max: 0.5, step: 0.01, label: 'Brightness' });
			adjustFolder.addBinding(discoBallParams, 'contrastAdjust', { min: 0.5, max: 2, step: 0.05, label: 'Contrast' });

			// Glitch (optional effects)
			const glitchFolder = discoFolder.addFolder({ title: 'Glitch', expanded: false });
			glitchFolder.addBinding(discoBallParams, 'glitchIntensity', { min: 0, max: 0.5, step: 0.01, label: 'Intensity' });
			glitchFolder.addBinding(discoBallParams, 'glitchFrequency', { min: 0, max: 10, step: 0.5, label: 'Frequency' });

			// Transform controls
			const transformFolder = discoFolder.addFolder({ title: 'Transform', expanded: true });
			transformFolder.addBinding(discoBallParams, 'positionX', { min: -2, max: 2, step: 0.1, label: 'Position X' });
			transformFolder.addBinding(discoBallParams, 'positionY', { min: -2, max: 2, step: 0.1, label: 'Position Y' });
			transformFolder.addBinding(discoBallParams, 'scale', { min: 0.5, max: 3, step: 0.1, label: 'Scale' });
			transformFolder.addBinding(discoBallParams, 'spinSpeed', { min: 0, max: 2, step: 0.05, label: 'Spin Speed' });
			transformFolder.addBinding(discoBallParams, 'tilt', { min: 0, max: 1, step: 0.05, label: 'Tilt' });
			transformFolder.addBinding(discoBallParams, 'ringTilt', { min: 0, max: 1.5, step: 0.05, label: 'Ring Tilt X' });
			transformFolder.addBinding(discoBallParams, 'ringTilt2', { min: -1, max: 1, step: 0.05, label: 'Ring Tilt Y' });
			transformFolder.addBinding(discoBallParams, 'ringSize', { min: 0.5, max: 2, step: 0.05, label: 'Ring Size' });
		}

		// Export button
		pane.addBlade({ view: 'separator' });
		pane.addButton({ title: 'Copy Params to Console' }).on('click', () => {
			console.log('Current shader params:');
			console.log(JSON.stringify(params, null, 2));
			if (discoBallParams) {
				console.log('Disco ball params:');
				console.log(JSON.stringify(discoBallParams, null, 2));
			}
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
