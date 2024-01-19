<script>
    export let name = "Button";
    export let selected = false;
</script>

<button 
    class:selected={selected}
    on:click
>
    {name}
</button>

<style>
    button {
        cursor: pointer;        
        padding: 12px 32px;
        font-size: 16px;
        border: 1px solid var(--label-gray);
        height: 32px;
        display: flex;
        justify-content: center;
        align-items: center;
    }

    button.selected {
        background-color: var(--label-gray);
        color: white;
    }
</style>