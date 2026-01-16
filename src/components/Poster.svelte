<script>
	import { onMount, createEventDispatcher } from 'svelte';

	let posterContainer;
	let scaleFactor = 1;
	const REFERENCE_WIDTH = 1133;
	const REFERENCE_HEIGHT = 1600;

	const dispatch = createEventDispatcher();

	let fontsLoaded = false;

	function updateScale() {
		if (!posterContainer) return;
		const { height } = posterContainer.getBoundingClientRect();
		scaleFactor = height / REFERENCE_HEIGHT;
	}

	function handleRsvpClick() {
		dispatch('openRsvp');
	}

	onMount(() => {
		// Wait for fonts to load
		document.fonts.ready.then(() => {
			fontsLoaded = true;
			dispatch('loaded');
		});

		updateScale();
		window.addEventListener('resize', updateScale);

		return () => {
			window.removeEventListener('resize', updateScale);
		};
	});
</script>

<div class="poster" bind:this={posterContainer}>
	<div class="poster-content" style="transform: scale({scaleFactor});">
		<!-- Names -->
		<div class="names-section">
			<h1 class="name" class:loaded={fontsLoaded}>COBIE COPE</h1>
			<span class="ampersand" class:loaded={fontsLoaded}>&</span>
			<h1 class="name" class:loaded={fontsLoaded}>SOPHIE COPE</h1>
		</div>

		<!-- Subtitle -->
		<p class="subtitle" class:loaded={fontsLoaded}>INVITE YOU TO</p>

		<!-- Main headline -->
		<div class="headline-section">
			<h2 class="headline" class:loaded={fontsLoaded}>THE SECOND</h2>
			<h2 class="headline" class:loaded={fontsLoaded}>SUMMER OF</h2>
			<h2 class="headline headline-love" class:loaded={fontsLoaded}>LOVE</h2>
		</div>

		<!-- Spacer for disco ball area -->
		<div class="disco-spacer"></div>

		<!-- Event details -->
		<div class="event-details">
			<p class="detail" class:loaded={fontsLoaded}>SATURDAY 19 SEPTEMBER</p>
			<p class="detail" class:loaded={fontsLoaded}>FAITH IN STRANGERS</p>
			<p class="detail" class:loaded={fontsLoaded}>MARGATE</p>
			<p class="detail" class:loaded={fontsLoaded}>17:30PM</p>
		</div>

		<!-- CTA Button -->
		<button class="cta-button" class:loaded={fontsLoaded} on:click={handleRsvpClick}>
			SPIN THE GLOBE TO RSVP
		</button>
	</div>
</div>

<style>
	/* Font faces for the downloaded fonts */
	@font-face {
		font-family: 'Horizon';
		src: url('/fonts/horizon.c7c834b7fe209bdc0a30119a6b1d26ae.b3ebf62f8ff8ae950dea860d56c49d1b.woff2') format('woff2');
		font-display: swap;
	}

	@font-face {
		font-family: 'CondensedBold';
		src: url('/fonts/scr-nsevbd-reg.3f1bb23215a22e898f7272aa2.4e21acea96a3d8e9d18d910b961567bb.woff2') format('woff2');
		font-display: swap;
	}

	@font-face {
		font-family: 'CondensedRegular';
		src: url('/fonts/scr-nsev-reg.039ecb79bbf7e9fd5981a06a01c.28e686f1a1e6846cf246683ff0cd3181.woff2') format('woff2');
		font-display: swap;
	}

	@font-face {
		font-family: 'SerifItalic';
		src: url('/fonts/cff149ee1e9d2be50ac77bcd86769d05.woff2') format('woff2');
		font-display: swap;
	}

	.poster {
		aspect-ratio: 1133 / 1600;
		max-width: 95vw;
		max-height: 95vh;
		position: relative;
		overflow: hidden;
		border-radius: 4px;
		box-shadow: 0 4px 30px rgba(0, 0, 0, 0.4);
		background: #101830;
	}

	.poster-content {
		width: 1133px;
		height: 1600px;
		transform-origin: top left;
		position: relative;
		z-index: 3;
		display: flex;
		flex-direction: column;
		align-items: center;
		padding: var(--content-padding-top, 45px) var(--content-padding-x, 40px);
		box-sizing: border-box;
	}

	/* Names section */
	.names-section {
		text-align: center;
		margin-top: var(--name-margin-top, 45px);
	}

	.name {
		font-family: 'Horizon', 'Rye', cursive;
		font-size: var(--name-font-size, 43px);
		color: #e58632;
		margin: 0;
		letter-spacing: var(--name-letter-spacing, 5px);
		line-height: var(--name-line-height, 0.7);
		text-transform: uppercase;
		opacity: 0;
		transition: opacity 0.3s ease;
	}

	.name.loaded {
		opacity: 1;
	}

	.ampersand {
		font-family: 'CondensedRegular', 'Anton', sans-serif;
		font-size: var(--amp-font-size, 39px);
		color: #ffffff;
		display: block;
		margin: var(--amp-margin, 0px) 0;
		opacity: 0;
		transition: opacity 0.3s ease;
	}

	.ampersand.loaded {
		opacity: 1;
	}

	/* Subtitle */
	.subtitle {
		font-family: 'CondensedRegular', 'Anton', sans-serif;
		font-size: var(--subtitle-font-size, 26px);
		color: #ffffff;
		letter-spacing: var(--subtitle-letter-spacing, 3px);
		margin: var(--subtitle-margin-top, 80px) 0 var(--subtitle-margin-bottom, 72px) 0;
		text-transform: uppercase;
		opacity: 0;
		transition: opacity 0.3s ease;
	}

	.subtitle.loaded {
		opacity: 1;
	}

	/* Headline */
	.headline-section {
		text-align: center;
		margin: 0;
	}

	.headline {
		font-family: 'Horizon', 'Rye', cursive;
		font-size: var(--headline-font-size, 87px);
		color: #e58632;
		margin: 0;
		line-height: var(--headline-line-height, 1.24);
		text-transform: uppercase;
		white-space: nowrap;
		opacity: 0;
		transition: opacity 0.3s ease;
	}

	.headline.loaded {
		opacity: 1;
	}

	.headline-love {
		font-size: var(--headline-love-font-size, 89px);
	}

	/* Disco spacer */
	.disco-spacer {
		flex: 1;
		min-height: var(--disco-spacer-min-height, 320px);
	}

	/* Event details */
	.event-details {
		text-align: center;
		margin: 0;
	}

	.detail {
		font-family: 'CondensedRegular', 'Anton', sans-serif;
		font-size: var(--detail-font-size, 26px);
		color: #ffffff;
		margin: var(--detail-margin, 4px) 0;
		letter-spacing: var(--detail-letter-spacing, 5px);
		line-height: var(--detail-line-height, 1.3);
		text-transform: uppercase;
		opacity: 0;
		transition: opacity 0.3s ease;
	}

	.detail.loaded {
		opacity: 1;
	}

	/* CTA Button */
	.cta-button {
		font-family: 'CondensedBold', 'Anton', sans-serif;
		font-size: var(--cta-font-size, 29px);
		color: #ffffff;
		background: transparent;
		border: none;
		cursor: pointer;
		letter-spacing: var(--cta-letter-spacing, 0px);
		text-transform: uppercase;
		margin-top: var(--cta-margin-top, 18px);
		margin-bottom: var(--cta-margin-bottom, 17px);
		padding: 16px 32px;
		position: relative;
		opacity: 0;
		transition: opacity 0.3s ease, transform 0.2s ease;
	}

	.cta-button.loaded {
		opacity: 1;
	}

	.cta-button:hover {
		transform: scale(1.05);
	}
</style>
