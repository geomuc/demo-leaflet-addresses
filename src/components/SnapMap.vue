<template>
<v-container>
  <v-row no-gutters>
    <v-col
      cols="12"
      sm="8"
    >
      <v-card
        class="pa-2"
        outlined
        tile
        height="500px"
        width="100%"
      >
        <v-img
          height="100%"
          width="100%"
          id="map8"
        >
        </v-img>
      </v-card>
    </v-col>
    <v-col
            cols="12"
            sm="4"
          >
            <v-card
              class="pa-2"
              outlined
              tile
            >
            </v-card>
          </v-col>

  </v-row>
</v-container>
</template>

<script>
/* eslint-disable */
import 'leaflet';
import 'leaflet/dist/leaflet.css';
import FeatureLayer from 'esri-leaflet/src/Layers/FeatureLayer/FeatureLayer';
import 'proj4';
import 'proj4leaflet';
import 'leaflet-geometryutil';
import 'leaflet-snap';

delete L.Icon.Default.prototype._getIconUrl;

L.Icon.Default.mergeOptions({
  iconRetinaUrl: require('leaflet/dist/images/marker-icon-2x.png'),
  iconUrl: require('leaflet/dist/images/marker-icon.png'),
  shadowUrl: require('leaflet/dist/images/marker-shadow.png')
});

export default {
  props: {
    irgendwas: String,
  },
  data: () => ({
    rw: 694451.45,
    hw: 5334297.15,
    markercoord: L.latLng(48.13,11.6),
  }),
  mounted(){

this.crs = new L.Proj.CRS('EPSG:25832',
      '+proj=utm +zone=32 +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +units=m +no_defs',
      {
    origin: [677016.63,6491169.04],
    resolutions: [
      132291.9312505292,
      66145.9656252646,
      26458.386250105836,
      19843.789687579378,
      13229.193125052918,
      6614.596562526459,
      2645.8386250105837,
      1322.9193125052918,
      661.4596562526459,
      264.5838625010584,
      132.2919312505292,
      66.1459656252646,
      26.458386250105836,
      19.843789687579378,
      13.229193125052918,
      6.614596562526459,
      2.6458386250105836,
      1.3229193125052918,
      0.6614596562526459
    ]
      })

      this.map = L.map('map8',{crs: this.crs, editable: true, doubleClickZoom: false}).setView([48.13, 11.6], 16);
      this.tileLayer = L.tileLayer.wms('https://geoportal.muenchen.de/geoserver/gsm/wms?', {
          maxZoom: 19,
          minZoom: 1,
          layers: 'g_stadtkarte_gesamt'
      }).addTo(this.map);

      this.buildings = FeatureLayer({
        url: 'https://services6.arcgis.com/o35AqnOAAmCIYvxP/arcgis/rest/services/buildings2/FeatureServer/0',
        attribution: '&copy; <a href="https://www.muenchen.de/rathaus/Stadtverwaltung/Kommunalreferat/geodatenservice/geobasisdaten.html">GeodatenService München</a>'

        //url: 'https://services6.arcgis.com/o35AqnOAAmCIYvxP/arcgis/rest/services/buildings_utm/FeatureServer/0'
      }).addTo(this.map);

      this.buildings.setStyle({
        color: '#576973',
        weight: 1
      });
      
      var guides = L.polyline([
          [48.128777,11.599889],
          [48.130138,11.602892],
      ], {
          weight: 5,
          color: 'red',
          opacity: 1.0
      }).addTo(this.map);

      this.marker = L.marker(this.markercoord).addTo(this.map);
      this.marker.snapediting = new L.Handler.MarkerSnap(this.map, this.marker);
      this.marker.snapediting.addGuideLayer(guides);
      this.marker.snapediting.addGuideLayer(this.buildings);
      this.marker.snapediting.enable();
      
  },
  methods: {
  }
};
</script>


<style>

.leaflet-fade-anim .leaflet-tile,.leaflet-zoom-anim .leaflet-zoom-animated { will-change:auto !important; }

.leaflet-container {
    background-color:rgba(255,0,0,0.0);
}

.sl2 {
  width: 200px;
  height: 25px;
  background-color: yellow;
  /*color: #bd924d;*/
  color: black;
  text-align: left;
  font-size: 1.2em;
  font-weight: bold;
  opacity: 0.5 ;
}
</style>
