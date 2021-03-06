<script>
    import { onMount } from "svelte";
    import * as d3 from "d3-force";
    import import_data from "./data";
    import NodeTooltip from "./NodeTooltip.svelte";
    import VizToolbar from "./VizToolbar.svelte";
    import { nodes_store, links_store } from "../stores";
    import { get } from "svelte/store";

    let hover_node_index = null;

    export const COLOURS = [
        "#4cc9f0",
        "#F72585",
        "#B0DB43",
        "#7E65CF",
        "#5AF2AE",
        "#F7F7F9",
    ];

    let options = [
        {
            name: "Network",
            active: true,
            callback: function () {},
        },
        {
            name: "Tree",
            active: false,
            callback: function () {},
        },
        {
            name: "Radial",
            active: false,
            callback: function () {},
        },
    ];

    export const FORCE_BOUNDARY = true;
    export let viz_expanded;

    function get_colour(node) {
        return COLOURS[node.size - 1];
    }

    function get_stroke_width(node) {
        // return 7 - node.size;
        return 5;
    }

    function getRadius(d) {
        return 24 - 3 * d.size;
    }

    function clamp(min, val, max) {
        return Math.max(min, Math.min(max, val));
    }

    function get_current_option() {
        for (let i = 0; i < options.length; i++) {
            if (options[i].active) {
                return options[i].name;
            }
        }
    }

    let svg,
        width,
        height,
        graph,
        min_size = 1,
        max_size = 6;
    // let data = import_data;

    let elementIsBeingDragged = false;
    let data = { nodes: [], links: [] };

    function generate_graph() {
        graph = d3
            .forceSimulation(data.nodes)
            .force(
                "link",
                d3
                    .forceLink(data.links)
                    .id(function (d) {
                        return d.id;
                    })
                    .strength(0.2)
            )
            .force("charge", d3.forceManyBody().strength(-1000))
            .force(
                "force-collide",
                d3.forceCollide().radius(function (d) {
                    return getRadius(d) + get_stroke_width(d);
                })
            )
            .on("tick", () => {
                data = data;
                if (!FORCE_BOUNDARY) return;
                data.nodes.forEach((d) => {
                    const r = getRadius(d);
                    d.x = clamp(r, d.x, width - r);
                    d.y = clamp(r, d.y, height - r);
                });
            });

        const current_option = get_current_option().toLowerCase();
        if (current_option === "network") {
            graph = graph
                .force("center-x", d3.forceX().x(width / 2))
                .force("center-y", d3.forceY().y(height / 2));
        } else if (current_option === "tree") {
            graph = graph.force("center-x", d3.forceX().x(width / 2)).force(
                "center-y",
                d3
                    .forceY()
                    .y(function (d) {
                        const val = Math.round(
                            ((d.size - min_size + 1) /
                                (max_size - min_size + 2)) *
                                height
                        );
                        return val;
                    })
                    .strength(2)
            );
        } else {
            const min_dimension = Math.round(Math.min(height, width) / 2);
            graph = graph.force('radial', d3.forceRadial().x(width / 2).y(height / 2).radius(function (d) {
                const val = Math.round(
                            ((d.size - min_size + 1) /
                                (max_size - min_size + 2)) *
                                min_dimension
                        );
                return val;
            }).strength(2));
        }

        graph.restart();
    }

    function update_max_and_min_sizes(nodes) {
        let min = 6;
        let max = 1;
        for (let i = 0; i < nodes.length; i++) {
            if (nodes[i].size >= max) {
                max = nodes[i].size;
            }
            if (nodes[i].size <= min) {
                min = nodes[i].size;
            }
        }
        min_size = min;
        max_size = max;
    }

    function resize() {
        if (!svg) return;
        let rect = svg.getBoundingClientRect();
        width = rect.width;
        height = rect.height;

        generate_graph();
    }

    onMount(async () => {
        let rect = svg.getBoundingClientRect();
        width = rect.width;
        height = rect.height;

        generate_graph();

        addEventListener("resize", resize);

        nodes_store.subscribe((val) => {
            data.nodes = val;
            update_max_and_min_sizes(val);
            generate_graph();
        });
        links_store.subscribe((val) => {
            data.links = val;
            generate_graph();
        });
    });

    function dragstart(d) {
        if (graph) graph.alphaTarget(0.3).restart();
        d.fx = d.x;
        d.fy = d.y;

        elementIsBeingDragged = true;

        function mousemove(e) {
            drag(e, d);
        }

        addEventListener("mousemove", mousemove);
        addEventListener(
            "mouseup",
            function mouseup() {
                dragend(d, mousemove);
            },
            { once: true }
        );
    }

    function drag(e, d) {
        const rect = svg.getBoundingClientRect();
        d.fx = e.clientX - rect.left;
        d.fy = e.clientY - rect.top;
    }

    function dragend(d, func) {
        if (graph) {
            graph.alphaTarget(0);
        }

        delete d.fx;
        delete d.fy;

        elementIsBeingDragged = false;
        hover_node_index = null;

        removeEventListener("mousemove", func);
    }

    $: options & generate_graph();
    $: viz_expanded & resize();
</script>

<div class="svg-wrapper">
    <VizToolbar bind:options />

    <svg bind:this={svg}>
        <g class="links">
            {#each data.links as link}
                <line
                    x1={link.source.x}
                    x2={link.target.x}
                    y1={link.source.y}
                    y2={link.target.y}
                    stroke-width="1"
                />
            {/each}
        </g>
        <g class="nodes">
            {#each data.nodes as node, i}
                <circle
                    r={getRadius(node)}
                    cx={node.x}
                    cy={node.y}
                    stroke-width={get_stroke_width(node)}
                    stroke={get_colour(node)}
                    on:mousedown={() => {
                        dragstart(node);
                    }}
                    on:mouseenter={() => {
                        if (elementIsBeingDragged) return;
                        hover_node_index = i;
                    }}
                    on:mouseleave={() => {
                        if (elementIsBeingDragged) return;
                        hover_node_index = null;
                    }}
                />
            {/each}
        </g>
    </svg>
</div>

{#if hover_node_index !== null}
    <NodeTooltip
        node={data.nodes[hover_node_index]}
        container_rect={svg.getBoundingClientRect()}
        colour={get_colour(data.nodes[hover_node_index])}
    />
{/if}

<style>
    .svg-wrapper {
        height: 100%;
        width: 100%;
        position: relative;
        overflow: hidden;
    }

    .links {
        stroke: #394648;
    }

    .nodes {
        fill: #394648;
    }

    circle {
        cursor: grab;
    }

    svg {
        width: 100%;
        height: 100%;
        background: rgb(24, 28, 29);
    }
</style>
