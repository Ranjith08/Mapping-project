<script>
  // @ts-ignore
  import maplibre from "maplibre-gl";
  import "maplibre-gl/dist/maplibre-gl.css";
  import { onMount } from "svelte";
  import MarkerPopup from "./MarkerPopup.svelte";
  import { buffer, point, bbox, pointsWithinPolygon } from "@turf/turf";

  /**
   * The public MapTiler key used for the map style.
   * @type {string}
   */
  const publicMaptilerKey = "tOLajJc34bJcEMuZ3CQn";

  /**
   * The map object.
   * @type {maplibre.Map}
   */
  export let map;

  /**
   * The filter expression for the data.
   * @type {string}
   */
  let filterExpression;

  /**
   * The appearance expression for the data.
   * @type {string}
   */
  let appearanceExpression;

  /**
   * The map container element.
   * @type {HTMLElement}
   */
  let mapContainer;

  /**
   * The JSON, from the blob.
   * @type {Object}
   */
  let resourceJSON;

  /**
   * The main data, downloaded.
   * @type {Blob}
   */
  let resourceBlob;

  /**
   * The buffer polygon around the cursor, it is circular
   * @type {GeoJSON.Feature<GeoJSON.Polygon>}
   */
  let bufferPolygon;

  /**
   * The events within the radius of the cursor
   * @type {GeoJSON.FeatureCollection<GeoJSON.Point>}
   */
  export let eventsInHighlight;

  // /**
  //  * The square size (lookingglass in screen coords)
  //  * @type {number}
  //  */
  // let squareSize = 1;

  // /**
  //  * The map zoom level to allow dpeendant buffer size changes
  //  * @type {number}
  //  */
  // let mapZoomLevel = 3;

  /**
   * The cursor position.
   * @type {{ x: number; y: number }}
   */
  let cursorPosition = { x: 0, y: 0 };

  /**
   * Fetches the resource from the server and adds it to the map.
   * @returns {Promise<void>}
   */
  async function getResource() {
    return await fetch("/HWdb_geocoded.geojson").then(async (response) => {
      return response.json();
    });
  }

  /**
   * Adds the layer to the map.
   * @returns {void}
   */
  const addLayer = async () => {
    resourceBlob = await getResource();

    // Add source and layer to the map
    map.addSource("hwdb", {
      type: "geojson",
      data: resourceBlob,
    });

    map.addSource("buffer", {
      type: "geojson",
      data: "",
    });

    map.addLayer({
      id: "buffer",
      type: "line",
      source: "buffer",
      paint: {
        "line-color": "rgba(0, 0, 0, 1)",
        "line-opacity": 0.9,
        "line-width": 10,
      },
    });

    map.addLayer({
      id: "point",
      type: "circle",
      source: "hwdb",
      paint: {
        "circle-radius": 4,
        "circle-color": "gray",
        "circle-opacity": 0.5,
      },
    });

    map.on("click", "point", async (e) => {
      let properties = e.features[0].properties;
      const coordinates = e.features[0].geometry.coordinates.slice();

      // Ensure that if the map is zoomed out such that multiple
      // copies of the feature are visible, the popup appears
      // over the copy being pointed to.
      while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
        coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
      }
      let popup = new maplibre.Popup()
        .setLngLat(coordinates)
        .setHTML("")
        .addTo(map);
      let child = popup.getElement();

      let p = new MarkerPopup({
        target: child.children[1],
        anchor: null,
        props: {
          title: properties.title,
          excerpt: properties.excerpt,
          date: properties.date,
          link: properties.links,
          map: map,
          coordinates: coordinates,
          point: e.point,
        },
      });
    });

    // Change the cursor to a pointer when the mouse is over the places layer.
    map.on("mouseenter", "point", async () => {
      map.getCanvas().style.cursor = "pointer";
    });

    // Change it back to a pointer when it leaves.
    map.on("mouseleave", "point", async () => {
      map.getCanvas().style.cursor = "";
    });
  };

  /**
   * Event handler for map initialization and resource loading.
   * @returns {void}
   */
  onMount(async () => {
    // Map initialization
    map = new maplibre.Map({
      container: mapContainer,
      style: `https://api.maptiler.com/maps/5b0fdf12-ac62-4bd8-975b-50ac01e3abbd/style.json?key=${publicMaptilerKey}`,
      center: [77.695313, 23.160563],
      pitch: 32,
      bearing: 20,
      zoom: 3,
      maxZoom: 14,
      minZoom: 3,
      transformRequest: (url) => {
        // cache header
        return {
          url: url,
          cache: "force-cache",
        };
      },
    });

    map.on("load", async () => {
      addLayer();
    });

    /**
     * Create a buffer polygon around the cursor
     * Get all the points within the buffer
     * Highlight the points
     */
    map.on("mousemove", async (e) => {
      let mousePosition = [e.lngLat.lng, e.lngLat.lat];
      bufferPolygon = buffer(point(mousePosition), 100);
      eventsInHighlight = pointsWithinPolygon(resourceBlob, bufferPolygon);
      map.getSource("buffer").setData(bufferPolygon);
    });

    // map.on("zoomend", async () => {
    //   mapZoomLevel = map.getZoom();
    // });
  });
</script>

<svelte:window
  on:mousemove={async (e) => (cursorPosition = { x: e.clientX, y: e.clientY })}
/>

<!-- Map and sidebar section -->
<section id="map" class=" h-full w-full">
  <div bind:this={mapContainer} class="h-full w-full" />
</section>

<!-- <div
  style="position: absolute; z-index: 1000; top: {cursorPosition.y -
    squareSize / 2}px; left: {cursorPosition.x -
    squareSize / 2}px; pointer-events: none"
>
  <svg width="400" height="400" xmlns="http://www.w3.org/2000/svg">
    <rect
      x="0"
      y="0"
      width={squareSize}
      height={squareSize}
      fill="transparent"
      stroke="black"
      stroke-width="5"
    />
  </svg>
</div>
