<template>
  <v-content>
    <v-container 
        fluid
      >
        <v-row no-gutters>
          <v-col
            cols="12"
            sm="4"
          >
            <v-card
              class="pa-2"
              outlined
              tile
            >
            <v-select
              v-model= "auswahl"
              :items="items"
              label="Select Address"
              outlined
              v-on:change="setMarker()"
            ></v-select>
            <v-text-field
              label="type label"
              v-model="labeltext"
              v-on:input="textChanged()"
            ></v-text-field>
            <v-text-field
              label="type angle"
              v-model="labelangle"
              v-on:input="angleChanged()"
            ></v-text-field>
            </v-card>
          </v-col>
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
                id="map"
              >
              </v-img>
            </v-card>
          </v-col>
        </v-row>
    </v-container>
    <editable-map/>
    <editable-labels2/>
    <srid25832/>
    <snap-map/>
    <linie-map/>
  </v-content>
</template>

<script>
/* eslint-disable */
import EditableMap from './EditableMap.vue';
import EditableLabels2 from './EditableLabels2.vue';
import Srid25832 from './Srid25832.vue';
import SnapMap from './SnapMap.vue';
import LinieMap from './LinieMap.vue';
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';
import 'leaflet-rotatedmarker';
import BasemapLayer from 'esri-leaflet/src/Layers/BasemapLayer';
import FeatureLayer from 'esri-leaflet/src/Layers/FeatureLayer/FeatureLayer';

delete L.Icon.Default.prototype._getIconUrl;

L.Icon.Default.mergeOptions({
  iconRetinaUrl: require('leaflet/dist/images/marker-icon-2x.png'),
  iconUrl: require('leaflet/dist/images/marker-icon.png'),
  shadowUrl: require('leaflet/dist/images/marker-shadow.png')
});

export default {
  components: { EditableMap, EditableLabels2, Srid25832, SnapMap, LinieMap},
  props: {
    source: String,
  },
  data: () => ({
    drawer: null,
    items: ['11, Dingolfingerstr.','9, Dingolfingerstr.','7, Dingolfingerstr.','5, Dingolfingerstr.' ],
    auswahl: [],
    longitude: 11.6,
    latitude: 48.13,
    longout: 11.6,
    latout: 48.13,
    labeltext: "Dingolfingerstr.",
    markercoord: L.latLng(48.13,11.6),
    labelmarker: L.latLng(48.13,11.6),
    staticAnchor: [0, 0],
    labelangle: 30,
    addresses: [{"id":1,"street":"Dingolfingerstr.","number":11,"location":{"type":"Point","coordinates":[11.615049,48.132873]}},{"id":2,"street":"Dingolfingerstr.","number":9,"location":{"type":"Point","coordinates":[11.61449,48.132632]}},{"id":3,"street":"Dingolfingerstr.","number":7,"location":{"type":"Point","coordinates":[11.614051,48.132444]}},{"id":4,"street":"Dingolfingerstr.","number":5,"location":{"type":"Point","coordinates":[11.613666,48.132249]}}],
  }),
  mounted(){

       this.map = L.map('map').setView([this.latitude, this.longitude], 16);
      /*this.tileLayer = L.tileLayer(
        'https://cartodb-basemaps-{s}.global.ssl.fastly.net/rastertiles/voyager/{z}/{x}/{y}.png',
        {
          maxZoom: 20,
          attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>, &copy; <a href="https://carto.com/attribution">CARTO</a>',
        }
      );     
      this.tileLayer.addTo(this.map);*/

    this.roads = FeatureLayer({
      url: 'https://services6.arcgis.com/o35AqnOAAmCIYvxP/arcgis/rest/services/roads_utm/FeatureServer/0'
    }).addTo(this.map);

    this.roads.setStyle({
      color: '#849fd1',
      weight: 2
    })

    this.buildings = FeatureLayer({
      url: 'https://services6.arcgis.com/o35AqnOAAmCIYvxP/arcgis/rest/services/buildings_utm/FeatureServer/0'
    }).addTo(this.map);

    this.buildings.setStyle({
      color: '#576973',
      weight: 1
    })

    this.marker = L.marker(this.markercoord,{draggable: true}).addTo(this.map);

    this.myIcon2 = L.divIcon({className: 'lc', html:'<div class="lc" id="lcid2">'+this.labeltext+'</div>',	iconSize: [0, 0],
			iconAnchor: [0, 0]});
    this.markerrotated=L.marker(this.markercoord, {icon: this.myIcon2,
    rotationAngle: this.labelangle, draggable: true
}).addTo(this.map);

  },
  methods: {
    setMarker(){  
      var res = this.auswahl.split(",")[0];
      var address = "";
      this.addresses.forEach(function(element) {
          if (element.number==res){
            address=element;
          };
      });
      var long = address.location.coordinates[0];
      var lat =  address.location.coordinates[1];
      this.marker.setLatLng([lat,long]);
      this.markerrotated.setLatLng([lat,long]);
      this.map.setView([lat,long], 17);
    },
    textChanged(){
        document.getElementById('lcid2').innerText=this.labeltext;
    },
    angleChanged(){
        this.markerrotated.setRotationAngle(this.labelangle);
    },
  }
};
</script>

<style>

.leaflet-container {
    background-color:rgba(255,0,0,0.0);
}

.lc {
  width: 200px;
  height: 30px;
  background-color:rgb(197, 248, 213);
  color: #292268;
  transform: rotate(0deg);
  text-align: center;
  font-size: 1.2em;
  font-weight: bold;
}
</style>