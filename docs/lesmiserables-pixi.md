---
theme: dashboard
title: Les Misérables Network (PIXI)
toc: false
---

```js
import * as reorder from "npm:reorder.js@2.2.6";
import * as PIXI from 'https://cdn.jsdelivr.net/npm/pixi.js@7.4.0/+esm'
```

# Les Misérables

This adjacency matrix user Reorder.js and PIXI.js.

```js
const graph = FileAttachment("./data/miserables_numbers.json").json();
```

```js
const links = graph.links.map((d) => Object.create(d));
const nodes = graph.nodes.map((d) => Object.create(d));
```

```js
const orderingsOptions = ({
  none: "none",
  name: "by Name",
  count: "by Frequency",
  group: "by Group (cluster)",
  leafOrder: "by Leaf Order",
  leafOrderDist: "by Leaf Order over Distance Matrix",
  barycenter: "by Crossing Reduction",
  rcm: "by Bandwidth Reduction (RCM)",
  spectral: "Spectral",
  nn2opt: "NN-2OPT",
});
```

```js
const distanceOptions = ({
  euclidean: "Euclidean (L2)",
  manhattan: "Manhattan (L1)",
  chebyshev: "Chebyshev",
  hamming: "Hamming",
  jaccard: "Jaccard",
  braycurtis: "Braycurtis",
  morans: "Morans",
});
```

```js
let orders = ({ nodes, links }, { distance = "manhattan" }) => {
  const n = nodes.length;

  const graph = reorder.graph().nodes(nodes).links(links).init();

  let dist_adjacency;

  const leafOrder = reorder
    .optimal_leaf_order()
    .distance(reorder.distance[distance]);

  const adjacency = reorder.graph2mat(graph);
  
  if (distance == 'morans') {
    leafOrder.distance(reorder.distance[distance](adjacency));
  }

  function computeLeaforder() {
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

  function computeNN2OPT() {
    return leafOrder(adjacency);

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
    spectral: computeSpectral,
    nn2opt: computeNN2OPT,
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
const app = new PIXI.Application({background: '#FFFFFF',
  antialias: true,
  transparent: true,
  width: width,
  height: height,
  view: document.getElementById("matrix")
});
```

```js
const container = new PIXI.Container();
app.stage.addChild(container);
let squares = [];
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

  const labels = graph.nodes.map((d) => d.name);

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
    .attr('width', x.bandwidth() - 1)  
    .attr('height', x.bandwidth() - 1)
    .attr('net_s', ([s, t, v]) => s)
    .attr('net_t', ([s, t, v]) => t)
    .attr('color', function ([s, t, v]) {
      const color =  nodes[s].group === nodes[t].group ? d3.color(c(nodes[t].group)) : d3.color("black");
      return color.toString();
      })
    .attr('alpha', ([, , v]) => z(v)); 

    createVisualElements();
    draw(nodeIds);
}
```
```js
function createVisualElements(nodeIds){
  let custom = d3.select(customBase);
  var elements = custom.selectAll('custom.rect');
  
  elements.each(function(d,i) { // For each virtual/custom element...
    var node = d3.select(this);
    const sq = PIXI.Sprite.from(PIXI.Texture.WHITE);
    sq.tint = node.attr('color');
    sq.alpha = node.attr('alpha');
    sq.anchor.set(0,0);
    sq.width = node.attr('width');
    sq.height = node.attr('height');
    sq.eventMode = 'dynamic';
    sq.cursor = 'pointer';
    sq.on('pointerdown', (event) => { console.log(nodes[node.attr('net_s')].name,nodes[node.attr('net_t')].name); })
    
    squares.push(sq);
    container.addChild(sq);
  });
  console.log(squares);
}
```
```js
function draw(permutation){
  
  x.domain(permutation);
  
  let custom = d3.select(customBase);
  var elements = custom.selectAll('custom.rect');
  
  elements.each(function(d,i) { // For each virtual/custom element...
    var node = d3.select(this);
    //console.log(d,i,node);
    const sq = squares[i];
    sq.x = x(d[0])
    sq.y = x(d[1]) 
  });

  return permutation;
}
```
```js
sorting
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
const sorting = view(Inputs.select(Object.keys(orderingsOptions), {
  label: "Ordering method",
  format: (d) => orderingsOptions[d]
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
```js
const a = d3.range(nodes.length).sort((a, b) => d3.ascending(nodes[a].name, nodes[b].name))
```
```js
a.map((i) => nodes[i].name)
```

<canvas id="matrix">
</canvas>