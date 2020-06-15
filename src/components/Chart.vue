<template>
    <div>
        <div id="chartComponent">
        </div>
    </div>
</template>

<script>
    import * as d3 from "d3"
    export default {
        name: 'Chart',
        props: {
            msg: String,
        },
        data() {
            return {
                height: 900,
                width: 1600,
                maxLinkCount: Math.max(...data.nodes.map(d => this.getLinkCount(d.id))),
                maxWeight: Math.max(...data.links.map(d => d.weight)),
                maxScaleFactor: 4000,
                nodes: Array,
                renderedNodes: Object,
                simulation: Object,
                pimmelpommel: Object
            }
        },
        methods: {
            drag(simulation) {
                function dragstarted(d) {
                    if (!d3.event.active) simulation.alphaTarget(0.2).restart();
                    d.fx = d.x;
                    d.fy = d.y;
                }

                function dragged(d) {
                    d.fx = d3.event.x;
                    d.fy = d3.event.y;
                }

                function dragended(d) {
                    if (!d3.event.active) simulation.alphaTarget(0);
                    d.fx = null;
                    d.fy = null;
                }

                return d3.drag()
                    .on("start", dragstarted)
                    .on("drag", dragged)
                    .on("end", dragended);
            },
            getLinkCount(nodeId) {
                return data.links.filter(link => {
                    return link.source === nodeId || link.target === nodeId
                }).length
            },
            highlightNodes(searchString){
                this.nodes.forEach((node) => {
                    if(node.name.includes(searchString)){
                        node.highlighted = true
                    }else{
                        node.highlighted = false
                    }
                })
                this.pimmelpommel = this.renderedNodes.selectAll("g")
                    .data(this.nodes)
                    .join(enter=>
                            enter.append("g")
                            .each(function (d) {
                                const text = d3.select(this).append("svg:text")
                                    .text(d.name)
                                    .attr("fill", "black")
                                    .attr("font-size", `${d.bbox.scaleFactor}em`)
                                    .attr("text-anchor", "middle")
                                    .attr("text-rendering", "optimizeSpeed")
                                const bbox = text.node().getBBox()
                                d3.select(this).append("rect")
                                    .attr("fill", "wheat")
                                    .attr("fill-opacity", 0.8)
                                    .attr("width", bbox.width)
                                    .attr("height", bbox.height)
                                    .attr("x", bbox.x)
                                    .attr("y", bbox.y)
                                    .attr("shape-rendering", "optimizeSpeed")
                                text.raise()
                            })
                            .call(this.drag(this.simulation)),
                        update => update
                            .each(function (d) {
                                d3.select(this).select("rect").attr("fill", d.highlighted?"red":"wheat")
                            }))
            },
            getTextBBox(textElement, node) {
                const areaScaleFactor = this.maxScaleFactor * (this.getLinkCount(node.id) / this.maxLinkCount)
                const baseBBox = textElement.attr("font-size", "1em").text(node.name).node().getBBox()
                const baseArea = baseBBox.width * baseBBox.height
                const linearScaleFactor = Math.sqrt(areaScaleFactor) / Math.sqrt(baseArea)
                return {
                    width: baseBBox.width * linearScaleFactor,
                    height: baseBBox.height * linearScaleFactor,
                    scaleFactor: linearScaleFactor
                }
            },
            createChart() {
                const svg = d3.select("#chartComponent")
                    .append("svg")
                    .attr("viewBox", [0, 0, this.width, this.height]);

                const sampleBBox = svg.append("text")
                const links = data.links.map(d => Object.create(d));
                this.nodes = data.nodes.map(d => Object.assign({highlighted:true, bbox: this.getTextBBox(sampleBBox, d)}, d));
                sampleBBox.remove()

                const collide = bboxCollide((d) => [[-d.bbox.width / 2, -d.bbox.height / 2], [d.bbox.width / 2, d.bbox.height / 2]])
                    .strength(0.1)
                    .iterations(1)

                this.simulation = d3.forceSimulation(this.nodes)
                    .force("link", d3.forceLink(links).distance((d) => 30 - (Math.log(d.weight) / Math.log(this.maxWeight)) * 40))
                    .force("charge", d3.forceManyBody().strength(-60).distanceMax(100))
                    .force("collision", collide)
                    .force("center", d3.forceCenter(this.width / 2, this.height / 2))
                    .force("box", () => this.nodes.forEach((d) => {
                            d.x = Math.max(d.bbox.width / 2, Math.min(this.width - d.bbox.width / 2, d.x))
                            d.y = Math.max(d.bbox.height / 2, Math.min(this.height - d.bbox.height / 2, d.y));
                        })
                    )

                const zoomGroup = svg.append("g")
                // .attr("shape-rendering", "optimizeSpeed")

                svg.call(d3.zoom()
                    .extent([[0, 0], [this.width, this.height]])
                    .scaleExtent([1, 3])
                    .on("zoom", () => zoomGroup.attr("transform", d3.event.transform)));

                const link = zoomGroup.append("g")
                    .attr("stroke", "#999")
                    .attr("stroke-opacity", 0.6)
                    .selectAll("line")
                    .data(links)
                    .join("line")
                    .attr("stroke-width", d => d.weight)
                    .attr("shape-rendering", "optimizeSpeed")

                this.renderedNodes = zoomGroup.append("g")

                this.highlightNodes("Aar")

                this.simulation.on("tick", () => {
                    link
                        .attr("x1", d => d.source.x)
                        .attr("y1", d => d.source.y)
                        .attr("x2", d => d.target.x)
                        .attr("y2", d => d.target.y);

                    this.pimmelpommel
                        .attr("transform", (d) => `translate(${d.x},${d.y})`)

                });

            }

        },
        // console.log(this.weightMedian)
        mounted() {
            // console.log(this.weightAverage)
            this.createChart();
            this.highlightNodes("Aar")
        }
    }
    import {bboxCollide} from "d3-bboxCollide"

    import data from '../assets/data.json'
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    @import url('https://fonts.googleapis.com/css?family=Lato|Open+Sans|Oswald|Raleway|Roboto|Indie+Flower|Gamja+Flower');
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

    text {
        will-change: transform;
    }

    g {
        will-change: transform;
    }
</style>
