<template>
<v-container>
  <v-row no-gutters>
    <v-col
      cols="12"
      sm="12"
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
          id="map3"
        >
        </v-img>
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
import 'leaflet-editable';

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
    longitude: 11.6,
    latitude: 48.13,
  }),
  mounted(){

      // the following part is mostly based on an example from https://esri.github.io/esri-leaflet/examples/editable.html

      this.map = L.map('map3',{editable: true, doubleClickZoom: false}).setView([this.latitude, this.longitude], 16);
      /*this.tileLayer = L.tileLayer.wms('https://geoportal.muenchen.de/geoserver/gsm/wms?', {
          maxZoom: 30,
          //layers: 'g_stadtkarte_gesamt'
          layers: 'g_osm-gsm-mvg-style',
          attribution: '&copy; <a href="https://www.muenchen.de/rathaus/Stadtverwaltung/Kommunalreferat/geodatenservice/geobasisdaten.html">GeodatenService München</a>, &copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>'
      }).addTo(this.map);*/
      
      this.tileLayer = L.tileLayer(
        'https://cartodb-basemaps-{s}.global.ssl.fastly.net/rastertiles/voyager/{z}/{x}/{y}.png',
        {
          maxZoom: 30,
          attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>, &copy; <a href="https://carto.com/attribution">CARTO</a>',
        }
      );     
      this.tileLayer.addTo(this.map);

      this.buildings = FeatureLayer({
        //url: 'https://services6.arcgis.com/o35AqnOAAmCIYvxP/arcgis/rest/services/buildings2/FeatureServer/0'
        url: 'https://services6.arcgis.com/o35AqnOAAmCIYvxP/arcgis/rest/services/buildings_utm/FeatureServer/0'
      }).addTo(this.map);

      this.buildings.setStyle({
        color: '#576973',
        weight: 1
      });

      L.EditControl = L.Control.extend({
          options: {
              position: 'topleft',
              callback: null,
              kind: '',
              html: ''
          },
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

      L.NewPolygonControl = L.EditControl.extend({
          options: {
              position: 'topleft',
              callback: this.map.editTools.startPolygon,
              kind: 'polygon',
              html: '▰'
          }
      });

      L.NewRectangleControl = L.EditControl.extend({
          options: {
              position: 'topleft',
              callback: this.map.editTools.startRectangle,
              kind: 'rectangle',
              html: '⬛'
          }
      });

      this.map.addControl(new L.NewPolygonControl());
      this.map.addControl(new L.NewRectangleControl());

      this.buildings.on('click', (e) => {
        if ((e.originalEvent.ctrlKey || e.originalEvent.metaKey) && e.layer.editEnabled()) {
          e.layer.editor.deleteShapeAt(e.latlng);
          this.buildings.deleteFeature(e.layer.feature.id);
        }
      });

      this.buildings.on('dblclick', (e) => {
        e.layer.toggleEdit();
        if (!e.layer.editEnabled()) {
          this.buildings.updateFeature(e.layer.toGeoJSON());
        }
      });

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
</style>