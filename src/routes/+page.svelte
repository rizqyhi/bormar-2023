<script lang="ts">
	import Map from '$lib/Map.svelte';
	import categories from '$lib/constants/categories';

	let currentRoute = '10k';
	$: currentRouteGpx = categories[currentRoute].gpxFilename;

	function changeCurrentRouteTo(route: string) {
		currentRoute = route;
	}
</script>

<svelte:head>
	<title>Bormar 2023 Routes</title>
</svelte:head>

<nav class="bg-slate-800">
	<div class="flex items-center md:px-16 px-4 h-16">
		<a href="/" class="text-white font-bold">Bormar 2023 Routes</a>
		<div class="ml-auto flex">
			<a
				href="#"
				class="ml-2 px-4 py-1 rounded-md text-white bg-red-500 bg-opacity-30"
				on:click={() => changeCurrentRouteTo('10k')}>10K</a
			>
			<a
				href="#"
				class="ml-2 px-4 py-1 rounded-md text-white bg-blue-500 bg-opacity-30"
				on:click={() => changeCurrentRouteTo('hm')}>Half Marathon</a
			>
			<a
				href="#"
				class="ml-2 px-4 py-1 rounded-md text-white bg-yellow-500 bg-opacity-30"
				on:click={() => changeCurrentRouteTo('fm')}>Full Marathon</a
			>
		</div>
	</div>
</nav>

<Map gpxFile={currentRouteGpx} />
