<script>
    import { onMount } from 'svelte';
    import Face from '../components/Face.svelte';
    import Poster from '../components/Poster.svelte';
    import Rsvp from '../components/Rsvp.svelte';
    import { fade } from 'svelte/transition';

    let showRsvp = false;
    let showThankYou = false;
    let isLoaded = false;
    let rsvpLoaded = false;

    onMount(() => {
    // Preload the RSVP component
    setTimeout(() => {
      rsvpLoaded = true;
    }, 1000); // Delay for 1 second to ensure other components load first
  });


    function handleLoaded() {
        console.log('Poster component loaded');
        const minLoadTime = 1200; // 1.2 seconds minimum loading time
        const startTime = Date.now();
        const elapsedTime = Date.now() - startTime;

        if (elapsedTime >= minLoadTime) {
            isLoaded = true;
        } else {
            setTimeout(() => {
                isLoaded = true;
            }, minLoadTime - elapsedTime);
        }
    }

    
   
    console.log("🚀 ~ isLoaded:", isLoaded)

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
    {#if !isLoaded}
    <div class="loader" transition:fade={{ duration: 500 }}>
        <img src="/assets/ogface.svg" alt="Loading..." />
    </div>
    {/if}

<Poster on:loaded={handleLoaded} on:openRsvp={toggleRsvp} />

    <!-- <Face /> -->
    
    {#if rsvpLoaded}
    <div class="rsvp-preload" class:show={showRsvp}>
      <div class="modal" transition:fade on:click={handleModalClick}>
        <Rsvp on:success={handleFormSuccess} />
      </div>
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
    :global(body) {
        margin: 0;
        padding: 0;
        overflow: hidden;
    }

      /* Add these font-face declarations if they're not already in your global styles */
  @font-face {
    font-family: 'Zuume';
    src: url('/zuume.woff2') format('woff2');
    font-display: swap;
  }

  @font-face {
    font-family: 'Bernoru';
    src: url('/bernoru.woff2') format('woff2');
    font-display: swap;
  }

  .rsvp-preload {
    display: none;
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: -1;
  }

  .rsvp-preload.show {
    display: block;
    z-index: 1000;
  }

	/* Fill the entire viewport and center children */
	main {
		width: 100vw;
		height: 100vh;
		margin: 0;
		padding: 0;
		display: flex;
		justify-content: center;
		align-items: center;
        background-image: url('/assets/bg2.jpg');
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

    .loader {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-color: #FEFA99;
        display: flex;
        justify-content: center;
        align-items: center;
        z-index: 2000;
    }

    .loader img {
        width: 100px;
        height: 100px;
        animation: spin 2s linear infinite;
    }

    @keyframes spin {
        0% { transform: rotate(0deg); }
        100% { transform: rotate(360deg); }
    }

    /* Add these classes for font styling */
    .font-Zuume {
        font-family: 'Zuume', sans-serif;
    }

    .font-Bern {
        font-family: 'Bernoru', sans-serif;
    }
</style>
