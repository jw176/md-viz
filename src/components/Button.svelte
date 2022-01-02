<script>
    export let name;
    export let active;
    export let colour = "cyan";
</script>

<div class="btn" on:click class:active style="{active ? `color: ${colour}` : ''}">
    {name}
</div>

<style>
    .btn {
        color: var(--text);
        padding: 4px;
        margin: auto 5px;
        cursor: pointer;
    }

    .active {
        font-weight: 600;
    }
</style>