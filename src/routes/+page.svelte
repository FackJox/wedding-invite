<script>
	import Poster from '../components/Poster.svelte';
</script>

<main><Poster /></main>

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
</style>
