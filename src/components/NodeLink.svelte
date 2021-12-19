<script>
    import { onMount } from "svelte";
    import * as d3 from "d3-force";
    import import_data from "./data";
    import NodeTooltip from "./NodeTooltip.svelte";
    import VizToolbar from "./VizToolbar.svelte";
    import { nodes_store, links_store } from "../stores";

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
            name: "Tree",
            active: false,
            callback: function () {},
        },
        {
            name: "Radial",
            active: true,
            callback: function () {},
        },
    ];

    export const FORCE_BOUNDARY = true;

    function get_colour(node) {
        return COLOURS[node.size - 1];
    }

    function get_stroke_width(node) {
        // return 7 - node.size;
        return 5;
    }

    function getRadius(d) {
        return 30 - 4 * d.size;
    }

    function clamp(min, val, max) {
        return Math.max(min, Math.min(max, val));
    }

    let svg, width, height, graph;
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
                    .strength(0.5)
            )
            .force("charge", d3.forceManyBody().strength(-1000))
            .force("center-x", d3.forceX().x(width / 2))
            .force(
                "center-y",
                d3.forceY().y(function (d) {
                    if (!options[0].active) return (height / 2);
                    const val = Math.round((d.size / 6) * height);
                    return val;
                }).strength(options[0].active ? 2 : 0.2)
            )
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

        graph.restart();
    }

    onMount(async () => {
        let rect = svg.getBoundingClientRect();
        width = rect.width;
        height = rect.height;

        generate_graph();

        addEventListener("resize", () => {
            let rect = svg.getBoundingClientRect();
            width = rect.width;
            height = rect.height;

            generate_graph();
        });

        nodes_store.subscribe((val) => {
            data.nodes = val;
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
