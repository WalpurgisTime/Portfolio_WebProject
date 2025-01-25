
<script>
	import Header from './Header.svelte';
	import '../app.css';

	let { children } = $props();

</script>

<div class="app">
	<Header />


	<main>
		<div class="centered-content">
			{@render children()}
		</div>
	</main>


	<footer>

		<!--p>
			visit <a href="https://svelte.dev/docs/kit">svelte.dev/docs/kit</a> to learn about Corentin Jouan
		</p-->

	</footer>

</div>

<style>

	.app {
		display: flex;
		justify-content: center;
		flex-direction: column;
		min-height: 100vh;
	}

	main {

		flex: 1;
		display: flex;
		flex-direction: column;
		padding: 1rem;
		width: 100%;
		max-width: 64rem;
		margin: 0 auto;
		box-sizing: border-box;
	}

  .centered-content {
      display: flex;
      justify-content: center; /* Centre horizontalement */
      align-items: center; /* Centre verticalement */
      width: 100%; /* Largeur du conteneur */
      height: 100%; /* Hauteur du conteneur */
      min-height: calc(100vh - 64px); /* Assure que le contenu occupe tout l’espace disponible (moins footer/header si besoin) */
  }

	footer {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		padding: 12px;
	}

	footer a {
		font-weight: bold;
	}

	@media (min-width: 480px) {
		footer {
			padding: 12px 0;
		}
	}
</style>
