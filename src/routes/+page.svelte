<script>
	import ThreeScene from '../lib/ThreeScene.svelte';
	import ThreeFromUnity from '../lib/ThreeFromUnity.svelte';
</script>

<svelte:head>
	<title>How to play Sverdle</title>
	<meta name="description" content="How to play Sverdle" />
</svelte:head>



<div class="container">
	<div class="component">
		<ThreeScene />
	</div>
	<div class="component">
		<ThreeFromUnity />
	</div>
</div>

<style>
    .container {
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    .component {
        margin-bottom: 20px; /* Adjust the spacing as needed */
    }
</style>