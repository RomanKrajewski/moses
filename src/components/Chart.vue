<template>
    <div>
        <div id="chartComponent">
        </div>
    </div>
</template>

<script>
    import * as d3 from "d3"
    import {bboxCollide} from "d3-bboxCollide"
    import data from '../assets/data.json'

    export default {
        name: 'Chart',
        props: {
            msg: String,
        },
        data() {
            return {
                height: 900,
                width: 1200
            }
        },
        methods: {
            createChart() {
                const svg = d3.select("#chartComponent")
                    .append("svg")
                    .attr("viewBox", [0, 0, this.width, this.height]);
                const sampleBBox = svg.append("text").text("sample").node().getBBox()

                const links = data.links.map(d => Object.create(d));
                const nodes = data.nodes.map(d => Object.assign( {bbox: sampleBBox}, d));

                const collide = bboxCollide(function (d) {
                    return [[-d.bbox.width/2, -d.bbox.height/2],[d.bbox.width/2, d.bbox.height/2]]
                })
                    .strength(0.2)
                    .iterations(2)

                const simulation = d3.forceSimulation(nodes)
                        .force("link", d3.forceLink(links).distance(5))
                        .force("charge", d3.forceManyBody().strength(-3))
                        .force("collision", collide)
                        .force("center", d3.forceCenter(this.width / 2, this.height / 2))


                const link = svg.append("g")
                    .attr("stroke", "#999")
                    .attr("stroke-opacity", 0.6)
                    .selectAll("line")
                    .data(links)
                    .join("line")
                    .attr("stroke-width", d => d.weight);

                const node = svg.append("g")
                    .selectAll("g")
                    .data(nodes)
                    .join("g")
                    .each(function(d) {
                        const text = d3.select(this).append("svg:text").text(d.name).attr("fill", "black").attr("text-anchor", "middle")
                        const bbox = text.node().getBBox()
                        d3.select(this).append("rect").attr("fill", "wheat").attr("width", bbox.width).attr("height", bbox.height).attr("x", bbox.x).attr("y", bbox.y)
                        text.raise()
                    })
                // .attr("fill", color)
                // .call(drag(simulation));

                simulation.on("tick", () => {
                    link
                        .attr("x1", d => d.source.x)
                        .attr("y1", d => d.source.y)
                        .attr("x2", d => d.target.x)
                        .attr("y2", d => d.target.y);

                    node
                        .attr("transform", (d) => `translate(${Math.max(20, Math.min(this.width - 20, d.x))}, ${Math.max(20, Math.min(this.height - 20, d.y))})`)

                });


            }

        },
        mounted() {
            this.createChart();
        }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    h3 {
        margin: 40px 0 0;
    }

    ul {
        list-style-type: none;
        padding: 0;
    }

    li {
        display: inline-block;
        margin: 0 10px;
    }

    a {
        color: #42b983;
    }
</style>
