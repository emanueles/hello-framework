---
theme: dashboard
title: PIXI Example
toc: false
---
<canvas id="dessin">
</canvas>

```js
import * as PIXI from 'https://cdn.jsdelivr.net/npm/pixi.js@7.4.0/+esm'
```


```js
const width = 4000
const height = 2000
const app = new PIXI.Application({background: '#FFFFFF',
     antialias: true,
    transparent: true,
    resizeTo: window,
    view: document.getElementById("dessin")
    });

```

```js
const container = new PIXI.ParticleContainer(100000, {
    scale: true,
    position: true,
    rotation: true,
    uvs: true,
    alpha: true,
});
app.stage.addChild(container);

const squares = [];

for (let i=0; i < 1000; i++)
  {
    
      for (let j=0; j < 100; j++)
      { 
        const sq = PIXI.Sprite.from(PIXI.Texture.WHITE);
        sq.anchor.set(0.5);
        sq.scale.set(0.25)
        // scatter them all
        sq.x = i*4 + 4;
        sq.y = j*4 + 4;
        let tint = 'orange';
        if (j % 2 == 0) tint = 'steelblue'
        sq.tint = tint;
        squares.push(sq);
        container.addChild(sq);

      }
  }
```




