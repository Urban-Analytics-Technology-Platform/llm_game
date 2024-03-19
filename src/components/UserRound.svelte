<script lang="ts">
	import type { Tile as TileType } from '../types';
	import Tile from '../components/Tile.svelte';
	import { dndzone } from 'svelte-dnd-action';
	import { flip } from 'svelte/animate';
	import { fade } from 'svelte/transition';
	import { createEventDispatcher, onMount } from 'svelte';

	interface RoundProps {
		tiles: Array<TileType>;
		roundComplete?: boolean;
	}

	const dispatch = createEventDispatcher();
	const flipDurationMs = 300;

	let { tiles, roundComplete } = $props<RoundProps>();
	let submitted = $state(false);

	let current_order_guess = $state<Array<TileType> | null>(null);

	onMount(() => {
		submitted = false;
		current_order_guess = tiles.sort((t) => Math.random() - 0.5);
	});
	$effect(() => {submitted = false;}});

	let correct_order = $derived(
		[...tiles].sort((a, b) => (a.air_pollution_index > b.air_pollution_index ? 1 : -1))
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

    function getCorrectRank(tileId: string) {
        return correct_order.findIndex((t) => t.id === tileId);
    }

	function submit() {
		submitted = true;
		let i = 0;
		for (const tile of current_order_guess) {
			console.log("tile id", tile.id, "index", i, "correct rank", getCorrectRank(tile.id));
			i = i + 1;
		}
		dispatch('done', { score: correct_count });
	}

	console.log("correct_order ids ", correct_order.map((t) => t.id));


</script>

{#if current_order_guess}
	<div class="round" in:fade={{ delay: 1000, duration: 1000 }}>
		<h2>Your Guess</h2>
		<div
			use:dndzone={{ items: current_order_guess, flipDurationMs, dragDisabled: submitted }}
			class="tiles"
			on:consider={handleDndConsider}
			on:finalize={handleDndFinalize}
		>
			{#each current_order_guess as tile, index (tile.id)}
				<div animate:flip={{ duration: flipDurationMs }}>
					<Tile
						id={tile.id}
						isCorrect={tile.id === correct_order.at(index).id}
						showResult={roundComplete}
						showPrediction={false}
						file={tile.filename.replace('.tif', '.png')}
						prediction={tile.prediction}
						value={tile.air_pollution_index}
                        			predictedRank={index}
                       				actualRank={getCorrectRank(tile.id)}
					/>
				</div>
			{/each}
		</div>
		{#if !submitted}
			<button on:click={submit}>This is my guess</button>
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
