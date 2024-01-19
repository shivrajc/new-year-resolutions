<script>
    import Button from "./Button.svelte";
    import { createEventDispatcher } from "svelte";

    const dispatch = createEventDispatcher();
    export let viewGender = true;

    function handleClick(selection) {
        dispatch('button-click', {viewGender: selection === "Gender" ? true : false})
    }
</script>

<div class="button-group">
    <Button 
        name={"Age Group"}
        selected={!viewGender} 
        on:click={() => handleClick("Age Group")}
    />
    <Button 
        name={"Gender"}
        selected={viewGender} 
        on:click={() => handleClick("Gender")}
    />
</div>

<style>
    .button-group {
        display: flex;
        justify-content: center;
        align-items: center;
        height: 60px;
        margin: 12px 0;
    }
</style>