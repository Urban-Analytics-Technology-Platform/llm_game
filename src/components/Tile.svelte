<script lang="ts" context="module">
	import { fade } from 'svelte/transition';
	export interface TileProps {
		file: string;
		prediction: number;
		value: number;
        predictedRank: number; // 0 to 4, 0 = lowest polluation score
        actualRank: number; // as above
		showResult: boolean;
		showPrediction: boolean;
	}
</script>

<script lang="ts">
	let { file, prediction, value, predictedRank, actualRank, showResult, showPrediction } = $props<TileProps>();
    let isCorrect = $state(predictedRank === actualRank);
	console.log("predictedRank ", predictedRank, "actualRank ", actualRank);
</script>

<div class="tile">
	<img class="tile-img" src={`/images/${file}`} />
	{#if showResult}
		<div transition:fade={{ delay: 250, duration: 300 }} class="flex flex-column items-center">
			{#if isCorrect}
				<h2 class="correct">Correct</h2>
			{:else}
				<h2 class="wrong">Wrong</h2>
			{/if}
			{#if showPrediction}
				<p class="my-1"><b>AI's Prediction</b><br />{prediction.toFixed(3)} (#{predictedRank + 1})</p>
				<p class="my-1"><b>Actual Value</b><br />{value.toFixed(3)} (#{actualRank + 1})</p>
			{/if}
		</div>
	{/if}
</div>

<style>
	.tile {
		display: flex;
		flex-direction: column;
		align-items: center;
	}
	.tile-img {
		width: 200px;
		height: 200px;
	}
	.correct {
		color: green;
		font-weight: 700;
	}

	.wrong {
		color: red;
	}
</style>
