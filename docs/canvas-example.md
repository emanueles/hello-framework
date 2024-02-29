---
theme: dashboard
title: Canvas Example
toc: false
---
```js
  const canvas = document.getElementById('dessin')   // l'élément

  // définir la taille du canvas
  const WIDTH = 10000
  const HEIGHT = 10000
  canvas.setAttribute('width', WIDTH)
  canvas.setAttribute('height', HEIGHT)

  const context = canvas.getContext('2d')   // le contexte
  
  for (let i=0; i < 1000; i++)
  {
      for (let j=0; j < 1000; j++)
      {
        context.fillStyle = 'white'
        if (j % 2 == 0)
          context.fillStyle = 'indianred'
          context.fillRect(2*j, 2*i, 2, 2);
      }
  }
  display(context.canvas);

```

<canvas id="dessin">
</canvas>
