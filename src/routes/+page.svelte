<script>
	import Poster from '../components/Poster.svelte';
	import Rsvp from '../components/Rsvp.svelte';
	import { fade } from 'svelte/transition';

	let showRsvp = false;
	let showThankYou = false;

	function toggleRsvp() {
		showRsvp = !showRsvp;
	}

	function handleFormSuccess() {
		showRsvp = false;
		showThankYou = true;
		setTimeout(() => {
			showThankYou = false;
		}, 4000);
	}
</script>

<main>
	<Poster on:openRsvp={toggleRsvp} />
	
	{#if showRsvp}
		<div class="modal" transition:fade>
			<Rsvp on:success={handleFormSuccess} />
		</div>
	{/if}

	{#if showThankYou}
		<div class="thank-you" transition:fade>
			<p>Thank you for your RSVP!</p>
		</div>
	{/if}
</main>

<style>
	/* Fill the entire viewport and center children */
	main {
		width: 100vw;
		height: 100vh;
		margin: 0;
		padding: 0;
		display: flex;
		justify-content: center;
		align-items: center;
		background-color: #f0f0f0; /* just a gray background for the wireframe */
	}

    .modal {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background-color: rgba(0, 0, 0, 0.5);
		display: flex;
		justify-content: center;
		align-items: center;
		z-index: 1000;
	}

	.thank-you {
		position: fixed;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		background-color: #4CAF50;
		color: white;
		padding: 20px;
		border-radius: 5px;
		z-index: 1001;
	}
</style>
