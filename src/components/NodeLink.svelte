<script>
    import { onMount } from "svelte";
    import * as force from "d3-force";
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

    onMount(async () => {
        let rect = svg.getBoundingClientRect();
        width = rect.width;
        height = rect.height;

        graph = force
            .forceSimulation(data.nodes)
            .force(
                "link",
                force.forceLink(data.links).id(function (d) {
                    return d.id;
                })
            )
            .force("charge", force.forceManyBody().strength(-300))
            // .force("center-x", d3.forceCenter(width / 2, height / 2))
            .force("center-x", force.forceX().x(width / 2))
            .force("center-y", force.forceY().y(height / 2))
            .force(
                "idk",
                force.forceCollide().radius(function (d) {
                    return getRadius(d) + 2;
                })
            )
            .on('tick', () => {data=data});
    });

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

    svg {
        width: 100%;
        height: 100%;
    }
</style>
