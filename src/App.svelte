<script>
    import NodeLink from "./components/NodeLink.svelte";
    import Editor from "./components/Editor.svelte";
	import ExpandIcon from "./icons/ExpandIcon.svelte";

    let viz_expanded = false;
</script>

<main class:viz_expanded>
    <div class="editor-wrapper" class:hidden={viz_expanded}>
        <Editor />
    </div>
    <div class="viz-wrapper">
		<div class="icon" on:click={() => {viz_expanded = !viz_expanded;}}>
			<ExpandIcon></ExpandIcon>
		</div>
        <NodeLink bind:viz_expanded/>
    </div>
</main>

<style>
    main {
        width: 100%;
        height: 100%;
        margin: 0;
        padding: 0;
        display: grid;
        grid-template-columns: 1fr 1.2fr;
        grid-template-rows: 1fr;
        overflow: hidden;
    }

	.viz_expanded {
		display: block;
	}

    .hidden {
        display: none;
    }

	.icon {
		height: 40px;
		width: 40px;
		position: absolute;
		left: 40px;
		top: 40px;
		z-index: 100;
        cursor: pointer;
	}

    .editor-wrapper {
        padding: 40px;
        background: rgb(24, 28, 29);
    }

    .viz-wrapper,
    .editor-wrapper {
        height: 100%;
        overflow: hidden;
        position: relative;
    }
</style>
