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
          id="map9"
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
//import 'leaflet-editable';
//import 'leaflet.fullscreen';
//import 'leaflet.fullscreen/Control.FullScreen.css';

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
    markercoord: L.latLng(48.13,11.595),
    markercoord2: L.latLng(48.13,11.605),
    poslabel: [],
    poslabel2: [],
    mitte: [],
    hnrtext: "22a",
    angle: 0
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

      this.map = L.map('map9',{crs: this.crs, editable: true, doubleClickZoom: false}).setView([48.13, 11.6], 16);
      this.tileLayer = L.tileLayer.wms('https://geoportal.muenchen.de/geoserver/gsm/wms?', {
          maxZoom: 20,
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

      //this.marker = L.marker(this.markercoord,{draggable: true}).addTo(this.map);
      this.marker = L.marker(this.markercoord).addTo(this.map);
      this.marker.snapediting = new L.Handler.MarkerSnap(this.map, this.marker);
      //this.marker.snapediting.addGuideLayer(guides);
      this.marker.snapediting.addGuideLayer(this.buildings);
      this.marker.snapediting.enable();

      this.line = L.polyline([
          this.markercoord,
          this.markercoord2,
      ], {
          weight: 2,
          color: 'red',
          opacity: 1.0
      }).addTo(this.map);
      this.mitte[0] = (this.markercoord.lng+this.markercoord2.lng)/2;
      this.mitte[1] = (this.markercoord.lat+this.markercoord2.lat)/2;
      this.mittemarker = L.marker([this.mitte[1],this.mitte[0]]).addTo(this.map);

    this.myIcon = L.divIcon({className: 'lc3', html:'<div class="lc3" id="lcid2">'+this.hnrtext+'</div>',	iconSize: [0, 0],
			iconAnchor: [0, 0]});
    this.hnrtextmarker=L.marker([this.mitte[1],this.mitte[0]], {icon: this.myIcon,         rotationAngle: this.angle,
}).addTo(this.map);

      this.poslabel[0]=this.markercoord.lng;
      this.poslabel[1]=this.markercoord.lat;

      this.marker.on("dragend",(e) => {
        var pos=e.target.getLatLng();
        this.poslabel[1]=pos.lat;
        this.poslabel[0]=pos.lng;
        console.log("Punkt1")
        console.log(this.poslabel);
        this.changeLine();
      });

      this.marker2 = L.marker(this.markercoord2).addTo(this.map);
      this.marker2.snapediting = new L.Handler.MarkerSnap(this.map, this.marker2);
      //this.marker.snapediting.addGuideLayer(guides);
      this.marker2.snapediting.addGuideLayer(this.buildings);
      this.marker2.snapediting.enable();

      this.poslabel2[0]=this.markercoord2.lng;
      this.poslabel2[1]=this.markercoord2.lat;

      this.marker2.on("dragend",(e) => {
        var pos=e.target.getLatLng();
        this.poslabel2[1]=pos.lat;
        this.poslabel2[0]=pos.lng;
        console.log("Punkt2")
        console.log(this.poslabel2);
        this.changeLine();
      });

      /*this.line = L.polyline([
          [this.poslabel[1],this.poslabel[0]],
          [this.poslabel2[1],this.poslabel2[0]],
      ], {
          weight: 5,
          color: 'red',
          opacity: 1.0
      }).addTo(this.map);*/
      
  },
  methods: {
    changeLine: function(){
      this.line.remove();
      this.line = L.polyline([
        [this.poslabel[1],this.poslabel[0]],
        [this.poslabel2[1],this.poslabel2[0]],
      ], {
          weight: 2,
          color: 'red',
          opacity: 1.0
      }).addTo(this.map);

      this.mittemarker.remove();
      this.mitte[0] = (this.poslabel[0]+this.poslabel2[0])/2;
      this.mitte[1] = (this.poslabel[1]+this.poslabel2[1])/2;
      this.mittemarker = L.marker([this.mitte[1],this.mitte[0]]).addTo(this.map);

      var neux=this.poslabel2[1]-this.poslabel[1];
      var neuy=this.poslabel2[0]-this.poslabel[0];
      var angleneu = Math.atan2(neuy, neux)* (180 / Math.PI)-90; // In radians
      if (angleneu <0){angleneu=angleneu+360};
      if (angleneu >360){angleneu=angleneu-360};
      console.log(angleneu);
      this.angle=angleneu;

      this.hnrtextmarker.remove();
      this.hnrtextmarker=L.marker([this.mitte[1],this.mitte[0]], {icon: this.myIcon, rotationAngle: this.angle}).addTo(this.map);

    }
  }
};
</script>


<style>

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

.lc3 {
  width: 200px;
  height: 30px;
  /*background-color:rgb(197, 248, 213);*/
  color: #292268;
  transform: rotate(0deg);
  text-align: left;
  font-size: 1.2em;
  font-weight: bold;
}
</style>
