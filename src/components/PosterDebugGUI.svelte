<script>
	import { onMount, onDestroy } from 'svelte';
	import { Pane } from 'tweakpane';

	export let config = {
		// Names
		nameFontSize: 43,
		nameLetterSpacing: 5,
		nameLineHeight: 0.7,
		nameMarginTop: 45,

		// Ampersand
		ampFontSize: 39,
		ampMargin: 0,

		// Subtitle
		subtitleFontSize: 26,
		subtitleLetterSpacing: 3,
		subtitleMarginTop: 80,
		subtitleMarginBottom: 72,

		// Headline
		headlineFontSize: 87,
		headlineLineHeight: 1.24,
		headlineLoveFontSize: 89,

		// Disco spacer
		discoSpacerMinHeight: 320,

		// Event details
		detailFontSize: 26,
		detailLetterSpacing: 5,
		detailLineHeight: 1.3,
		detailMargin: 4,

		// CTA
		ctaFontSize: 29,
		ctaLetterSpacing: 0,
		ctaMarginTop: 18,
		ctaMarginBottom: 17,

		// Padding
		contentPaddingTop: 45,
		contentPaddingX: 40,

		// Reference image
		showReferenceImage: false,
		referenceOpacity: 15,
	};

	export let visible = false;

	let pane;
	let container;

	function copyConfig() {
		const configStr = JSON.stringify(config, null, 2);
		navigator.clipboard.writeText(configStr).then(() => {
			alert('Config copied to clipboard!');
		});
	}

	function applyConfig() {
		const root = document.documentElement;
		root.style.setProperty('--name-font-size', `${config.nameFontSize}px`);
		root.style.setProperty('--name-letter-spacing', `${config.nameLetterSpacing}px`);
		root.style.setProperty('--name-line-height', config.nameLineHeight);
		root.style.setProperty('--name-margin-top', `${config.nameMarginTop}px`);
		root.style.setProperty('--amp-font-size', `${config.ampFontSize}px`);
		root.style.setProperty('--amp-margin', `${config.ampMargin}px`);
		root.style.setProperty('--subtitle-font-size', `${config.subtitleFontSize}px`);
		root.style.setProperty('--subtitle-letter-spacing', `${config.subtitleLetterSpacing}px`);
		root.style.setProperty('--subtitle-margin-top', `${config.subtitleMarginTop}px`);
		root.style.setProperty('--subtitle-margin-bottom', `${config.subtitleMarginBottom}px`);
		root.style.setProperty('--headline-font-size', `${config.headlineFontSize}px`);
		root.style.setProperty('--headline-line-height', config.headlineLineHeight);
		root.style.setProperty('--headline-love-font-size', `${config.headlineLoveFontSize}px`);
		root.style.setProperty('--disco-spacer-min-height', `${config.discoSpacerMinHeight}px`);
		root.style.setProperty('--detail-font-size', `${config.detailFontSize}px`);
		root.style.setProperty('--detail-letter-spacing', `${config.detailLetterSpacing}px`);
		root.style.setProperty('--detail-line-height', config.detailLineHeight);
		root.style.setProperty('--detail-margin', `${config.detailMargin}px`);
		root.style.setProperty('--cta-font-size', `${config.ctaFontSize}px`);
		root.style.setProperty('--cta-letter-spacing', `${config.ctaLetterSpacing}px`);
		root.style.setProperty('--cta-margin-top', `${config.ctaMarginTop}px`);
		root.style.setProperty('--cta-margin-bottom', `${config.ctaMarginBottom}px`);
		root.style.setProperty('--content-padding-top', `${config.contentPaddingTop}px`);
		root.style.setProperty('--content-padding-x', `${config.contentPaddingX}px`);
		// Reference overlay - set both opacity and visibility
		if (config.showReferenceImage) {
			root.style.setProperty('--reference-opacity', (config.referenceOpacity / 100).toString());
			root.style.setProperty('--reference-visibility', 'visible');
		} else {
			root.style.setProperty('--reference-opacity', '0');
			root.style.setProperty('--reference-visibility', 'hidden');
		}
	}

	onMount(() => {
		pane = new Pane({ container, title: 'Poster Config' });

		// Reference image folder
		const refFolder = pane.addFolder({ title: 'Reference Image' });
		refFolder.addBinding(config, 'showReferenceImage', { label: 'Show Reference' });
		refFolder.addBinding(config, 'referenceOpacity', { min: 10, max: 100, step: 5, label: 'Opacity %' });

		// Names folder
		const namesFolder = pane.addFolder({ title: 'Names (COBIE/SOPHIE)' });
		namesFolder.addBinding(config, 'nameFontSize', { min: 40, max: 150, step: 1, label: 'Font Size' });
		namesFolder.addBinding(config, 'nameLetterSpacing', { min: 0, max: 20, step: 1, label: 'Letter Spacing' });
		namesFolder.addBinding(config, 'nameLineHeight', { min: 0.7, max: 2, step: 0.05, label: 'Line Height' });
		namesFolder.addBinding(config, 'nameMarginTop', { min: 0, max: 100, step: 1, label: 'Margin Top' });

		// Ampersand folder
		const ampFolder = pane.addFolder({ title: 'Ampersand (&)' });
		ampFolder.addBinding(config, 'ampFontSize', { min: 20, max: 100, step: 1, label: 'Font Size' });
		ampFolder.addBinding(config, 'ampMargin', { min: 0, max: 30, step: 1, label: 'Margin Y' });

		// Subtitle folder
		const subtitleFolder = pane.addFolder({ title: 'Subtitle (INVITE YOU TO)' });
		subtitleFolder.addBinding(config, 'subtitleFontSize', { min: 10, max: 60, step: 1, label: 'Font Size' });
		subtitleFolder.addBinding(config, 'subtitleLetterSpacing', { min: 0, max: 30, step: 1, label: 'Letter Spacing' });
		subtitleFolder.addBinding(config, 'subtitleMarginTop', { min: 0, max: 80, step: 1, label: 'Margin Top' });
		subtitleFolder.addBinding(config, 'subtitleMarginBottom', { min: 0, max: 80, step: 1, label: 'Margin Bottom' });

		// Headline folder
		const headlineFolder = pane.addFolder({ title: 'Headline (THE SECOND...)' });
		headlineFolder.addBinding(config, 'headlineFontSize', { min: 60, max: 200, step: 1, label: 'Font Size' });
		headlineFolder.addBinding(config, 'headlineLineHeight', { min: 0.7, max: 1.5, step: 0.01, label: 'Line Height' });
		headlineFolder.addBinding(config, 'headlineLoveFontSize', { min: 60, max: 220, step: 1, label: 'LOVE Font Size' });

		// Disco spacer folder
		const discoFolder = pane.addFolder({ title: 'Disco Spacer' });
		discoFolder.addBinding(config, 'discoSpacerMinHeight', { min: 100, max: 600, step: 10, label: 'Min Height' });

		// Event details folder
		const detailsFolder = pane.addFolder({ title: 'Event Details' });
		detailsFolder.addBinding(config, 'detailFontSize', { min: 20, max: 80, step: 1, label: 'Font Size' });
		detailsFolder.addBinding(config, 'detailLetterSpacing', { min: 0, max: 15, step: 1, label: 'Letter Spacing' });
		detailsFolder.addBinding(config, 'detailLineHeight', { min: 0.7, max: 2, step: 0.05, label: 'Line Height' });
		detailsFolder.addBinding(config, 'detailMargin', { min: 0, max: 20, step: 1, label: 'Margin Y' });

		// CTA folder
		const ctaFolder = pane.addFolder({ title: 'CTA Button' });
		ctaFolder.addBinding(config, 'ctaFontSize', { min: 16, max: 60, step: 1, label: 'Font Size' });
		ctaFolder.addBinding(config, 'ctaLetterSpacing', { min: 0, max: 15, step: 1, label: 'Letter Spacing' });
		ctaFolder.addBinding(config, 'ctaMarginTop', { min: -100, max: 80, step: 1, label: 'Margin Top' });
		ctaFolder.addBinding(config, 'ctaMarginBottom', { min: 0, max: 150, step: 1, label: 'Margin Bottom' });

		// Padding folder
		const paddingFolder = pane.addFolder({ title: 'Content Padding' });
		paddingFolder.addBinding(config, 'contentPaddingTop', { min: 0, max: 150, step: 5, label: 'Padding Top' });
		paddingFolder.addBinding(config, 'contentPaddingX', { min: 0, max: 100, step: 5, label: 'Padding X' });

		// Copy button
		pane.addButton({ title: 'Copy Config to Clipboard' }).on('click', copyConfig);

		// Listen for changes
		pane.on('change', () => {
			applyConfig();
		});

		// Apply initial config
		applyConfig();
	});

	onDestroy(() => {
		if (pane) pane.dispose();
	});
</script>

{#if visible}
	<div class="gui-container" bind:this={container}></div>
{/if}

<style>
	.gui-container {
		position: fixed;
		top: 10px;
		right: 10px;
		z-index: 9999;
		max-height: 90vh;
		overflow-y: auto;
	}

	:global(.tp-dfwv) {
		width: 300px !important;
	}
</style>
