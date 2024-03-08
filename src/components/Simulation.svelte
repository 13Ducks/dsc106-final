<script>
    import Chart from "./Chart.svelte";
    import { createEventDispatcher } from "svelte";

    const dispatch = createEventDispatcher();
    let playCount = 0;

    function handleSimPlayed(data) {
        playCount = data.detail.playCount;
        if (playCount == 100) {
            dispatch("simDone");
        }
    }
</script>

<p>Games Played: {playCount}</p>
<div class="grid">
    {#each Array(16) as _, i}
        <div class="chart-container">
            <Chart key={i} on:simPlayed={handleSimPlayed} />
        </div>
    {/each}
</div>

<style>
    .grid {
        display: grid;
        grid-template-columns: repeat(4, auto);
    }

    .chart-container {
        width: 100%; /* Ensure each chart container takes the full width */
    }
</style>
