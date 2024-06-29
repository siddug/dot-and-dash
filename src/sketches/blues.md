
# Blues
Learning p5.js and understanding basics of curves and flow fields.


<script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.4.0/p5.js"></script>


<hr/>

## A grid of cells with an arrow in each of them

<br/>

<div id="sketch1"></div>

```js

let width = 600;
let height = 600;
let pid = 3;
let seed = 60;
let randomize = false;

function presetup(p) {
  // setup randomizer
  if (!randomize) {
    p.randomSeed(seed);
    p.noiseSeed(seed);
  }

  // setup pixel denisty
  p.pixelDensity(pid);

  // draw the base canvas
  p.createCanvas(width, height);
}



class Notebook {
  constructor(p) {
    this.p = p;
    this.grid_w = 15;
    this.grid_h = 15;
    this.padding = 0.2;
    this.cells = [];
    this.padding = 0.2;
    this.arrow = 0.1;
    this.rows = width / this.grid_w;
    this.cols = height / this.grid_h;
    this.show_grid = true;

    // handy fn to create more data
    this.createCells();
  }

  createCells() {
    for (var i = 0; i < this.rows; i++) {
      let row = [];
      for (var j = 0; j < this.cols; j++) {
        const x = i * this.grid_w;
        const y = j * this.grid_h;

        // base grid cell
        let cell = {
          x,
          y,
          width: this.grid_w,
          height: this.grid_h,
          padding: {
            x: 0.2 * this.grid_w,
            y: 0.2 * this.grid_h,
          },
          angle: this.p.random(0, 360),
          center: {
            x: x + this.grid_w / 2,
            y: y + this.grid_h / 2,
          },
        };

        // base flow vector
        cell.flow = {
          start: {
            x: x + cell.padding.x,
            y: y + cell.height / 2,
          },
          end: {
            x: x + this.grid_w - cell.padding.x,
            y: y + cell.height / 2,
          },
        };

        // arrow head
        cell.arrow = [
          {
            start: {
              x:
                cell.flow.end.x -
                this.p.cos(45) * this.arrow * this.p.min(this.grid_w, this.grid_h),
              y:
                cell.flow.end.y -
                this.p.cos(45) * this.arrow * this.p.min(this.grid_w, this.grid_h),
            },
            end: {
              x: cell.flow.end.x,
              y: cell.flow.end.y,
            },
          },
          {
            start: {
              x:
                cell.flow.end.x -
                this.p.cos(45) * this.arrow * this.p.min(this.grid_w, this.grid_h),
              y:
                cell.flow.end.y +
                this.p.cos(45) * this.arrow * this.p.min(this.grid_w, this.grid_h),
            },
            end: {
              x: cell.flow.end.x,
              y: cell.flow.end.y,
            },
          },
        ];


        cell.angle = 45;
        row.push(cell);
      }

      this.cells.push(row);
    }
  }

  onetime() {
    this.p.background("black");
    this.drawFlow();
  }

  everytime() {
    // no need for everytime unless interactivity or animation is needed
  }

  // Drawing a flow
  drawFlow() {
    for (var i = 0; i < this.rows; i++) {
      for (var j = 0; j < this.cols; j++) {
        if (this.show_grid) {
          this.drawCell(this.cells[i][j]);
        }
        this.drawArrow(this.cells[i][j]);
      }
    }
  }

  // Drawing a grid
  drawCell(cell) {
    this.p.stroke("#E7DFE8");
    this.p.fill("white");
    this.p.rect(cell.x, cell.y, cell.width, cell.height);
  }
  
  // Drawing an arrow
  drawArrow(cell) {
    this.p.push(); // Save the current drawing context
    this.p.translate(cell.center.x, cell.center.y); // Move to the center of the cell
    this.p.rotate(this.p.radians(cell.angle)); // Rotate by the given angle in degrees
    this.p.translate(-1 * cell.center.x, -1 * cell.center.y); // Move back from the center

    this.p.stroke("black");
    this.p.line(
      cell.flow.start.x,
      cell.flow.start.y,
      cell.flow.end.x,
      cell.flow.end.y
    );

    // draw an arrow head to convey the direction.
    this.p.line(
      cell.arrow[0].start.x,
      cell.arrow[0].start.y,
      cell.arrow[0].end.x,
      cell.arrow[0].end.y
    );
    this.p.line(
      cell.arrow[1].start.x,
      cell.arrow[1].start.y,
      cell.arrow[1].end.x,
      cell.arrow[1].end.y
    );

    this.p.pop();
  }

}

function getSketch(nbClass) {
  let nb;
  return (p) => {
    p.setup = () => {
      // mandatory
      presetup(p);

      nb = new nbClass(p);
      // notebook sepcific
      nb.onetime(p);
    };
    p.draw = () => {
      if (nb) {
          nb.everytime();
      }
    };
  }
}

new p5(getSketch(Notebook), 'sketch1');
```

<hr/>

## Curved arrows (my fn)

<br/>

<div id="sketch2"></div>

```js
class Notebook2 extends Notebook {
  constructor(p) {
    super(p);

    // changing angle of arrows.
    for (var i = 0; i < this.cells.length; i++) {
      for (var j = 0 ; j < this.cells[i].length; j++) {
        const cell = this.cells[i][j];
        cell.angle = 90 + (((180 * i) / this.rows) * j) / this.cols;
      }
    }
  }
}

new p5(getSketch(Notebook2), 'sketch2');
```

<hr/>

## Drawing curves (small step_length)

<br/>

<div id="sketch3"></div>

```js
class Notebook3 extends Notebook2 {
  constructor(p) {
    super(p);
    this.colors = ["#ffffff"];
    this.step_length = this.grid_w * 0.4;
  }

  drawRegularGrid() {
    // Equal grid
    const step_length_x = 10;
    const step_length_y = 10;

    for (var i = 0; i < width; i += step_length_x) {
      for (var j = 0; j < height; j += step_length_y) {
        this.drawCurve(i, j);
      }
    }
  }

hexToRgb(hex, alpha) {
    hex = hex.replace('#', '');

    var bigint = parseInt(hex, 16);

    var r = (bigint >> 16) & 255;
    var g = (bigint >> 8) & 255;
    var b = bigint & 255;

    return this.p.color(r, g, b, alpha);
  }

  drawCurve(sx, sy) {

    let completed = false;


    this.p.strokeWeight(1);
    
    this.p.noFill();

    this.p.beginShape();

    let x = sx;
    let y = sy;

    let visits = {};

    this.p.curveVertex(x, y);
    this.p.curveVertex(x, y);

    while (!completed) {
      if (x < 0 || x > width || y < 0 || y > height) {
        completed = true;
        break;
      }

      this.p.stroke(this.hexToRgb(this.colors[Math.floor(x) % this.colors.length],155 + this.p.random(100)));
      

      // get the current cell
      const row = Math.floor(x / this.grid_w);
      const col = Math.floor(y / this.grid_h);
      const cell = this.cells[row][col];

      if ((visits[row] || {})[col] || !cell) {
        completed = true;
        break;
      }

      const angle = cell.angle;

      x = x + this.p.cos(this.p.radians(angle)) * this.step_length;
      y = y + this.p.sin(this.p.radians(angle)) * this.step_length;

      visits[row] = visits[row] || {};
      visits[row][col] = true;

      this.p.curveVertex(x, y);
    }

    this.p.curveVertex(x, y);

    this.p.endShape();
  }

  drawFlow() {
    this.drawRegularGrid();
  }
}

new p5(getSketch(Notebook3), 'sketch3');
```

<hr/>

## Drawing curves (mid step length)

<br/>

<div id="sketch4"></div>

```js
class Notebook4 extends Notebook3 {
  constructor(p) {
    super(p);
    this.colors = ["#ffffff"];
    this.step_length = this.grid_w * 0.7;
  }
}

new p5(getSketch(Notebook4), 'sketch4');
```

<hr/>

## Drawing curves (colors)

<br/>

<div id="sketch5"></div>

```js
class Notebook5 extends Notebook4 {
  constructor(p) {
    super(p);
    this.colors = ["#caf0f8", "#ade8f4", "#48cae4", "#00b4d8", "#0096c7", "#0077b6", "#023e8a", "#03045e"];
    this.step_length = this.grid_w * 1.2;
  }
}

new p5(getSketch(Notebook5), 'sketch5');
```

<hr/>

## Field (randomly uniform grid)

<br/>

<div id="sketch6"></div>

```js
class Notebook6 extends Notebook5 {
  constructor(p) {
    super(p);
  }

  drawFlow() {
    this.drawUniformlyRandomGrid();
  }

  drawUniformlyRandomGrid() {
    const numElements = 10000; // Number of elements to draw

    for (let i = 0; i < numElements; i++) {
      const x1 = this.p.random(width);
      const y1 = this.p.random(height);

      this.drawCurve(x1, y1);
    }
  }
}

new p5(getSketch(Notebook6), 'sketch6');
```

<hr/>

## Adding Perlin Noise

<br/>

<div id="sketch7"></div>

```js
class Notebook7 extends Notebook6 {
  constructor(p) {
    super(p);

    // changing angle of arrows.
    for (var i = 0; i < this.cells.length; i++) {
      for (var j = 0 ; j < this.cells[i].length; j++) {
        const cell = this.cells[i][j];
         // perlin continuous
        this.p.noiseDetail(5, 0.6);
        cell.angle = this.p.map(this.p.noise(0.005*j, 0.005*i), 0, 1, 0, 720);
      }
    }

  }

}

new p5(getSketch(Notebook7), 'sketch7');
```

<hr/>

## Adding Perlin Noise (discontinuous)

<br/>

<div id="sketch8"></div>

```js
class Notebook8 extends Notebook7 {
  constructor(p) {
    super(p);

    // changing angle of arrows.
    for (var i = 0; i < this.cells.length; i++) {
      for (var j = 0 ; j < this.cells[i].length; j++) {
        const cell = this.cells[i][j];
         // perlin continuous
        this.p.noiseDetail(5, 0.6);
        cell.angle = Math.floor(this.p.map(this.p.noise(0.005*j, 0.005*i), 0, 1, 0, 720/30))*30;
      }
    }
  }

}

new p5(getSketch(Notebook8), 'sketch8');
```

<hr/>

## Random Noise

<br/>

<div id="sketch9"></div>

```js
class Notebook9 extends Notebook8 {
  constructor(p) {
    super(p);

    // changing angle of arrows.
    for (var i = 0; i < this.cells.length; i++) {
      for (var j = 0 ; j < this.cells[i].length; j++) {
        const cell = this.cells[i][j];
        cell.angle = this.p.random() * 260;
      }
    }
  }

}

new p5(getSketch(Notebook9), 'sketch9');
```