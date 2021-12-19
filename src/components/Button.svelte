<script>
    export let name;
    export let active;
</script>

<div class="btn" on:click class:active>
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
        color: cyan;
        font-weight: 600;
    }
</style>