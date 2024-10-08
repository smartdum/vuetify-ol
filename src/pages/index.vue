<template>
  <div ref="mapRoot" style="width: 800px; height: 600px"></div>
  <div class="text-center">
    Hello, World!
    <v-btn @click="zoomTo(-2, 47)">Zoom to -2 and 47</v-btn>
    <div id="mouse-coords">Coordinates: {{ coords.lon }}, {{ coords.lat }}</div>
  </div>
</template>

<script lang="ts" setup>
import { reactive, ref, onMounted } from "vue";
import View from "ol/View";
import Map from "ol/Map";
// Couches de fond (OSM)
import TileLayer from "ol/layer/Tile";
import OSM from "ol/source/OSM";
// couches vectorielles
import VectorLayer from "ol/layer/Vector";
import VectorSource from "ol/source/Vector";
import GeoJSON from "ol/format/GeoJSON";
// methodes OL pour calcul
import { fromLonLat, toLonLat } from "ol/proj";
import "ol/ol.css"; // Uncomment this if you need to import OpenLayers styles
import "ol-ext/dist/ol-ext.css";
import LayerSwitcher from "ol-ext/control/LayerSwitcher";
import LayerShop from "ol-ext/control/LayerShop";

const mapRoot = ref<HTMLDivElement | null>(null); // Vue reference to the map DOM element

const map = new Map();

const layers = [
  new TileLayer({
    source: new OSM(),
    title: "OpenStreetMap",
    basemap: true,
    name: "OpenStreetMap",
  }),
  new VectorLayer({
    source: new VectorSource({
      url: "https://openlayers.org/data/vector/us-states.json",
      format: new GeoJSON(),
    }),
    title: "US States",
    name: "US States",
  }),
];
map.setLayers(layers);

const view = new View({
  center: [0, 0],
  zoom: 2,
  projection: "EPSG:900913", // google Mercator [202589625,4587985214]
  // projection: "EPSG:4326", // GPS [-2.2589,47.3698]
});
map.setView(view);

// Reactive state to hold mouse coordinates
const coords = reactive({
  lon: 0,
  lat: 0,
});

const zoomTo = (x: number, y: number) => {
  // console.log(x, y);
  const [lon, lat] = fromLonLat([x, y]);
  map.getView().setCenter([lon, lat]);
  map.getView().setZoom(10);
  console.log(map.getView().getCenter());
};

const ctrlLayers = new LayerShop({ position: "topright" }, { layers: layers });
map.addControl(ctrlLayers);

onMounted(() => {
  if (mapRoot.value) {
    map.setTarget(mapRoot.value);

    // Add event listener for pointer move (mouse movement)
    map.on("pointermove", (event) => {
      const coordinate = event.coordinate; // Get the coordinates in the map's projection
      console.log(coordinate);
      const [lon, lat] = toLonLat(coordinate); // Convert to longitude/latitude

      // Update the reactive state
      coords.lon = lon.toFixed(6); // Limit decimal places for better readability
      coords.lat = lat.toFixed(6);
    });
  }
});
</script>
<style>
.ol-layerswitcher label {
  color: black;
}
</style>
