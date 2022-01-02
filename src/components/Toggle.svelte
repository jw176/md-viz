<script>
    import Button from "./Button.svelte";

    export let colour;
    export let options;

    function select_option(name) {
        for (let i = 0; i < options.length; i++) {
            if (options[i].name === name) {
                options[i].active = true;
            } else {
                options[i].active = false;
            }
        }
    }
</script>

<div class="toggle-container">
    {#each options as option}
        <Button
            name={option.name}
            on:click={() => {
                select_option(option.name);
            }}
            bind:active={option.active}
            {colour}
        />
    {/each}
</div>

<style>
    .toggle-container {
        display: flex;
        flex-direction: row;
        justify-content: center;
        align-items: center;
        padding: 4px;
        border: 1px solid var(--lines);
        border-radius: 5px;
        margin: 0px 20px;
    }
</style>
