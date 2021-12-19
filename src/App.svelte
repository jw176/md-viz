<script>
	import NodeLink from "./components/NodeLink.svelte";
	import Editor from "./components/Editor.svelte";
</script>

<main>
	<div class="editor-wrapper">
		<Editor></Editor>
	</div>
	<div class="viz-wrapper">
		<NodeLink></NodeLink>
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

	.editor-wrapper {
		padding: 40px;
		background: rgb(24, 28, 29);
	}

	.viz-wrapper {
		overflow: hidden;
		position: relative;
	}

</style>