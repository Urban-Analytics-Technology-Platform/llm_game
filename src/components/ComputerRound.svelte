<script lang="ts">
	import type { Tile as TileType } from '../types';
	import Tile from '../components/Tile.svelte';
	import { flip } from 'svelte/animate';
	import { fade } from 'svelte/transition';

	interface RoundProps {
		tiles: Array<TileType>;
		round_complete?: boolean;
		roundNo: number;
		onDone: () => void;
	}

	let { tiles, round_complete, roundNo, onDone } = $props<RoundProps>();

	let current_order_guess = $state(tiles.sort((t) => Math.random() - 0.5));
	let computer_guess = $state(tiles.sort((a, b) => (a.prediction > b.prediction ? 1 : -1)));

	let correct_order = $derived(
		tiles.sort((a, b) => (a.air_pollution_index > b.air_pollution_index ? 1 : -1))
	);

	let correct_count = $derived(
		computer_guess.filter((t, index) => t.id === correct_order.at(index)?.id).length
	);
</script>

<div class="round" in:fade={{ delay: 1000, duration: 1000 }}>
	<h1>Computer's guess</h1>
	<div class="tiles">
		{#each computer_guess as tile, index (tile.id)}
			<div>
				<Tile
					isCorrect={tile.id === correct_order.at(index).id}
					showPrediction={round_complete}
					file={tile.filename.replace('.tif', '.png')}
					prediction={tile.prediction}
				/>
			</div>
		{/each}
	</div>
	{#if round_complete}
		<p>Computer got {correct_count} out of {current_order_guess.length}</p>
	{/if}
</div>

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
