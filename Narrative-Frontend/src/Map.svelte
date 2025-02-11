<script>
  // @ts-ignore
  import maplibre from "maplibre-gl";
  import "maplibre-gl/dist/maplibre-gl.css";
  import { mount, onMount } from "svelte";
  import MarkerPopup from "./MarkerPopup.svelte";
  import { buffer, point, bbox, pointsWithinPolygon } from "@turf/turf";

  const publicMaptilerKey = "XtQybTQjRpKFSRHVSG0G";
  export let map;
  let filterExpression;
  let appearanceExpression;
  let mapContainer;
  let resourceJSON;
  let resourceBlob;
  let bufferPolygon;
  export let eventsInHighlight;

  let cursorPosition = { x: 0, y: 0 };

  async function getResource() {
    return await fetch("/HWdb_geocoded.geojson").then(async (response) => {
      return response.json();
    });
  }

  const addLayer = async () => {
    resourceBlob = await getResource();

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

      while (Math.abs(e.lngLat.lng - coordinates[0]) > 180) {
        coordinates[0] += e.lngLat.lng > coordinates[0] ? 360 : -360;
      }
      let popup = new maplibre.Popup()
        .setLngLat(coordinates)
        .setHTML("")
        .addTo(map);
      let child = popup.getElement();

      mount(MarkerPopup, {
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

    map.on("mouseenter", "point", async () => {
      map.getCanvas().style.cursor = "pointer";
    });

    map.on("mouseleave", "point", async () => {
      map.getCanvas().style.cursor = "";
    });
  };

  onMount(async () => {
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
        return { url: url, cache: "force-cache" };
      },
    });

    map.on("load", async () => {
      addLayer();
    });

    map.on("mousemove", async (e) => {
      let mousePosition = [e.lngLat.lng, e.lngLat.lat];
      bufferPolygon = buffer(point(mousePosition), 100);
      eventsInHighlight = pointsWithinPolygon(resourceBlob, bufferPolygon);
      map.getSource("buffer").setData(bufferPolygon);
    });
  });
</script>

<svelte:window
  on:mousemove={async (e) => (cursorPosition = { x: e.clientX, y: e.clientY })}
/>

<section id="map" class="h-full w-full">
  <div bind:this={mapContainer} class="h-full w-full"></div>
</section>

<!-- Optional debugging SVG or additional visual elements -->
<!--
  <div style="position: absolute; z-index: 1000; top: {cursorPosition.y - squareSize / 2}px; left: {cursorPosition.x - squareSize / 2}px; pointer-events: none">
    <svg width="400" height="400" xmlns="http://www.w3.org/2000/svg">
      <rect x="0" y="0" width={squareSize} height={squareSize} fill="transparent" stroke="black" stroke-width="5" />
    </svg>
  </div>
-->
