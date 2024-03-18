<script lang="ts">
	import { onMount } from 'svelte';
	import type { Tile as TileType } from '../types';
	import Tile from '../components/Tile.svelte';
	import { dndzone } from 'svelte-dnd-action';
	import { flip } from 'svelte/animate';
	import { fade } from 'svelte/transition';

	interface RoundProps {
		tiles: Array<TileType>;
		round_complete?: boolean;
		roundNo: number;
		onDone: () => void;
	}

	const flipDurationMs = 300;

	let { tiles, round_complete, roundNo, onDone } = $props<RoundProps>();

	let name = $derived(tiles.at(0)?.round_name);
	let current_order_guess = $state<Array<TileType> | null>(null);

	onMount(() => {
		current_order_guess = tiles.sort((t) => Math.random() - 0.5);
	});

	let correct_order = $derived(
		tiles.sort((a, b) => (a.air_pollution_index > b.air_pollution_index ? 1 : -1))
	);

	let correct_count = $derived(
		current_order_guess?.filter((t, index) => t.id === correct_order.at(index)?.id).length
	);

	function handleDndConsider(e: any) {
		current_order_guess = e.detail.items;
	}

	function handleDndFinalize(e: any) {
		current_order_guess = e.detail.items;
	}

	function submit() {
		round_complete = true;
		onDone();
	}
</script>

{#if current_order_guess}
	<div class="round" in:fade={{ delay: 2000, duration: 1000 }}>
		<h1>Round {roundNo} : {name}</h1>
		<div
			use:dndzone={{ items: current_order_guess, flipDurationMs }}
			class="tiles"
			on:consider={handleDndConsider}
			on:finalize={handleDndFinalize}
		>
			{#each current_order_guess as tile, index (tile.id)}
				<div animate:flip={{ duration: flipDurationMs }}>
					<Tile
						id={tile.id}
						isCorrect={tile.id === correct_order.at(index).id}
						showPrediction={round_complete}
						file={tile.filename.replace('.tif', '.png')}
						prediction={tile.air_pollution_index}
					/>
				</div>
			{/each}
		</div>
		{#if !round_complete}
			<button on:click={submit}>This is my guess</button>
		{:else if correct_count === current_order_guess.length}
			<p>Wow! you got them all right</p>
		{:else}
			<p>Unfortunately you only got {correct_count} out of {current_order_guess.length}</p>
		{/if}
	</div>
{:else}
	<div>Loading...</div>
{/if}

<style>
	div.tiles {
		display: flex;
		flex-direction: row;
		column-gap: 20px;
		margin: auto;
		justify-content: center;
	}
	button {
		margin: 20px;
		background-color: #e1341e;
		color: white;
		font-weight: bold;
		border-radius: 1.2rem;
		padding: 10px 20px;
		box-sizing: border-box;
		border: none;
	}
</style>
