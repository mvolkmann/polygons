<script>
  import {onMount} from 'svelte';

  const HEIGHT = 900;
  const POLYGONS = 500;
  const RADIANS_PER_DEGREE = Math.PI / 180;
  const SIZE_PERCENT = 25;
  const WIDTH = 1000;

  let angle = 0;
  let box;
  let center = {x: WIDTH / 2, y: HEIGHT / 2};
  let polygon = [];
  let polygons = [polygon];
  let svg;
  let x;
  let y;

  onMount(() => {
    svg.style.setProperty('--width', WIDTH);
    svg.style.setProperty('--height', HEIGHT);
    box = svg.getBoundingClientRect();

    for (let i = 0; i < POLYGONS; i++) {
      randomPolygon();
    }
  });

  function addPoint(x, y) {
    let point = svg.createSVGPoint();
    point.x = x;
    point.y = y;

    // Convert from screen coordinates to SVG coordinates.
    const ctm = svg.getCTM().inverse();
    point = point.matrixTransform(ctm);

    polygon.push(point);
  }

  function getPoints(polygon) {
    return polygon.map(({x, y}) => `${x},${y}`).join(' ');
  }

  function matrixMultiply(left, right) {
    return left.map(row =>
      row.map((v, index) => v * right[index]).reduce((acc, v) => acc + v)
    );
  }

  function newPolygon() {
    if (polygon.length) {
      polygon = [];
      polygons.push(polygon);
      polygons = polygons; // trigger reactivity
    }
  }

  function onClick(event) {
    addPoint(event.clientX - box.left, event.clientY - box.top);
    polygon = polygon; // trigger reactivity
    polygons = polygons; // trigger reactivity
  }

  const randomInt = max => Math.ceil(Math.random() * max);

  function randomPolygon() {
    let factor = (100 - SIZE_PERCENT) / 100;
    const dx = randomInt(box.width * factor);
    const dy = randomInt(box.height * factor);

    const points = 3; //2 + randomInt(3);

    factor = SIZE_PERCENT / 100;
    for (let i = 0; i < points; i++) {
      const x = randomInt(box.width * factor);
      const y = randomInt(box.height * factor);
      addPoint(dx + x, dy + y);
    }
    newPolygon();
  }

  function rotate(event) {
    const newAngle = event.target.value;
    const deltaAngle = newAngle - angle;
    const radians = deltaAngle * RADIANS_PER_DEGREE;
    const cos = Math.cos(radians);
    const sin = Math.sin(radians);
    const {x, y} = center;
    const leftMatrix = [
      [cos, -sin, x - x * cos + y * sin],
      [sin, cos, y - x * sin - y * cos],
      [0, 0, 1]
    ];
    for (let polygon of polygons) {
      for (const point of polygon) {
        const rightMatrix = [point.x, point.y, 1];
        const [x, y] = matrixMultiply(leftMatrix, rightMatrix);
        point.x = x;
        point.y = y;
      }
      polygon = polygon; // trigger reactivity
    }
    polygons = polygons; // trigger reactivity
    angle = newAngle;
  }
</script>

<main>
  <form on:submit|preventDefault>
    <button disabled={polygon.length === 0} on:click={newPolygon}>New Polygon</button>
    <input
      type="range"
      min="0"
      max="360"
      on:input={rotate}
      bind:value={angle} />
    <span>{angle}</span>
  </form>
  <svg
    viewBox={`0 0 ${WIDTH} ${HEIGHT}`}
    xmlns="http://www.w3.org/2000/svg"
    on:click={onClick}
    bind:this={svg}>
    {#each polygons as polygon, index}
      {#if index === polygons.length - 1}
        <polyline points={getPoints(polygon)} />
      {:else}
        <polygon points={getPoints(polygon)} />
      {/if}
      {#each polygon as point}
        <circle cx={point.x} cy={point.y} r={2} />
      {/each}
    {/each}
    <circle class="center" cx={center.x} cy={center.y} r={5} />
  </svg>
</main>

<style>
  .center {
    fill: green;
    stroke: green;
  }

  circle {
    fill: red;
    stroke: red;
  }

  polygon,
  polyline {
    fill: none;
    stroke: blue;
  }

  svg {
    height: var(--height, 0);
    width: var(--width, 0);

    background-color: white;
    margin-top: 1rem;
    outline: 1px solid red;
  }
</style>
