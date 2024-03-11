---
theme: dashboard
title: Les Misérables Network
toc: false
---

```js
// import {require} from "npm:d3-require";
// const reorder = require("reorder.js@2.2.6");
import * as reorder from "npm:reorder.js@2.2.6";

```

```js
const data = FileAttachment("./data/miserables.json").json();
```

```js
function buildNetwork(data, {width}) {

  const height = 640;
const color = d3.scaleOrdinal(d3.schemeCategory10);

// Copy the data to protect against mutation by d3.forceSimulation.
const links = data.links.map((d) => Object.create(d));
const nodes = data.nodes.map((d) => Object.create(d));

const simulation = d3.forceSimulation(nodes)
    .force("link", d3.forceLink(links).id((d) => d.id))
    .force("charge", d3.forceManyBody())
    .force("center", d3.forceCenter(width / 2, height / 2))
    .on("tick", ticked);

const svg = d3.create("svg")
    .attr("width", width)
    .attr("height", height)
    .attr("viewBox", [0, 0, width, height])
    .attr("style", "max-width: 100%; height: auto;");

const link = svg.append("g")
    .attr("stroke", "var(--theme-foreground-faint)")
    .attr("stroke-opacity", 0.6)
  .selectAll("line")
  .data(links)
  .join("line")
    .attr("stroke-width", (d) => Math.sqrt(d.value));

const node = svg.append("g")
    .attr("stroke", "var(--theme-background)")
    .attr("stroke-width", 1.5)
  .selectAll("circle")
  .data(nodes)
  .join("circle")
    .attr("r", 5)
    .attr("fill", (d) => color(d.group))
    .call(drag(simulation));

node.append("title")
    .text((d) => d.id);

function ticked() {
  link
      .attr("x1", (d) => d.source.x)
      .attr("y1", (d) => d.source.y)
      .attr("x2", (d) => d.target.x)
      .attr("y2", (d) => d.target.y);

  node
      .attr("cx", (d) => d.x)
      .attr("cy", (d) => d.y);
}

return svg.node();
}
```

```js
function drag(simulation) {

  function dragstarted(event) {
    if (!event.active) simulation.alphaTarget(0.3).restart();
    event.subject.fx = event.subject.x;
    event.subject.fy = event.subject.y;
  }

  function dragged(event) {
    event.subject.fx = event.x;
    event.subject.fy = event.y;
  }

  function dragended(event) {
    if (!event.active) simulation.alphaTarget(0);
    event.subject.fx = null;
    event.subject.fy = null;
  }

  return d3.drag()
      .on("start", dragstarted)
      .on("drag", dragged)
      .on("end", dragended);
}

```
# Les Misérables

This network is built using D3.

<div class="grid grid-cols-1">
  <div class="card">${resize((width) => buildNetwork(data, {width}))}</div>
</div>

This adjacency matrix user Reorder.js.

```js
const graph = FileAttachment("./data/miserables_numbers.json").json();
```

```js
const orderingsOptions = [
  "none",
  "name",
  "count",
  "group",
  "leafOrder",
  "leafOrderDist",
  "barycenter",
  "rcm",
  "spectral"
]
```

```js
const distanceOptions = ({
  euclidean: "Euclidean (L2)",
  manhattan: "Manhattan (L1)",
  minkowski: "Minkowski",
  chebyshev: "Chebyshev",
  hamming: "Hamming",
  jaccard: "Jaccard",
  braycurtis: "Braycurtis"
})
```


```js
let orders = ({ nodes, links }, { distance = "manhattan" }) => {
  const n = nodes.length;

  const graph = reorder.graph().nodes(nodes).links(links).init();

  let dist_adjacency;

  const leafOrder = reorder
    .optimal_leaf_order()
    .distance(reorder.distance[distance]);

  function computeLeaforder() {
    const adjacency = reorder.graph2mat(graph);
    return leafOrder(adjacency);
  }

  function computeLeaforderDist() {
    if (!dist_adjacency) dist_adjacency = reorder.graph2valuemats(graph);
    return reorder.valuemats_reorder(dist_adjacency, leafOrder);
  }

  function computeBarycenter() {
    const barycenter = reorder.barycenter_order(graph);
    return reorder.adjacent_exchange(graph, ...barycenter);
  }

  function computeRCM() {
    return reorder.reverse_cuthill_mckee_order(graph);
  }

  function computeSpectral() {
    return reorder.spectral_order(graph);
  }

  const orders = {
    none: () => d3.range(n),
    name: () =>
      d3.range(n).sort((a, b) => d3.ascending(nodes[a].name, nodes[b].name)),
    count: () => d3.range(n).sort((a, b) => nodes[b].count - nodes[a].count),
    group: () =>
      d3
        .range(n)
        .sort(
          (a, b) =>
            d3.ascending(nodes[a].group, nodes[b].group) ||
            d3.ascending(nodes[a].name, nodes[b].name)
        ),
    leafOrder: computeLeaforder,
    leafOrderDist: computeLeaforderDist,
    barycenter: computeBarycenter,
    rcm: computeRCM,
    spectral: computeSpectral
  };

  return orders;
}
```

```js
const permutations = orders(graph, {distance})
```

```js
let customBase = document.createElement('custom');
```

```js
  const margin = { top: 80, right: 0, bottom: 10, left: 80},
    width = 720,
    height = 720;
```

```js
const nodeIds = d3.range(graph.nodes.length);
const x = d3
      .scaleBand()
      .domain(nodeIds)
      .range([0, width]),
    z = d3
      .scaleLinear()
      .domain([0,4])
      .clamp(true),
    c = d3.scaleOrdinal(d3.range(10), d3.schemeCategory10);
```

```js
function setup() {

  const links = graph.links.map((d) => Object.create(d));
  const nodes = graph.nodes.map((d) => Object.create(d));

  const labels = graph.nodes.map((d) => d.name);

  const canvas = document.getElementById('matrix');
  canvas.setAttribute('width', width)
  canvas.setAttribute('height', height)

  const matrix = links
    .flatMap(({ source, target, value }) => [
      [source, target, value],
      [target, source, value]
    ])
    .concat(nodeIds.map(i => [i, i]));    

    let custom = d3.select(customBase);

    let join = custom.selectAll('custom.rect').data(matrix);
    
  let rects = join
    .enter()
    .append('custom')
    .attr('class', 'rect')  
    .attr('width', x.bandwidth() - 2)  
    .attr('height', x.bandwidth() - 2)
    .attr('fillStyle', function ([s, t, v]) {
      const color =  s.group === t.group ? d3.color(c(t.group)) : d3.color("white");
      color.opacity = z(v);
      return color.toString();
      });

    draw(nodeIds);

}
```
```js
function draw(permutation){
  
  x.domain(permutation);
  
  const canvas = document.getElementById('matrix');
  const context = canvas.getContext('2d');
  
  context.save();
  context.clearStyle = "000000";
  context.clearRect(0, 0, width, height); // Clear the canvas.
  
  let custom = d3.select(customBase);
  var elements = custom.selectAll('custom.rect');
  
  elements.each(function(d,i) { // For each virtual/custom element...
    var node = d3.select(this);
    console.log(d,i,node);
    context.fillStyle = node.attr("fillStyle");
    context.fillRect(x(d[0]), x(d[1]), node.attr('width'), node.attr('height'));  
  });

  return permutation;
}
```

```js
setup();
```

```js
const current = draw(permutations[sorting]())
```

```js
current
```

```js
const sorting = view(Inputs.select(orderingsOptions, {
  label: "Ordering method"
}));
```

```js
const distance = view(Inputs.select(Object.keys(distanceOptions), {
  label: "Distance (for leafOrders):",
  format: (d) => distanceOptions[d],
  disabled: !["leafOrder", "leafOrderDist"].includes(sorting),
  value: this?.value || "manhattan"
}));
```

<canvas id="matrix">
</canvas>


