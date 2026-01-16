<script>
	import { onMount, onDestroy, tick } from 'svelte';
	import { browser } from '$app/environment';

	// Accept container element for sizing (optional, falls back to parent)
	export let container = null;

	let canvas;
	let gl;
	let program;
	let animationId;
	let textureLoaded = false;
	let mounted = false;

	// Uniform locations
	let uTimeLoc;
	let uOffsetLoc;
	let uResolutionLoc;

	// Animation state
	let startTime = 0;
	let panOffset = { x: 0, y: 0 };
	let panDirection = { x: 0.894, y: 0.447 }; // Default diagonal direction (normalized)
	const PAN_SPEED = 0.001; // Constant pan speed

	const VERTEX_SHADER = `
		attribute vec2 aPosition;
		void main() {
			gl_Position = vec4(aPosition, 0.0, 1.0);
		}
	`;

	const FRAGMENT_SHADER = `
		precision mediump float;

		uniform sampler2D uTexture;
		uniform float uTime;
		uniform vec2 uOffset;
		uniform vec2 uResolution;

		void main() {
			vec2 uv = gl_FragCoord.xy / uResolution;

			// Flip Y coordinate (WebGL has origin at bottom-left)
			uv.y = 1.0 - uv.y;

			// 1. Lava lamp distortion - multiple organic layers
			float t = uTime * 0.15;

			// Large slow blobs
			float blob1 = sin(uv.x * 1.5 + t * 0.7) * cos(uv.y * 1.2 + t * 0.5);
			float blob2 = cos(uv.x * 1.8 - t * 0.6) * sin(uv.y * 1.4 + t * 0.8);

			// Medium swirling motion
			float swirl = sin(uv.x * 3.0 + uv.y * 2.0 + t * 1.2) * 0.5;
			swirl += cos(uv.y * 2.5 - uv.x * 1.5 + t * 0.9) * 0.5;

			// Small ripples for detail
			float ripple1 = sin(uv.x * 6.0 + t * 2.0) * cos(uv.y * 5.0 - t * 1.5);
			float ripple2 = cos(uv.x * 5.0 - t * 1.8) * sin(uv.y * 7.0 + t * 2.2);

			// Combine distortions with different weights
			vec2 distort;
			distort.x = blob1 * 0.04 + swirl * 0.025 + ripple1 * 0.008;
			distort.y = blob2 * 0.04 + swirl * 0.02 + ripple2 * 0.008;

			uv += distort;

			// 2. Apply pan offset
			uv += uOffset;

			// 3. Scale with breathing effect (closer/further from camera)
			// Oscillates between ~0.45 and ~0.55 for subtle depth pulsing
			float breathe = sin(uTime * 0.2) * 0.05;
			float scale = 0.5 + breathe;

			// Scale from center of viewport
			uv = (uv - 0.5) * scale + 0.5;

			// 4. Sample texture (mirrored repeat handled by GPU)
			gl_FragColor = texture2D(uTexture, uv);
		}
	`;

	function createShader(type, source) {
		const shader = gl.createShader(type);
		gl.shaderSource(shader, source);
		gl.compileShader(shader);

		if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
			console.error('Shader compile error:', gl.getShaderInfoLog(shader));
			gl.deleteShader(shader);
			return null;
		}
		return shader;
	}

	function createProgram(vertexShader, fragmentShader) {
		const prog = gl.createProgram();
		gl.attachShader(prog, vertexShader);
		gl.attachShader(prog, fragmentShader);
		gl.linkProgram(prog);

		if (!gl.getProgramParameter(prog, gl.LINK_STATUS)) {
			console.error('Program link error:', gl.getProgramInfoLog(prog));
			return null;
		}
		return prog;
	}

	function initWebGL() {
		gl = canvas.getContext('webgl');
		if (!gl) {
			console.error('WebGL not supported');
			return false;
		}

		// Create shaders
		const vertexShader = createShader(gl.VERTEX_SHADER, VERTEX_SHADER);
		const fragmentShader = createShader(gl.FRAGMENT_SHADER, FRAGMENT_SHADER);
		if (!vertexShader || !fragmentShader) return false;

		// Create program
		program = createProgram(vertexShader, fragmentShader);
		if (!program) return false;

		gl.useProgram(program);

		// Create fullscreen quad
		const positions = new Float32Array([
			-1, -1,  1, -1,  -1, 1,
			-1,  1,  1, -1,   1, 1
		]);

		const positionBuffer = gl.createBuffer();
		gl.bindBuffer(gl.ARRAY_BUFFER, positionBuffer);
		gl.bufferData(gl.ARRAY_BUFFER, positions, gl.STATIC_DRAW);

		const aPositionLoc = gl.getAttribLocation(program, 'aPosition');
		gl.enableVertexAttribArray(aPositionLoc);
		gl.vertexAttribPointer(aPositionLoc, 2, gl.FLOAT, false, 0, 0);

		// Get uniform locations
		uTimeLoc = gl.getUniformLocation(program, 'uTime');
		uOffsetLoc = gl.getUniformLocation(program, 'uOffset');
		uResolutionLoc = gl.getUniformLocation(program, 'uResolution');

		return true;
	}

	function loadTexture() {
		const texture = gl.createTexture();
		const image = new Image();

		image.onload = () => {
			gl.bindTexture(gl.TEXTURE_2D, texture);
			gl.texImage2D(gl.TEXTURE_2D, 0, gl.RGBA, gl.RGBA, gl.UNSIGNED_BYTE, image);

			// Use MIRRORED_REPEAT for seamless infinite tiling
			gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_S, gl.MIRRORED_REPEAT);
			gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_WRAP_T, gl.MIRRORED_REPEAT);
			gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MIN_FILTER, gl.LINEAR);
			gl.texParameteri(gl.TEXTURE_2D, gl.TEXTURE_MAG_FILTER, gl.LINEAR);

			textureLoaded = true;
		};

		image.src = '/assets/background.jpg';
	}

	function handleResize() {
		if (!canvas || !gl) return;

		// Size to container if provided, otherwise parent element
		const target = container || canvas.parentElement;
		if (!target) return;

		const rect = target.getBoundingClientRect();
		canvas.width = rect.width;
		canvas.height = rect.height;
		gl.viewport(0, 0, canvas.width, canvas.height);

		if (uResolutionLoc) {
			gl.uniform2f(uResolutionLoc, canvas.width, canvas.height);
		}
	}

	function handleMouseMove(e) {
		if (!canvas) return;

		// Get canvas bounds for relative positioning
		const rect = canvas.getBoundingClientRect();
		const centerX = rect.left + rect.width / 2;
		const centerY = rect.top + rect.height / 2;

		// Calculate direction from center to mouse
		const dx = e.clientX - centerX;
		const dy = e.clientY - centerY;

		// Normalize to get unit direction vector (constant speed)
		const length = Math.sqrt(dx * dx + dy * dy);

		if (length > 1) {
			panDirection.x = dx / length;
			panDirection.y = dy / length;
		}
	}

	function handleTouchMove(e) {
		if (e.touches.length > 0) {
			const touch = e.touches[0];
			handleMouseMove({ clientX: touch.clientX, clientY: touch.clientY });
		}
	}

	function render(timestamp) {
		if (!textureLoaded || !gl) {
			animationId = requestAnimationFrame(render);
			return;
		}

		const time = (timestamp - startTime) / 1000;

		// Always pan at constant speed in current direction
		panOffset.x += panDirection.x * PAN_SPEED;
		panOffset.y += panDirection.y * PAN_SPEED;

		// Keep offset bounded to prevent floating point precision issues
		// Texture mirrors every 2 units, so wrap at 4 for seamless looping
		// Handle negative values: ((x % n) + n) % n
		panOffset.x = ((panOffset.x % 4) + 4) % 4;
		panOffset.y = ((panOffset.y % 4) + 4) % 4;

		// Update uniforms
		gl.uniform1f(uTimeLoc, time);
		gl.uniform2f(uOffsetLoc, panOffset.x, panOffset.y);

		// Draw
		gl.drawArrays(gl.TRIANGLES, 0, 6);

		animationId = requestAnimationFrame(render);
	}

	onMount(async () => {
		mounted = true;

		// Wait for DOM update so canvas is available
		await tick();

		if (!initWebGL()) {
			console.error('Failed to initialize WebGL');
			return;
		}

		loadTexture();
		handleResize();

		startTime = performance.now();
		animationId = requestAnimationFrame(render);

		window.addEventListener('resize', handleResize);
		window.addEventListener('mousemove', handleMouseMove);
		window.addEventListener('touchmove', handleTouchMove, { passive: true });
	});

	onDestroy(() => {
		if (!browser) return;

		if (animationId) {
			cancelAnimationFrame(animationId);
		}
		window.removeEventListener('resize', handleResize);
		window.removeEventListener('mousemove', handleMouseMove);
		window.removeEventListener('touchmove', handleTouchMove);
	});
</script>

{#if mounted}
	<canvas bind:this={canvas} class="background-shader"></canvas>
{/if}

<style>
	.background-shader {
		position: absolute;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		z-index: 0;
		pointer-events: none;
		border-radius: inherit;
	}
</style>
