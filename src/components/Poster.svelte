<script>
	import { onMount, createEventDispatcher } from 'svelte';
	import { tweened } from 'svelte/motion';
	import { cubicOut } from 'svelte/easing';
	import Face from './Face.svelte';
  console.log('Poster: Face component imported');

	let currentState = 'normal';
	let morphTo;
	let posterContainer;
	let scaleFactor = 1;
	const REFERENCE_WIDTH = 420;
	const REFERENCE_HEIGHT = 594;

	let isDragging = false;
	let startX, startY;
	let currentX = 0,
		currentY = 0;
	let facePosition;
	let ovalTopElement;

	const progress = tweened(0, {
		duration: 1000,
		easing: cubicOut
	});

	$: scale = 1 - $progress * 0.8; // Shrink to 50% of original size
	$: rotation = $progress * 360; // Rotate 360 degrees
	$: ovalStyle = `transform: translate(${currentX}px, ${currentY}px) scale(${scale}) rotate(${rotation}deg);`;


  let fontLoaded = false;
    let assetsLoaded = false;
    let allLoaded = false;
    let faceLoaded = false;
    

	const dispatch = createEventDispatcher();

  function handleFaceLoaded() {
  console.log('Poster: handleFaceLoaded called');
  faceLoaded = true;
  console.log('Poster: faceLoaded set to true');
  checkAllLoaded();
}

function checkAllLoaded() {
  console.log('Poster: Checking all loaded:', { fontLoaded, assetsLoaded, faceLoaded });
  if (fontLoaded && assetsLoaded && faceLoaded) {
    console.log('Poster: All loaded, setting allLoaded to true');
    allLoaded = true;
    dispatch('loaded');
  } else {
    console.log('Poster: Not all loaded yet');
  }
}

 
    function preloadImage(src) {
        return new Promise((resolve, reject) => {
            const img = new Image();
            img.onload = () => resolve(img);
            img.onerror = reject;
            img.src = src;
        });
    }


	function startDrag(event) {
		isDragging = true;
		if (event.type === 'mousedown') {
			startX = event.clientX - currentX;
			startY = event.clientY - currentY;
			window.addEventListener('mousemove', drag);
			window.addEventListener('mouseup', stopDrag);
		} else if (event.type === 'touchstart') {
			startX = event.touches[0].clientX - currentX;
			startY = event.touches[0].clientY - currentY;
			window.addEventListener('touchmove', drag);
			window.addEventListener('touchend', stopDrag);
		}
		event.preventDefault(); // Prevent text selection
	}

	function drag(event) {
		if (!isDragging) return;
		let clientX, clientY;
		if (event.type === 'mousemove') {
			clientX = event.clientX;
			clientY = event.clientY;
		} else if (event.type === 'touchmove') {
			clientX = event.touches[0].clientX;
			clientY = event.touches[0].clientY;
		}
		currentX = clientX - startX;
		currentY = clientY - startY;
		checkFaceCollision();
		event.preventDefault(); // Prevent scrolling on touch devices
	}

	function stopDrag() {
		isDragging = false;
		window.removeEventListener('mousemove', drag);
		window.removeEventListener('mouseup', stopDrag);
		window.removeEventListener('touchmove', drag);
		window.removeEventListener('touchend', stopDrag);
	}

	function checkFaceCollision() {
		if (!facePosition || !ovalTopElement) return;

		const faceRect = facePosition.getBoundingClientRect();
		const ovalRect = ovalTopElement.getBoundingClientRect();

		if (
			ovalRect.left < faceRect.right &&
			ovalRect.right > faceRect.left &&
			ovalRect.top < faceRect.bottom &&
			ovalRect.bottom > faceRect.top
		) {
			stopDrag();
			animateToFace();
		}
	}

async function animateToFace() {
  // Get the bounding rectangle of the face element
  const faceRect = facePosition.getBoundingClientRect();
  console.log("🚀 ~ animateToFace ~ faceRect:", faceRect)

  // Get the bounding rectangle of the oval element
  const ovalRect = ovalTopElement.getBoundingClientRect();

  // Get the bounding rectangle of the poster container
  const posterRect = posterContainer.getBoundingClientRect();

  // Calculate the target X position to center the oval horizontally on the face
  const targetX = faceRect.left + faceRect.width / 2 - ovalRect.width / 2 - ovalRect.left;

  // Calculate the target Y position, adjusting for the scaleFactor
  const targetY = -144.2

  console.log("🚀 ~ animateToFace ~ targetY:", targetY)

  // Animate the progress to 1 (full animation)
  await progress.set(1);
  // Update the current X and Y positions to the target positions
  currentX = targetX;
  currentY = targetY;

    await handleMorph('mouth');
    await new Promise((resolve) => setTimeout(resolve, 500));

    // Make the oval disappear
    ovalTopElement.style.opacity = '0';
    await new Promise((resolve) => setTimeout(resolve, 300)); // Wait for the fade-out animation

    await handleMorph('normal');
    await new Promise((resolve) => setTimeout(resolve, 500));
    await handleMorph('wink');
    await new Promise((resolve) => setTimeout(resolve, 200));
    await handleMorph('normal');
    await new Promise((resolve) => setTimeout(resolve, 100));


  

    handleOvalClick()

}

	function handleMorphReady(event) {
		morphTo = event.detail.morphTo;
	}

	function handleMorph(newState) {
		console.log('handleMorph called with', newState);
		if (morphTo) {
			morphTo(newState);
			currentState = newState;
			console.log('Morphed to', newState);
		} else {
			console.log('morphTo function not available');
		}
	}

	function updateScale() {
		if (!posterContainer) return;
		const { width, height } = posterContainer.getBoundingClientRect();
		scaleFactor = height / REFERENCE_HEIGHT;
	}


	function handleOvalClick() {
		dispatch('openRsvp');
	}

  onMount(() => {
        const fontZuume = new FontFace('Zuume', 'url(/zuume.woff2)');
        const fontBern = new FontFace('Bernoru', 'url(/bernoru.woff2)');

        const assetUrls = [
            '/assets/bg2.jpg',
            '/assets/heart.svg',
            '/assets/smiley.svg',
            '/assets/oval.svg',
            // Add any other asset URLs here
        ];

        Promise.all([
            fontZuume.load(),
            fontBern.load(),
            ...assetUrls.map(preloadImage)
        ])
        .then(([zuumeFont, bernFont, ...loadedImages]) => {
            document.fonts.add(zuumeFont);
            document.fonts.add(bernFont);
            fontLoaded = true;
            assetsLoaded = true;
            checkAllLoaded();
        })
        .catch((error) => {
            console.error('Error loading fonts or assets:', error);
            // Optionally set fontLoaded and assetsLoaded to true here if you want to proceed even if loading fails
        });

  

		updateScale();
		window.addEventListener('resize', updateScale);
		return () => window.removeEventListener('resize', updateScale);
	});

	let celebrationText = 'FEEL THE LOVE. COME CELEBRATE WITH US';
</script>

<div class="poster" bind:this={posterContainer}>
	<!-- <div class="template-overlay"></div> -->

	<div class="poster-content" style="transform: scale({scaleFactor});">
		<div class="banner">
			<h1 class:font-Zuume={fontLoaded}>SMILE LIKE IT'S A WEDDING</h1>
		</div>

		<div class="header">
			<h2 class:font-Bern={fontLoaded}>CECI & JACK ARE EXCITED TO INVITE</h2>
			<h2 class:font-Bern={fontLoaded}>YOU TO CELEBRATE THEIR WEDDING</h2>
		</div>

		<div class="small-hearts-row">
			<img src="/assets/heart.svg" alt="Heart" class="small-heart" />
			<img src="/assets/heart.svg" alt="Heart" class="small-heart" />
		</div>

		<div class="large-hearts-row">
			<img src="/assets/heart.svg" alt="Heart" class="large-heart" />
			<img src="/assets/heart.svg" alt="Heart" class="large-heart" />
		</div>

    <div class="face-container" bind:this={facePosition}>
      {#if true}
        <Face on:morphReady={handleMorphReady} on:faceLoaded={handleFaceLoaded} />
        {console.log('Poster: Face component rendered')}
      {/if}
      </div>


		<div class="celebration-text">
			<svg viewBox="0 0 500 250" width="180%" height="100%">
				<path id="curve" d="M10,10 Q250,240 490,10" fill="none" />
				<text class:font-Zuume={fontLoaded}>
					<textPath href="#curve" startOffset="50%" text-anchor="middle">
						{celebrationText}
					</textPath>
				</text>
			</svg>
		</div>

		<div class="smiley-row">
			<img src="/assets/smiley.svg" alt="Smiley" class="smiley left" />
			<img src="/assets/smiley.svg" alt="Smiley" class="smiley right" />
		</div>

    <div class="oval-diamond">
      <div
      class="oval-container top"
      bind:this={ovalTopElement}
      on:mousedown={startDrag}
      on:touchstart={startDrag}
      style={ovalStyle}
  >
      <img src="/assets/oval.svg" alt="Oval" class="oval-image" />
      <svg viewBox="0 0 249 70" class="oval-outline">
          <path class="dashed-outline" d="M248.232 34.7679C248.232 35.8991 248.033 37.0237 247.635 38.1483C247.237 39.2729 246.639 40.3909 245.85 41.4955C245.053 42.6068 244.072 43.6981 242.891 44.7828C241.71 45.8608 240.344 46.9255 238.792 47.9703C237.234 49.015 235.504 50.0332 233.589 51.0313C231.673 52.0295 229.586 52.9944 227.335 53.9327C225.076 54.8709 222.66 55.7826 220.085 56.6543C217.511 57.5261 214.785 58.3645 211.909 59.1631C209.039 59.9616 206.032 60.7202 202.895 61.4389C199.758 62.1509 196.497 62.823 193.119 63.4552C189.749 64.0807 186.268 64.6596 182.684 65.192C179.101 65.7244 175.435 66.2101 171.687 66.6427C167.931 67.0752 164.114 67.4545 160.229 67.7806C156.343 68.1067 152.41 68.3861 148.421 68.6057C144.439 68.8253 140.43 68.9917 136.387 69.0982C132.343 69.2113 128.293 69.2645 124.235 69.2645C120.171 69.2645 116.121 69.2113 112.077 69.0982C108.034 68.9917 104.025 68.8253 100.043 68.6057C96.0612 68.3861 92.1207 68.1067 88.235 67.7806C84.3494 67.4545 80.5324 67.0752 76.7773 66.6427C73.0289 66.2101 69.363 65.7244 65.7795 65.192C62.1959 64.6596 58.7222 64.0807 55.3446 63.4552C51.967 62.823 48.7061 62.1509 45.5687 61.4389C42.4314 60.7202 39.4245 59.9616 36.5549 59.1631C33.6785 58.3645 30.9599 57.5261 28.3786 56.6543C25.8043 55.7826 23.3878 54.8709 21.1292 53.9327C18.8774 52.9944 16.7904 52.0295 14.8751 51.0313C12.9597 50.0332 11.2297 49.015 9.67138 47.9703C8.11988 46.9255 6.75373 45.8608 5.57294 44.7828C4.39216 43.6981 3.41045 42.6068 2.61411 41.4955C1.82463 40.3909 1.22737 39.2729 0.829193 38.1483C0.431021 37.0237 0.231934 35.8991 0.231934 34.7679C0.231934 33.6366 0.431021 32.512 0.829193 31.3874C1.22737 30.2628 1.82463 29.1448 2.61411 28.0335C3.41045 26.9289 4.39216 25.8309 5.57294 24.7529C6.75373 23.6682 8.11988 22.6102 9.67138 21.5654C11.2297 20.5207 12.9597 19.5025 14.8751 18.5044C16.7904 17.5062 18.8774 16.5413 21.1292 15.603C23.3878 14.6581 25.8043 13.7531 28.3786 12.8814C30.9599 12.0096 33.6785 11.1712 36.5549 10.3726C39.4245 9.5741 42.4314 8.8155 45.5687 8.09681C48.7061 7.37813 51.967 6.70604 55.3446 6.08052C58.7222 5.455 62.1959 4.87606 65.7795 4.3437C69.363 3.81135 73.0289 3.32557 76.7773 2.89303C80.5324 2.4605 84.3494 2.08119 88.235 1.75512C92.1207 1.4224 96.0612 1.14957 100.043 0.929971C104.025 0.710373 108.034 0.544013 112.077 0.430888C116.121 0.324416 120.171 0.264526 124.235 0.264526C128.293 0.264526 132.343 0.324416 136.387 0.430888C140.43 0.544013 144.439 0.710373 148.421 0.929971C152.41 1.14957 156.343 1.4224 160.229 1.75512C164.114 2.08119 167.931 2.4605 171.687 2.89303C175.435 3.32557 179.101 3.81135 182.684 4.3437C186.268 4.87606 189.749 5.455 193.119 6.08052C196.497 6.70604 199.758 7.37813 202.895 8.09681C206.032 8.8155 209.039 9.5741 211.909 10.3726C214.785 11.1712 217.511 12.0096 220.085 12.8814C222.66 13.7531 225.076 14.6581 227.335 15.603C229.586 16.5413 231.673 17.5062 233.589 18.5044C235.504 19.5025 237.234 20.5207 238.792 21.5654C240.344 22.6102 241.71 23.6682 242.891 24.7529C244.072 25.8309 245.053 26.9289 245.85 28.0335C246.639 29.1448 247.237 30.2628 247.635 31.3874C248.033 32.512 248.232 33.6366 248.232 34.7679Z" fill="none" stroke="#fefa99" stroke-width="2" />
      </svg>
      <div class="oval-text top-text">DRAG TO MOUTH TO RSVP</div>
  </div>  


      <div class="oval-middle">
          <img src="/assets/oval.svg" alt="Oval" class="oval left" />
          <text class="oval-text left-text">BRISTOL</text>
  
          <img src="/assets/oval.svg" alt="Oval" class="oval right" />
          <text class="oval-text right-text">29.03.2025</text>
      </div>
  
      <img src="/assets/oval.svg" alt="Oval" class="oval bottom" />
      <text class="oval-text bottom-text">MORE DETAILS TO FOLLOW</text>
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
		background-color: #fefa99;
		box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
		position: relative;
		overflow: hidden;
	}

	.poster-content {
        width: 420px;
        height: 594px;
        -webkit-transform-origin: top left;
        transform-origin: top left;
        -webkit-text-size-adjust: 100%;
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
		text-align: center;
        display: -webkit-flex;
        display: flex;
        -webkit-justify-content: center;
        justify-content: center;
        -webkit-align-items: center;
        align-items: center;
        left: 50%;
        -webkit-transform: translateX(-50%);
        transform: translateX(-50%);

		white-space: nowrap;
		overflow: hidden;
    }

	h1 {
		color: #fefa99;
		font-size: 61px;
		letter-spacing: -2px;
		margin: 0;
		padding-top: 4px;
		text-align: center;

		white-space: nowrap;
    -webkit-text-size-adjust: none;
    width: 100%;
    display: block;
    line-height: 1;
	}

/* Safari desktop and mobile only */
@supports (-webkit-touch-callout: none) and (-webkit-appearance: none) {
    h1 {
        font-size: 54px;
        letter-spacing: -3px;
    }
}
	.header {
		position: absolute;
		top: 86px;
		left: 50%;
		transform: translateX(-50%);
		width: 400px;
		text-align: center;
		-webkit-transform: translateX(-50%);
        transform: translateX(-50%);
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
		white-space: nowrap;
    -webkit-text-size-adjust: none;
    width: 100%;
    display: block;
		
	}

	.header h2:first-child {
		margin-bottom: -2px; /* Adjust this value to control spacing between the two lines */
		letter-spacing: -1px;
		font-size: 17px;
		white-space: nowrap;
    -webkit-text-size-adjust: none;
    width: 100%;
    display: block;
	}

	/* Safari desktop and mobile only */
@supports (-webkit-touch-callout: none) and (-webkit-appearance: none) {
    .header h2 {
		letter-spacing: -2px;
		font-size: 17px;
    }

	.header h2:first-child {
		letter-spacing: -2px;
		margin-bottom: -1px; /* Adjust this value to control spacing between the two lines */
		font-size: 17px;
	}
}

	.large-hearts-row {
		position: absolute;
		top: 112px; /* Adjust this value to position the hearts below the header */
		left: 0;
		right: 0;
		display: flex;
		justify-content: space-between;
		align-items: center; /* This will vertically align the hearts */
		padding: 0 62px; /* Adjust this value to control how close to the edges the smileys are */
	}

	.small-hearts-row {
		position: absolute;
		top: 172px; /* Adjust this value to position the hearts below the header */
		left: 0;
		right: 0;
		display: flex;
		justify-content: space-between;
		align-items: center; /* This will vertically align the hearts */
		padding: 0 8px; /* Adjust this value to control how close to the edges the smileys are */
	}

	.large-heart {
		width: 85px; /* Adjust the size for smaller hearts */
		height: 85px;
	}

	.small-heart {
		width: 80px; /* Adjust the size for larger hearts */
		height: 45px;
	}

	.face-container {
		position: absolute;
		top: 165px; /* Adjust this value to position the face below the hearts */
		left: 49.5%;
		transform: translateX(-50%);
		width: 100%;
		display: flex;
		justify-content: center;
		transform-style: preserve-3d;
    -webkit-transform-style: preserve-3d;
    backface-visibility: visible;
    -webkit-backface-visibility: visible;
}

/* Update the global SVG styles */
:global(.face-container svg) {
  transform: scale(1, -1);
  -webkit-transform: scale(1, -1);
  width: 85%;
  height: 100%;
}

	.celebration-text {
		position: absolute;
		top: 255px; /* Adjust this value as needed */
		left: 11%;
		transform: translateX(-50%);
		width: 100%;
		text-align: center;
	}

	.celebration-text svg {
		overflow: visible;
	}

	.celebration-text text {
		fill: #343233; /* Text color */
		font-size: 23px; /* Adjust as needed */
	}

	.smiley-row {
		position: absolute;
		bottom: 87px; /* Adjust this value to position the smileys from the bottom */
		left: 0;
		right: 0;
		display: flex;
		justify-content: space-between;
		padding: 0 25px; /* Adjust this value to control how close to the edges the smileys are */
	}

	.smiley {
		width: 60px; /* Adjust the size as needed */
		height: 60px;
	}


	
  .oval-container.top {
        position: relative;
    }

    .oval-container.top .oval-image {
        width: 100%;
        height: 100%;
    }

    .oval-container.top .oval-outline {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        pointer-events: none;
    }

    .oval-container.top .dashed-outline {
        stroke-dasharray: 10 8; /* Shorter dashes and gaps */
        stroke-width: 8;

        stroke-dashoffset: 0;
        animation: dash 20s linear infinite;
    }

    @keyframes dash {
        to {
            stroke-dashoffset: 300;
        }
    }

    .oval-container.top {
        position: absolute;
        z-index: 10;
        bottom: 55px;
        width: 179px;
        height: 90px;
        left: 29%;
        transform: translateX(-50%);
        cursor: grab;
        transition: transform 0.3s ease, opacity 0.3s ease;
    }

.oval-container.top .oval-image {
    width: 100%;
    height: 100%;
}

.oval-container.top .oval-text {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    z-index: 11;
    width: 150px;
    font-size: 12px;
    text-align: center;
    color: #fefa99;
    font-family: 'Bernoru', sans-serif;
    pointer-events: none;
}

/* Update the existing .oval-text styles */
.oval-text {
    position: absolute;
    transform: translate(-50%, -50%);
    font-family: 'Bernoru', sans-serif;
    color: #fefa99;
    pointer-events: none;
    text-align: center;
}

	.oval.bottom {
		transform: rotate(0deg);
		position: absolute;
		bottom: 15px;
		width: 130px;
		height: 50px;
		left: 50.7%;
		transform: translateX(-50%);
	}

	.oval.left {
		transform: rotate(-0deg);
		position: absolute;
		bottom: 30px;
		width: 100px;
		height: 50px;
		left: 21%;
		transform: translateX(-50%);
	}

	.oval.right {
		transform: rotate(0deg);
		position: absolute;
		bottom: 30px;
		width: 100px;
		height: 50px;
		left: 80%;
		transform: translateX(-50%);
	}

	.oval-text {
		position: absolute;
		transform: translate(-50%, -50%);
		font-family: 'Bernoru', sans-serif;
		color: #fefa99;
		pointer-events: none;
		text-align: center;
	}



	.bottom-text {
		bottom: 20px;
		left: 50.7%;
		font-size: 9px;
		width: 110px;
	}

	.left-text {
		bottom: 45px;
		left: 21%;
		font-size: 10px;
	}

	.right-text {
		bottom: 45px;
		left: 80%;
		font-size: 10px;
	}
</style>
