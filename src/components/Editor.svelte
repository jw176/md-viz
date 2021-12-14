<script>
    import { EditorState } from "prosemirror-state";
    import { EditorView } from "prosemirror-view";
    import { Schema, DOMParser } from "prosemirror-model";
    import { schema } from "prosemirror-schema-basic";
    import { addListNodes } from "prosemirror-schema-list";
    import { exampleSetup } from "prosemirror-example-setup";
    import { onMount } from "svelte";

    let editor, content;

    // Mix the nodes from prosemirror-schema-list into the basic schema to
    // create a schema with list support.

    onMount(async () => {
        const mySchema = new Schema({
            nodes: addListNodes(schema.spec.nodes, "paragraph block*", "block"),
            marks: schema.spec.marks,
        });

        window.view = new EditorView(editor, {
            state: EditorState.create({
                doc: DOMParser.fromSchema(mySchema).parse(
                    content
                ),
                plugins: exampleSetup({ schema: mySchema }),
            }),
        });
    });
</script>

<div class="editor" bind:this={editor} />
<div class="content" bind:this={content}></div>

<style>
    
</style>
