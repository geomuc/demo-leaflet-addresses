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
          id="map7"
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
            <v-text-field
              label="Rechtswert"
              v-model="rw"
            ></v-text-field>
              <v-text-field
              label="Hochwert"
              v-model="hw"
            ></v-text-field>
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
import 'leaflet-editable';
import 'leaflet.fullscreen';
import 'leaflet.fullscreen/Control.FullScreen.css';

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

    //labelmarker: L.latLng(48.13,11.6),
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

      this.map = L.map('map7',{crs: this.crs, editable: true, doubleClickZoom: false}).setView([48.13, 11.6], 16);
      this.tileLayer = L.tileLayer.wms('https://geoportal.muenchen.de/geoserver/gsm/wms?', {
          maxZoom: 19,
          minZoom: 1,
          layers: 'g_stadtkarte_gesamt'
      }).addTo(this.map);

      this.map.on('moveend', (e) => { 
        var lat=this.map.getCenter().lat;
        var lng =this.map.getCenter().lng;
        console.log(this.map.getCenter()); 
        console.log(lat+" "+lng);
        var xy = this.crs.project({lat,lng});
        this.rw = xy.x;
        this.hw = xy.y;
      });

      this.buildings = FeatureLayer({
        url: 'https://services6.arcgis.com/o35AqnOAAmCIYvxP/arcgis/rest/services/buildings2/FeatureServer/0',
        attribution: '&copy; <a href="https://www.muenchen.de/rathaus/Stadtverwaltung/Kommunalreferat/geodatenservice/geobasisdaten.html">GeodatenService München</a>'

        //url: 'https://services6.arcgis.com/o35AqnOAAmCIYvxP/arcgis/rest/services/buildings_utm/FeatureServer/0'
      }).addTo(this.map);

      this.buildings.setStyle({
        color: '#576973',
        weight: 1
      });

      console.log("ich bin hier");

      // create a generic control to invoke editing
      L.EditControl = L.Control.extend({
          options: {
              position: 'topleft',
              callback: null,
              kind: '',
              html: ''
          },
          // when the control is added to the map, wire up its DOM dynamically and add a click listener
          onAdd: function (map) {
              var container = L.DomUtil.create('div', 'leaflet-control leaflet-bar'),
                  link = L.DomUtil.create('a', '', container);
              link.href = '#';
              link.title = 'Create a new ' + this.options.kind;
              link.innerHTML = this.options.html;
              L.DomEvent.on(link, 'click', L.DomEvent.stop)
                        .on(link, 'click', function () {
                          window.LAYER = this.options.callback.call(map.editTools);
                        }, this);
              return container;
          }
      });

console.log("ich bin hier2");

      // extend the control to draw polygons
      L.NewPolygonControl = L.EditControl.extend({
          options: {
              position: 'topleft',
              callback: this.map.editTools.startPolygon,
              kind: 'polygon',
              html: '▰'
          }
      });
console.log("ich bin hier3");
      // extend the control to draw rectangles
      L.NewRectangleControl = L.EditControl.extend({
          options: {
              position: 'topleft',
              callback: this.map.editTools.startRectangle,
              kind: 'rectangle',
              html: '⬛'
          }
      });

      // create fullscreen control
      var fsControl = L.control.fullscreen();
      // add fullscreen control to the map
      this.map.addControl(fsControl);

      // add the two new controls to the map
      this.map.addControl(new L.NewPolygonControl());
      this.map.addControl(new L.NewRectangleControl());



      // detect fullscreen toggling
      this.map.on('enterFullscreen', function(){
        if(window.console) window.console.log('enterFullscreen');
      });
      this.map.on('exitFullscreen', function(){
        if(window.console) window.console.log('exitFullscreen');
      });

console.log("ich bin hier4");

      // when users CMD/CTRL click an editable feature, remove it from the map and delete it from the service
      this.buildings.on('click', (e) => {
        if ((e.originalEvent.ctrlKey || e.originalEvent.metaKey) && e.layer.editEnabled()) {
          e.layer.editor.deleteShapeAt(e.latlng);
          // delete expects an id, not the whole geojson object
          this.buildings.deleteFeature(e.layer.feature.id);
        }
      });

      // when users double click a graphic toggle its editable status
      // when deselecting, pass the geometry update to the service
      this.buildings.on('dblclick', (e) => {
        e.layer.toggleEdit();
        if (!e.layer.editEnabled()) {
          this.buildings.updateFeature(e.layer.toGeoJSON());
        }
      });

      // when a new feature is drawn using one of the custom controls, pass the edit to the service
      this.map.on('editable:drawing:commit', (e) => {
        this.buildings.addFeature(e.layer.toGeoJSON());
        e.layer.toggleEdit();
      });

      
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
