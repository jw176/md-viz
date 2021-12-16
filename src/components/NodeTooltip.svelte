<script>
    export let node;
    export let container_rect;
    export let colour;
    import { fade } from 'svelte/transition';

    let x, y;

    function clamp(min, val, max){
        return Math.max(min, Math.min(max, val));
    }

    $: x = clamp(0, node.x - 100, container_rect.width - 200) + container_rect.left;
    $: y = clamp(0, node.y + 25, container_rect.height - 120) + container_rect.top;

</script>

<div class="node-tooltip" style="top:{y}px; left:{x}px; border-color:{colour}">
    <h3>{node.name}</h3>
    <div class="sub-content" transition:fade>
        {#each node.content as sub_content}
            <p>
                {sub_content}
            </p>
        {/each}
    </div>
</div>

<style>
    .node-tooltip {
        position: absolute;
        border: 2px solid white;
        color: white;
        border-radius: 10px;
        background: rgba(24, 28, 29, 0.95);
        padding: 18px;
        margin: 10px;
        min-height: 70px;
        min-width: 150px;
        display: flex;
        flex-direction: column;
        justify-content: flex-start;
        align-items: center;
        box-shadow: 0px 0px 30px 0px rgba(255, 255, 255, 0.1);
        pointer-events: none;
    }

    /* .sub-content {
        animation: expand-tooltip 2s linear 2s 1 forwards;
    }

    @keyframes expand-tooltip {
        0% {width: 0; height: 0; color: transparent;}
        20% {width: auto; height: auto; color: transparent;}
        100% {color: white;}
    } */
</style>