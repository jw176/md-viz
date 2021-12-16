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

                last_ids[level - 1] = id;

                if (level > 1) {
                    for (let j = level - 2; j >= 0; j--) {
                        if (last_ids[j] !== -1) {
                            links.push({
                                source: last_ids[j],
                                target: id,
                            });
                            break;
                        }
                    }
                }

                id++;

            } else if (type === "paragraph") {
                if (nodes.length > 0){
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

    async function update_node_tree(transaction) {
        if (transaction.steps.length === 0) return;

        let tree_data = parser(transaction.doc);
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

        window.view = new EditorView(editor, {
            state: EditorState.create({
                doc: DOMParser.fromSchema(mySchema).parse(content),
                plugins: exampleSetup({ schema: mySchema }),
            }),
            dispatchTransaction(transaction) {
                console.log(transaction);

                update_node_tree(transaction);

                let newState = view.state.apply(transaction);
                view.updateState(newState);
            },
        });
    });
</script>

<div class="editor" bind:this={editor} />
<div class="content" bind:this={content} />

<style>
</style>
