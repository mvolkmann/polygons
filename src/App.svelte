<script>
  let polygon = [];
  let polygons = [];
  let x;
  let y;

  function savePolygon() {
    if (polygon.length) polygons.push(polygon);
    polygon = [];
  }
</script>

<main>
  <form on:submit|preventDefault>
    <button disabled={polygon.length === 0} on:click={savePolygon}>Save Polygon</button>
  </form>
  sv
</main>

<style>
  main {
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }

  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 4em;
    font-weight: 100;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>
