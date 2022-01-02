<script>
    import Toggle from "./Toggle.svelte";

    export let settings;

    // let colours = ['cyan', 'green', 'purple'];
    let colours = ['#d137fc', 'cyan'];

</script>

<div class="outer">
    {#each settings.toggles as toggle, index}
        <Toggle bind:options={toggle} colour={colours[index]}></Toggle>
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
    }
</style>
