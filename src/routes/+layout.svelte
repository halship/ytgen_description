<script lang="ts">
	import favicon from '$lib/assets/favicon.svg';

	let { children } = $props();
</script>

<svelte:head>
	<title>Youtube説明文ジェネレータ</title>
	<link rel="icon" href={favicon} />
</svelte:head>

<main class="container">
	{@render children()}
</main>

<style>
	.container {
		max-width: 800px;
		margin: 0 auto;
		padding: 20px;
	}
</style>
