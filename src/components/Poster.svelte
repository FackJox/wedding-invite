<script>
	import { onMount, createEventDispatcher } from 'svelte';
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
    const dispatch = createEventDispatcher();

function handleOvalClick() {
    dispatch('openRsvp');
}

	onMount(() => {
		const fontZuume = new FontFace('Zuume', 'url(/zuume.woff2)');
		fontZuume.load().then(() => {
			document.fonts.add(fontZuume);
			fontLoaded = true;
		});

        const fontBern = new FontFace('Bernoru', 'url(/bernoru.woff2)');
		fontBern.load().then(() => {
			document.fonts.add(fontBern);
			fontLoaded = true;
		});

		updateScale();
		window.addEventListener('resize', updateScale);
		return () => window.removeEventListener('resize', updateScale);
	});

    let celebrationText = "FEEL THE LOVE. COME CELEBRATE WITH US";

</script>


<div class="poster" bind:this={posterContainer}>
    <div class="template-overlay"></div>

	<div class="poster-content" style="transform: scale({scaleFactor});">
        
        <div class="banner">
            <h1 class:font-Zuume={fontLoaded}>SMILE LIKE IT'S A WEDDING</h1>
          </div>

          <div class="header">
            <h2 class:font-Bern={fontLoaded}>CECI & JACK ARE EXCITED TO INVITE</h2>
            <h2 class:font-Bern={fontLoaded}>YOU TO CELEBRATE THEIR WEDDING</h2>
        </div>

        <div class="hearts-row">
            <img src="/assets/heart.svg" alt="Heart" class="heart small">
            <img src="/assets/heart.svg" alt="Heart" class="heart large">
            <img src="/assets/heart.svg" alt="Heart" class="heart large">
            <img src="/assets/heart.svg" alt="Heart" class="heart small">
          </div>

          <div class="face-container">
            <img src="/assets/face.svg" alt="Face" class="face">
          </div>

          <div class="celebration-text">
            <svg viewBox="0 0 500 250" width="100%" height="250">
              <path id="curve" d="M10,10 Q250,240 490,10" fill="none" />
              <text class:font-Zuume={fontLoaded}>
                <textPath href="#curve" startOffset="50%" text-anchor="middle">
                  {celebrationText}
                </textPath>
              </text>
            </svg>
          </div>
          
          

          <div class="smiley-row">
            <img src="/assets/smiley.svg" alt="Smiley" class="smiley left">
            <img src="/assets/smiley.svg" alt="Smiley" class="smiley right">
          </div>

          <div class="oval-diamond">
            <img src="/assets/oval.svg" alt="Oval" class="oval top" on:click={handleOvalClick}>
            <div class="oval-middle">
              <img src="/assets/oval.svg" alt="Oval" class="oval left">
              <img src="/assets/oval.svg" alt="Oval" class="oval right">
            </div>
            <img src="/assets/oval.svg" alt="Oval" class="oval bottom">
        </div>

	</div>
</div>

<style>

.template-overlay {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-image: url('/assets/layout.png');
        background-size: cover;
        background-position: center;
        background-repeat: no-repeat;
        opacity: 0.3;
        pointer-events: none;
        z-index: 10;
    }

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
	}

    .font-Zuume {
      font-family: 'Zuume', sans-serif;
      opacity: 1;
    }

    .font-Bern {
      font-family: 'Bernoru', sans-serif;
      opacity: 1;
    }


    .banner {
      width: 405px;
      height: 72px;
      margin-top: 10px;
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
      font-size: 61px;
      letter-spacing: -2px;
      margin: 0;
      padding-top: 4px;
      text-align: center;
    }

    .header {
    position: absolute;
    top: 86px;
    left: 50%;
    transform: translateX(-50%);
    width: 400px;
    text-align: center;
  }

  .header h2 {
    color: #343233;
    font-size: 16px;
    margin: 0;
    padding: 0;
    position: relative;
    top: auto;
    left: auto;
    transform: none;
  }

  .header h2:first-child {
    margin-bottom: -2px; /* Adjust this value to control spacing between the two lines */
    letter-spacing: -1px;
    font-size: 17px;

}

.hearts-row {
    position: absolute;
    top: 150px; /* Adjust this value to position the hearts below the header */
    left: 50%;
    transform: translateX(-50%);
    display: flex;
    justify-content: center;
    align-items: center; /* This will vertically align the hearts */
    width: 100%;
  }

  .heart {
    margin: 0 5px; /* Adjust the spacing between hearts */
  }

  .heart.small {
    width: 20px; /* Adjust the size for smaller hearts */
    height: 20px;
  }

  .heart.large {
    width: 30px; /* Adjust the size for larger hearts */
    height: 30px;
  }

  .face-container {
    position: absolute;
    top: 200px; /* Adjust this value to position the face below the hearts */
    left: 50%;
    transform: translateX(-50%);
    width: 100%;
    display: flex;
    justify-content: center;
  }

  .face {
    width: 100px; /* Adjust the size as needed */
    height: auto; /* This maintains the aspect ratio */
  }

  .celebration-text {
    position: absolute;
    top: 300px; /* Adjust this value as needed */
    left: 50%;
    transform: translateX(-50%);
    width: 100%;
    text-align: center;
  }

  .celebration-text svg {
    overflow: visible;
  }

  .celebration-text text {
    fill: #343233; /* Text color */
    font-size: 24px; /* Adjust as needed */
  }

  .smiley-row {
    position: absolute;
    bottom: 190px; /* Adjust this value to position the smileys from the bottom */
    left: 0;
    right: 0;
    display: flex;
    justify-content: space-between;
    padding: 0 40px; /* Adjust this value to control how close to the edges the smileys are */
  }

  .smiley {
    width: 50px; /* Adjust the size as needed */
    height: 50px;
  }

  .smiley.left {
    transform: rotate(-15deg); /* Optional: adds a slight tilt to the left smiley */
  }

  .smiley.right {
    transform: rotate(15deg); /* Optional: adds a slight tilt to the right smiley */
  }

  .oval.top {
    transform: rotate(0deg);
    position: absolute; 
    bottom: 40px;
    width: 130px; 
    height: 90px;
    left: 50%;
    transform: translateX(-50%);
    cursor: pointer;

  }

  .oval.bottom {
    transform: rotate(0deg);
    position: absolute; 
    bottom: 10px;
    width: 100px; 
    height: 50px;
    left: 50%;
    transform: translateX(-50%);
  }

  .oval.left {
    transform: rotate(-0deg);
    position: absolute; 
    bottom: 30px;
    width: 90px; 
    height: 50px;
    left: 20%;
    transform: translateX(-50%);
  }

  .oval.right {
    transform: rotate(0deg);
    position: absolute; 
    bottom: 30px;
    width: 90px; 
    height: 50px;
    left: 80%;
    transform: translateX(-50%);
  }
  
   
</style>
