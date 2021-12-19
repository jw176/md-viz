<script>
    import Button from "./Button.svelte";

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

<div class="outer">
    {#each options as option}
        <Button
            name={option.name}
            on:click={() => {
                select_option(option.name);
                option.callback();
            }}
            bind:active={option.active}
        />
    {/each}
</div>

<style>
    .outer {
        position: absolute;
        top: 40px;
        right: 10%;
        display: flex;
        flex-direction: row;
        justify-content: center;
        align-items: center;
        padding: 4px;
        border: 1px solid var(--lines);
        border-radius: 5px;
    }
</style>
