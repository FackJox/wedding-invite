<script>
	import { onMount, onDestroy } from 'svelte';
	import { browser } from '$app/environment';
	import * as THREE from 'three';
	import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js';
	import { EffectComposer, RenderPass, EffectPass, BloomEffect } from 'postprocessing';
	import { CrtAsciiEffect } from '$lib/shaders/CrtAsciiEffect.js';

	// Effect parameters - passed from parent (no default to avoid creating separate object)
	export let effectParams;

	// Disco ball color - matching reference red/coral
	const DISCO_COLOR = 0xe05050;
	const SPARKLE_COLOR = 0xe05050;

	let canvas;
	let mounted = false;
	let renderer;
	let scene;
	let camera;
	let composer;
	let discoBallModel;
	let decorations = [];
	let animationId;

	// Effect references
	let crtAsciiEffect = null;
	let bloomEffect = null;

	function initThree() {
		scene = new THREE.Scene();
		scene.background = null; // Transparent

		// Orthographic camera for flat poster look
		const aspect = canvas.width / canvas.height;
		const frustumSize = 3;
		camera = new THREE.OrthographicCamera(
			-frustumSize * aspect / 2,
			frustumSize * aspect / 2,
			frustumSize / 2,
			-frustumSize / 2,
			0.1,
			100
		);
		camera.position.z = 5;

		// Renderer
		renderer = new THREE.WebGLRenderer({
			canvas,
			alpha: true,
			antialias: true,
			powerPreference: 'high-performance'
		});
		renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
		renderer.outputColorSpace = THREE.SRGBColorSpace;
		renderer.toneMapping = THREE.ACESFilmicToneMapping;
		renderer.toneMappingExposure = 1.2;

		// Lighting
		const ambientLight = new THREE.AmbientLight(0xffffff, 0.4);
		scene.add(ambientLight);

		const directionalLight = new THREE.DirectionalLight(0xffffff, 1.2);
		directionalLight.position.set(2, 3, 5);
		scene.add(directionalLight);

		const rimLight = new THREE.DirectionalLight(0xff6666, 0.6);
		rimLight.position.set(-2, -1, 3);
		scene.add(rimLight);

		// Post-processing setup
		composer = new EffectComposer(renderer);
		const renderPass = new RenderPass(scene, camera);
		composer.addPass(renderPass);

		// CRT ASCII Effect
		crtAsciiEffect = new CrtAsciiEffect({
			cellSize: effectParams.cellSize,
			invert: effectParams.invert,
			colorMode: effectParams.colorMode,
			asciiStyle: effectParams.asciiStyle,
			scanlineIntensity: effectParams.scanlineIntensity,
			scanlineCount: effectParams.scanlineCount,
			curvature: effectParams.curvature,
			aberrationStrength: effectParams.aberrationStrength,
			vignetteIntensity: effectParams.vignetteIntensity,
			vignetteRadius: effectParams.vignetteRadius,
			noiseIntensity: effectParams.noiseIntensity,
			glitchIntensity: effectParams.glitchIntensity,
			glitchFrequency: effectParams.glitchFrequency,
			brightnessAdjust: effectParams.brightnessAdjust,
			contrastAdjust: effectParams.contrastAdjust,
			bloomMix: effectParams.bloomMix
		});

		// Bloom for glow
		bloomEffect = new BloomEffect({
			intensity: effectParams.bloomIntensity,
			luminanceThreshold: 0.3,
			luminanceSmoothing: 0.4,
			mipmapBlur: true
		});

		const effectPass = new EffectPass(camera, crtAsciiEffect, bloomEffect);
		composer.addPass(effectPass);

		return true;
	}

	function loadDiscoBall() {
		const loader = new GLTFLoader();

		loader.load(
			'/discoball.glb',
			(gltf) => {
				discoBallModel = gltf.scene;

				// Wireframe material - only tile outlines visible, front face only
				discoBallModel.traverse((child) => {
					if (child.isMesh) {
						child.material = new THREE.MeshBasicMaterial({
							color: DISCO_COLOR,
							wireframe: true,
							wireframeLinewidth: 2,
							side: THREE.FrontSide
						});
					}
				});

				discoBallModel.scale.setScalar(1.2);
				discoBallModel.position.set(0, 0, 0);
				scene.add(discoBallModel);

				addDecorations();
			},
			undefined,
			(error) => {
				console.error('Error loading disco ball:', error);
				createFallbackBall();
			}
		);
	}

	function createFallbackBall() {
		const geometry = new THREE.IcosahedronGeometry(0.8, 2);
		const material = new THREE.MeshBasicMaterial({
			color: DISCO_COLOR,
			wireframe: true,
			wireframeLinewidth: 2,
			side: THREE.FrontSide
		});
		discoBallModel = new THREE.Mesh(geometry, material);
		scene.add(discoBallModel);
		addDecorations();
	}

	function addDecorations() {
		// Ring around the ball
		const ringGeometry = new THREE.TorusGeometry(1.1, 0.02, 16, 100);
		const ringMaterial = new THREE.MeshBasicMaterial({ color: DISCO_COLOR });
		const ring = new THREE.Mesh(ringGeometry, ringMaterial);
		ring.rotation.x = Math.PI / 2;
		scene.add(ring);
		decorations.push(ring);

		// Second tilted ring
		const ring2 = ring.clone();
		ring2.rotation.x = Math.PI / 2;
		ring2.rotation.z = Math.PI / 6;
		scene.add(ring2);
		decorations.push(ring2);

		// Sparkle stars
		const sparkleTexture = createSparkleTexture();
		const sparklePositions = [
			{ x: 1.3, y: 0.8, scale: 0.15 },
			{ x: 1.5, y: 0.5, scale: 0.1 },
			{ x: -1.3, y: 0.9, scale: 0.12 },
			{ x: -1.4, y: -0.6, scale: 0.08 },
			{ x: 1.2, y: -0.7, scale: 0.1 },
			{ x: -1.1, y: 0.3, scale: 0.06 },
			{ x: 0.9, y: 1.1, scale: 0.08 }
		];

		sparklePositions.forEach(pos => {
			const sparkleMaterial = new THREE.SpriteMaterial({
				map: sparkleTexture,
				color: SPARKLE_COLOR,
				transparent: true,
				blending: THREE.AdditiveBlending
			});
			const sparkle = new THREE.Sprite(sparkleMaterial);
			sparkle.position.set(pos.x, pos.y, 0.1);
			sparkle.scale.setScalar(pos.scale);
			sparkle.userData.baseScale = pos.scale;
			sparkle.userData.phase = Math.random() * Math.PI * 2;
			scene.add(sparkle);
			decorations.push(sparkle);
		});

		// Wave decorations
		addWaveDecoration(0, 1.4, 1);
		addWaveDecoration(0, -1.4, -1);
	}

	function createSparkleTexture() {
		const offscreen = document.createElement('canvas');
		offscreen.width = 64;
		offscreen.height = 64;
		const ctx = offscreen.getContext('2d');

		ctx.fillStyle = 'white';
		ctx.beginPath();
		const cx = 32, cy = 32;
		const points = 4;
		const outerRadius = 30;
		const innerRadius = 8;

		for (let i = 0; i < points * 2; i++) {
			const radius = i % 2 === 0 ? outerRadius : innerRadius;
			const angle = (i * Math.PI) / points - Math.PI / 2;
			const x = cx + Math.cos(angle) * radius;
			const y = cy + Math.sin(angle) * radius;
			if (i === 0) ctx.moveTo(x, y);
			else ctx.lineTo(x, y);
		}
		ctx.closePath();
		ctx.fill();

		const texture = new THREE.CanvasTexture(offscreen);
		texture.needsUpdate = true;
		return texture;
	}

	function addWaveDecoration(x, y, direction) {
		const points = [];
		const segments = 40;
		const width = 1.8;
		const amplitude = 0.12;
		const frequency = 4;

		for (let i = 0; i <= segments; i++) {
			const t = (i / segments) - 0.5;
			const px = x + t * width;
			const py = y + Math.sin(t * Math.PI * frequency) * amplitude * direction;
			points.push(new THREE.Vector3(px, py, 0));
		}

		const geometry = new THREE.BufferGeometry().setFromPoints(points);
		const material = new THREE.LineBasicMaterial({ color: DISCO_COLOR, linewidth: 2 });
		const wave = new THREE.Line(geometry, material);
		scene.add(wave);
		decorations.push(wave);
	}

	function handleResize() {
		if (!canvas || !renderer || !camera) return;

		const parent = canvas.parentElement;
		if (!parent) return;

		// Use offsetWidth/Height to get pre-transform CSS dimensions
		// (getBoundingClientRect returns post-transform scaled dimensions)
		const width = parent.offsetWidth;
		const height = parent.offsetHeight;

		canvas.width = width;
		canvas.height = height;

		renderer.setSize(width, height);
		composer.setSize(width, height);

		if (crtAsciiEffect) {
			crtAsciiEffect.setSize(width, height);
		}

		// Update orthographic camera
		const aspect = width / height;
		const frustumSize = 3;
		camera.left = -frustumSize * aspect / 2;
		camera.right = frustumSize * aspect / 2;
		camera.top = frustumSize / 2;
		camera.bottom = -frustumSize / 2;
		camera.updateProjectionMatrix();
	}

	function animate(timestamp) {
		if (!renderer || !scene || !camera) return;

		const time = timestamp * 0.001;

		// Transform and rotate disco ball
		if (discoBallModel) {
			discoBallModel.position.set(effectParams.positionX || 0, effectParams.positionY || 0, 0);
			discoBallModel.scale.setScalar(effectParams.scale || 1.2);
			const spinSpeed = effectParams.spinSpeed ?? 0.1;
			const tilt = effectParams.tilt ?? 0.3;
			discoBallModel.rotation.x = tilt;
			discoBallModel.rotation.y = time * spinSpeed;
		}

		// Animate sparkles
		decorations.forEach(dec => {
			if (dec.isSprite && dec.userData.baseScale) {
				const twinkle = 1 + Math.sin(time * 3 + dec.userData.phase) * 0.3;
				dec.scale.setScalar(dec.userData.baseScale * twinkle);
			}
		});

		// Animate rings with tilt and size
		const ringTilt = effectParams.ringTilt ?? 0.5;
		const ringTilt2 = effectParams.ringTilt2 ?? 0.3;
		const ringSize = effectParams.ringSize ?? 1.1;
		decorations.forEach((dec, i) => {
			if (dec.geometry && dec.geometry.type === 'TorusGeometry') {
				dec.rotation.x = Math.PI / 2 + ringTilt;
				dec.rotation.y = ringTilt2;
				dec.rotation.z += 0.002 * (i % 2 === 0 ? 1 : -1);
				dec.scale.setScalar(ringSize);
			}
		});

		// Update effect params every frame (ensures reactivity works)
		if (crtAsciiEffect) {
			// Debug: log params occasionally
			if (Math.floor(time) % 5 === 0 && Math.floor(time * 10) % 10 === 0) {
				console.log('Effect params:', effectParams.cellSize, effectParams.asciiStyle, effectParams.scanlineIntensity);
			}
			crtAsciiEffect.cellSize = effectParams.cellSize;
			crtAsciiEffect.invert = effectParams.invert;
			crtAsciiEffect.colorMode = effectParams.colorMode;
			crtAsciiEffect.asciiStyle = effectParams.asciiStyle;
			crtAsciiEffect.scanlineIntensity = effectParams.scanlineIntensity;
			crtAsciiEffect.scanlineCount = effectParams.scanlineCount;
			crtAsciiEffect.curvature = effectParams.curvature;
			crtAsciiEffect.aberrationStrength = effectParams.aberrationStrength;
			crtAsciiEffect.vignetteIntensity = effectParams.vignetteIntensity;
			crtAsciiEffect.vignetteRadius = effectParams.vignetteRadius;
			crtAsciiEffect.noiseIntensity = effectParams.noiseIntensity;
			crtAsciiEffect.glitchIntensity = effectParams.glitchIntensity;
			crtAsciiEffect.glitchFrequency = effectParams.glitchFrequency;
			crtAsciiEffect.brightnessAdjust = effectParams.brightnessAdjust;
			crtAsciiEffect.contrastAdjust = effectParams.contrastAdjust;
			crtAsciiEffect.bloomMix = effectParams.bloomMix;
		}
		if (bloomEffect) {
			bloomEffect.intensity = effectParams.bloomIntensity;
		}

		// Render
		composer.render();

		animationId = requestAnimationFrame(animate);
	}

	onMount(async () => {
		if (!browser) return;
		mounted = true;

		await new Promise(r => setTimeout(r, 0));

		if (!initThree()) {
			console.error('Failed to initialize Three.js');
			return;
		}

		loadDiscoBall();
		handleResize();

		animationId = requestAnimationFrame(animate);
		window.addEventListener('resize', handleResize);
	});

	onDestroy(() => {
		if (!browser) return;

		if (animationId) {
			cancelAnimationFrame(animationId);
		}
		window.removeEventListener('resize', handleResize);

		if (renderer) renderer.dispose();
		if (composer) composer.dispose();
		decorations.forEach(dec => {
			if (dec.geometry) dec.geometry.dispose();
			if (dec.material) dec.material.dispose();
		});
	});
</script>

{#if mounted}
	<canvas bind:this={canvas} class="disco-ball-canvas"></canvas>
{/if}

<style>
	.disco-ball-canvas {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		display: block;
	}
</style>
