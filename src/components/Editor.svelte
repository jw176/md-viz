<script>
    import { EditorState } from "prosemirror-state";
    import { EditorView } from "prosemirror-view";
    import { Schema, DOMParser } from "prosemirror-model";
    import { schema } from "prosemirror-schema-basic";
    import { addListNodes } from "prosemirror-schema-list";
    import { exampleSetup } from "prosemirror-example-setup";
    import { onMount } from "svelte";
    import { nodes_store, links_store } from "../stores";

    let editor, content;

    // Mix the nodes from prosemirror-schema-list into the basic schema to
    // create a schema with list support.

    function parser(doc) {
        const content = doc.content.content; // an array of objects where each object represents a node eg, heading, paragraph

        // const headings = ["H1", "H2", "H3", "H4", "H5", "H6"];
        let last_ids = [-1, -1, -1, -1, -1, -1];

        let nodes = [];
        let links = [];

        let id = 1;

        for (let i = 0; i < content.length; i++) {
            let type = content[i].type.name;

            if (type === "heading") {
                let level = content[i].attrs.level;
                let name = "";
                if (content[i].content.content[0]) {
                    name = content[i].content.content[0].text;
                }

                nodes.push({
                    id: id,
                    name: name,
                    size: level,
                    content: [],
                });

                if (level > 1) {
                    const last_id = Math.max(...last_ids.slice(0, level - 1));
                    if (last_id !== -1) {
                        links.push({
                            source: last_id,
                            target: id,
                        });
                    }
                }

                last_ids[level - 1] = id;

                id++;
            } else if (type === "paragraph") {
                if (nodes.length > 0) {
                    if (content[i].content.content[0]) {
                        const text = content[i].content.content[0].text;
                        nodes[nodes.length - 1].content.push(text);
                    }
                }
            } else {
                // ignore for now?
            }
        }

        return {
            nodes: nodes,
            links: links,
        };
    }

    async function update_node_tree(obj, is_transaction) {
        if (is_transaction) {
            if (obj.steps.length === 0) return;
        }
        const doc = is_transaction ? obj.doc : obj;
        let tree_data = parser(doc);
        console.log(tree_data);
        // content_tree.set(tree_data);
        nodes_store.set(tree_data.nodes);
        links_store.set(tree_data.links);
    }

    onMount(async () => {
        const mySchema = new Schema({
            nodes: addListNodes(schema.spec.nodes, "paragraph block*", "block"),
            marks: schema.spec.marks,
        });

        const initial_doc = DOMParser.fromSchema(mySchema).parse(content);
        update_node_tree(initial_doc, false);

        window.view = new EditorView(editor, {
            state: EditorState.create({
                doc: initial_doc,
                plugins: exampleSetup({ schema: mySchema }),
            }),
            dispatchTransaction(transaction) {
                console.log(transaction);

                update_node_tree(transaction, true);

                let newState = view.state.apply(transaction);
                view.updateState(newState);
            },
        });
    });
</script>

<div class="editor" bind:this={editor} />

<div class="content" bind:this={content}>
    <h1>Markdown Visualiser</h1>

    <h2>Development</h2>
    This app was created with the following technologies: Svelte D3js ProseMirror
    <h3>Svelte</h3>
    <h3>D3js</h3>
    <h3>ProseMirror</h3>

    <h2>Usage</h2>
    <p>
        Simply edit the text in this editor and the document structure will be
        visualised on the right in the form of a node link diagram.
    </p>
    <p>
        You can interact with the visualisation by dragging nodes, and hovering
        over a node will show a tooltip with some information from that node.
    </p>
</div>

<style>
    .content {
        display: none;
    }
</style>
