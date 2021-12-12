<script>
    import { onMount } from "svelte";
    import * as d3 from "d3-force";
    import import_data from "./data";

    const TEXT_COLOUR = "#53D8FB";
    const NODE_COLOUR = "#394648";
    const OUTLINE_COLOUR = NODE_COLOUR;
    const COLOURS = [
        "#4cc9f0",
        "#F72585",
        "#B0DB43",
        "#7E65CF",
        "#5AF2AE",
        "#F7F7F9",
    ];

    function get_colour(node) {
        return COLOURS[node.size - 1];
    }

    function getRadius(d) {
        return 30 - 4 * d.size;
    }

    function RandomInteger(min, max) {
        return Math.floor(Math.random() * (max - min)) + min;
    }

    let svg, width, height, graph;
    let data = import_data;
    let lastUpdateTime;

    onMount(async () => {
        let rect = svg.getBoundingClientRect();
        width = rect.width;
        height = rect.height;

        graph = d3
            .forceSimulation(data.nodes)
            .force(
                "link",
                d3.forceLink(data.links).id(function (d) {
                    return d.id;
                })
            )
            .force("charge", d3.forceManyBody().strength(-300))
            .force("center-x", d3.forceX().x(width / 2))
            .force("center-y", d3.forceY().y(height / 2))
            .force(
                "idk",
                d3.forceCollide().radius(function (d) {
                    return getRadius(d) + 2;
                })
            )
            .on("tick", () => {
                data = data;
            });
    });

    function dragstart(d) {
        if(graph) graph.alphaTarget(0.2).restart();
        d.fx = d.x;
        d.fy = d.y;

        lastUpdateTime = null;

        function mousemove(e) {
            drag(e, d);
        }

        addEventListener('mousemove', mousemove)
        addEventListener('mouseup', function mouseup(){dragend(d, mousemove)}, {once: true})
    }

    function drag(e, d) {
        d.fx = e.clientX;
        d.fy = e.clientY;
    }

    function dragend(d, func) {
        console.log(d);
        if (graph) {
            graph.alphaTarget(0.01)
        };
        d.fx = null;
        d.fy = null;

        lastUpdateTime = performance.now();

        removeEventListener('mousemove', func)
    }

    $: data && lastUpdateTime && ((performance.now() - lastUpdateTime) > 5000) && graph.stop();


</script>

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
        {#each data.nodes as node}
            <circle
                r={getRadius(node)}
                cx={node.x}
                cy={node.y}
                stroke-width="5"
                stroke={get_colour(node)}
                
                on:mousedown={() => {dragstart(node)}}

                on:mouseenter={() => {
                    // console.log(`node ${node.id} hovered over`);
                }}
                on:mouseleave={() => {
                    // console.log(`node ${node.id} exited`);
                }}
            />
        {/each}
    </g>
</svg>

<style>
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
