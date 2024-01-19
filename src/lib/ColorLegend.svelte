<script>
    import { createEventDispatcher } from 'svelte';

    const dispatch = createEventDispatcher();
    export let colorScale;
    export let selectedColumns;

    function handleClick(column) {
        dispatch('legend-click', {column});
    }
</script>

<div class="legend">
    {#each selectedColumns as legend}    
        <div class="legend-item" on:click={() => handleClick(legend)}>
            <div class="dot" style="background-color: {colorScale(legend)};">&nbsp;</div>
            <span class="text" style="color: {colorScale(legend)};">{legend}</span>
        </div>
    {/each}
</div>
<style>
    .legend {
        display: flex;
        justify-content: center;
        align-items: center;
        gap: 28px;
        position: relative;
        margin-top: 12px;
    }

    .dot {
        width: 14px;
        height: 14px;
        border-radius: 50%;
        display: inline-block;
        position: relative;  
        top: 6px;      
    } 
    
    .text {
        font-size: 13px;
    }
</style>