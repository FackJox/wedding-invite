<script>
	import { onMount, createEventDispatcher } from 'svelte';
	import { tweened } from 'svelte/motion';
	import { cubicOut } from 'svelte/easing';
	import Face from './Face.svelte';

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

	const dispatch = createEventDispatcher();

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
    const faceRect = facePosition.getBoundingClientRect();
    const ovalRect = ovalTopElement.getBoundingClientRect();

    const targetX = faceRect.left + faceRect.width / 2 - ovalRect.width / 2 - ovalRect.left;
    const targetY = faceRect.top + faceRect.height / 1.75 - ovalRect.height / 2 - ovalRect.top;

    await progress.set(1);
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
		scaleFactor = width / REFERENCE_WIDTH;
	}

	let fontLoaded = false;

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
			<Face on:morphReady={handleMorphReady} />
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
          <div class="oval-shape"></div>
          <div class="oval-text top-text">DRAG TO MOUTH TO RSVP</div>
      </div>
  
      <div class="oval-middle">
          <img src="/assets/oval.svg" alt="Oval" class="oval left" />
          <text class="oval-text left-text">BRISTOL</text>
  
          <img src="/assets/oval.svg" alt="Oval" class="oval right" />
          <text class="oval-text right-text">30.03.2025</text>
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
		text-align: center;
	}

	h1 {
		color: #fefa99;
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
	}

	.face {
		width: 259px; /* Adjust the size as needed */
		height: auto; /* This maintains the aspect ratio */
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
