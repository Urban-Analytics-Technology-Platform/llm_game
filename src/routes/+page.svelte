<script lang="ts">
	import { walk } from 'svelte/compiler';
	import { tiles } from '../../src/rounds';
	import Round from '../components/Round.svelte';
	import Tile from '../components/Tile.svelte';
	import ComputerRound from '../components/ComputerRound.svelte';

	//@ts-ignore
	let containerEl;
	let current_round = $state(0);

	const scrollToBottom = async (node: any) => {
		node.scroll({ top: node.scrollHeight, behavior: 'smooth' });
	};

	$effect(() => {
		current_round;
		console.log('scrolling');
		setTimeout(() => {
			//@ts-ignore
			scrollToBottom(containerEl);
		}, 1000);
	});

	function doneRound() {
		current_round += 1;
	}
	let score = $state(0);
</script>

<div id="container" bind:this={containerEl}>
	<h1>Are you smarter than a foundation model!</h1>

	<p>Put the tiles in the order of increasing air pollution</p>
	{#each [0, 1, 2] as round}
		{#if round <= current_round}
			<Round
				roundNo={round + 1}
				tiles={tiles.filter((t) => t.round_number === round)}
				onDone={doneRound}
			/>
		{/if}
		{#if round < current_round}
			<ComputerRound
				onDone={() => {}}
				roundNo={round + 1}
				round_complete={true}
				tiles={tiles.filter((t) => t.round_number === round)}
			/>
		{/if}
	{/each}
</div>

<style>
	#container {
		width: 100vw;
		height: 100vh;
		background-image: linear-gradient(0deg, rgb(217, 175, 217) 0%, rgb(151, 217, 225) 100%);
		color: white;
		padding: 2rem;
		box-sizing: border-box;
		overflow-y: auto;
	}
</style>
