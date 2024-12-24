<script>
	import Face from '../components/Face.svelte';
    import Poster from '../components/Poster.svelte';
    import Rsvp from '../components/Rsvp.svelte';
    import { fade } from 'svelte/transition';

    let showRsvp = false;
    let showThankYou = false;

    function toggleRsvp() {
        showRsvp = !showRsvp;
    }

    function handleFormSuccess() {
        console.log('handleFormSuccess called');

        showRsvp = false;
        showThankYou = true;
        setTimeout(() => {
            console.log('Timeout callback executed');

            showThankYou = false;
        }, 10000);
    }

    function handleModalClick(event) {
        // Check if the click target is the modal background (not its children)
        if (event.target.classList.contains('modal')) {
            showRsvp = false;
        }
    }
</script>

<main>
    <Poster on:openRsvp={toggleRsvp} />

    <!-- <Face /> -->
    
    {#if showRsvp}
    <div class="modal" transition:fade on:click={handleModalClick}>
        <Rsvp on:success={handleFormSuccess} />
        </div>
    {/if}

    {#if showThankYou}
    <div class="thank-you" transition:fade>
        <h2 class="font-Zuume">Excited to see you! </h2>
        <p class="font-Bern">More details to follow.</p>
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
        /* background-image: url('/assets/bg1.jpg'); */
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
        width: 90%;
        max-width: 320px;
        background-color: #343233;
        border: 2px solid #FEFA99;
        border-radius: 20px;
        box-shadow: 0 4px 20px rgba(254, 250, 153, 0.2);
        overflow: hidden;
        z-index: 1001;
        padding: 15px;
        display: flex;
        flex-direction: column;
        align-items: center;
        animation: heartbeat 1.5s ease-in-out infinite;
    }

    @keyframes heartbeat {
        0%, 100% {
            transform: translate(-50%, -50%) scale(1);
        }
        50% {
            transform: translate(-50%, -50%) scale(1.05);
        }
    }

    .thank-you p {
        font-size: 14px;
        color: #FEFA99;
        text-align: center;
        margin-bottom: 10px;
    }

    .thank-you h2 {
        font-size: 50px;
        text-align: center;
        margin: 0;
        background-color: #FEFA99;
        color: #343233;
        padding: 6px 3px;
        box-shadow: 0 2px 4px rgba(254, 250, 153, 0.2);
        width: 100%;
        box-sizing: border-box;
    }

    /* Add these classes for font styling */
    .font-Zuume {
        font-family: 'Zuume', sans-serif;
    }

    .font-Bern {
        font-family: 'Bernoru', sans-serif;
    }
</style>
