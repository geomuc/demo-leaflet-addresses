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
            <v-text-field
              label="type street"
              v-model="streetname"
              v-on:input="streetChanged()"
            ></v-text-field>
            <v-btn 
              color="primary"
              v-on:click="findStreet()">
              findStreet
              </v-btn>
            <v-text-field
              label="type number"
              v-model="housenumber"
              v-on:input="housenumberChanged()"
            ></v-text-field>
            <v-btn 
              color="primary"
              v-on:click="saveAddress()">
              SaveAddress
              </v-btn>
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
    markercoord: L.latLng(48.13,11.595),
    markercoord2: L.latLng(48.13,11.605),
    poslabel: [],
    poslabel2: [],
    mitte: [],
    hnrtext: "22a",
    angle: 0,
    winkelxy: 0,
    streetname: "Müllerstr.",
    housenumber: "53"
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


      this.poslabel[0]=this.markercoord.lng;
      this.poslabel[1]=this.markercoord.lat;
      this.poslabel2[0]=this.markercoord2.lng;
      this.poslabel2[1]=this.markercoord2.lat;
      this.mitte[0] = (this.markercoord.lng+this.markercoord2.lng)/2;
      this.mitte[1] = (this.markercoord.lat+this.markercoord2.lat)/2;

      this.drawLine();
      this.drawMarker();
      this.drawMarker2();
      this.drawMittemarker();


      this.myIcon = L.divIcon({className: 'lc3', html:'<div class="lc3" id="lcid2">'+this.housenumber+'</div>',	iconSize: [0, 0], iconAnchor: [0, 0]});
      this.drawHnrtext();
      //this.hnrtextmarker=L.marker([this.mitte[1],this.mitte[0]], {icon: this.myIcon, rotationAngle: this.angle,}).addTo(this.map);

      
  },
  methods: {
    changeLine: function(){

      this.line.remove();
      this.drawLine();
      this.hnrtextmarker.remove();
      this.drawHnrtext();
      this.mittemarker.remove();
      this.drawMittemarker();
    },
    moveLine: function(dlat,dlong){
      this.poslabel[0]=this.poslabel[0]+dlong;
      this.poslabel[1]=this.poslabel[1]+dlat;
      this.poslabel2[0]=this.poslabel2[0]+dlong;
      this.poslabel2[1]=this.poslabel2[1]+dlat;
      //this.mitte[0] = (this.poslabel[0]+this.poslabel2[0])/2;
      //this.mitte[1] = (this.poslabel[1]+this.poslabel2[1])/2;

      this.line.remove();
      this.drawLine();
      this.marker.remove();
      this.drawMarker();
      this.marker2.remove();
      this.drawMarker2();
      this.hnrtextmarker.remove();
      this.drawHnrtext();
      this.mittemarker.remove();
      this.drawMittemarker();

    },
    drawLine: function(){
      this.line = L.polyline([
        [this.poslabel[1],this.poslabel[0]],
        [this.poslabel2[1],this.poslabel2[0]],
      ], {
          weight: 2,
          color: 'green',
          opacity: 1.0,
      }).addTo(this.map);
    },
    drawMittemarker: function(){
      this.mittemarker = L.marker([this.mitte[1],this.mitte[0]]).addTo(this.map);
      this.mittemarker.snapediting = new L.Handler.MarkerSnap(this.map, this.mittemarker);
      this.mittemarker.snapediting.addGuideLayer(this.buildings);
      this.mittemarker.snapediting.addGuideLayer(this.line);
      this.mittemarker.snapediting.enable();

      this.mittemarker.on("dragend",(e) => {
        var pos=e.target.getLatLng();
        var mittelat_alt = this.mitte[1];
        var mittelong_alt = this.mitte[0];
        this.mitte[1]=pos.lat;
        this.mitte[0]=pos.lng;
        var delta_lat = this.mitte[1]-mittelat_alt;
        var delta_long = this.mitte[0]-mittelong_alt;
        this.moveLine(delta_lat,delta_long);
      });
    },
    drawMarker: function(){
      this.marker = L.marker([this.poslabel[1],this.poslabel[0]]).addTo(this.map);
      this.marker.snapediting = new L.Handler.MarkerSnap(this.map, this.marker);
      this.marker.snapediting.addGuideLayer(this.buildings);
      this.marker.snapediting.addGuideLayer(this.line);
      this.marker.snapediting.enable();

      this.marker.on("dragend",(e) => {
        var pos=e.target.getLatLng();
        this.poslabel[1]=pos.lat;
        this.poslabel[0]=pos.lng;
        this.changeMarker2();
        this.changeLine();
      });
    },
    drawMarker2: function(){
      this.marker2 = L.marker([this.poslabel2[1],this.poslabel2[0]]).addTo(this.map);
      this.marker2.snapediting = new L.Handler.MarkerSnap(this.map, this.marker2);
      this.marker2.snapediting.addGuideLayer(this.buildings);
      this.marker2.snapediting.addGuideLayer(this.line);
      this.marker2.snapediting.enable();

      this.marker2.on("dragend",(e) => {
        var pos=e.target.getLatLng();
        this.poslabel2[1]=pos.lat;
        this.poslabel2[0]=pos.lng;
        this.changeMarker();
        this.changeLine();
      });
    },
    drawHnrtext: function(){
      this.myIcon = L.divIcon({className: 'lc3', html:'<div class="lc3" id="lcid2">'+this.housenumber+'</div>',	iconSize: [0, 0], iconAnchor: [0, 0]});
      this.hnrtextmarker=L.marker([this.mitte[1],this.mitte[0]], {icon: this.myIcon, rotationAngle: this.winkelxy}).addTo(this.map);
    },
    changeMarker2: function(){
      /********************
       * Falls der "linke" Marker bewegt wird, wird auch der "rechte" (marker2) mitrotiert 
      ********************/
      var angle_alt=this.angle;
      var s2_alt =  Math.sqrt((this.poslabel2[1]-this.mitte[1])**2+(this.poslabel2[0]-this.mitte[0])**2);
      var neux=this.poslabel[1]-this.mitte[1];
      var neuy=this.poslabel[0]-this.mitte[0];
      var angleneu = Math.atan2(neuy, neux)* (180 / Math.PI)-90;
      if (angleneu <0){angleneu=angleneu+360};
      if (angleneu >360){angleneu=angleneu-360};
      this.angle=angleneu+180;
      if (this.angle <0){this.angle=this.angle+360};
      if (this.angle >360){this.angle=this.angle-360};
      this.winkelxy=Math.atan2(neuy, neux*1.5)* (180 / Math.PI)-90+180;;
      if (this.winkelxy <0){this.winkelxy=this.winkelxy+360};
      if (this.winkelxy >360){this.winkelxy=this.winkelxy-360};
      this.poslabel2[1]=this.mitte[1]+((Math.sin(angleneu/180*Math.PI))*s2_alt);
      this.poslabel2[0]=this.mitte[0]-((Math.cos(angleneu/180*Math.PI))*s2_alt);
      this.line.remove();
      this.drawLine();
      this.marker2.remove();
      this.drawMarker2();
    },
    changeMarker: function(){
      /********************
       * Falls der "rechte" Marker bewegt wird, wird auch der "linke" (marker2) mitrotiert 
      ********************/
      var angle_alt=this.angle;
      var s_alt =  Math.sqrt((this.poslabel[1]-this.mitte[1])**2+(this.poslabel[0]-this.mitte[0])**2);
      var neux=this.poslabel2[1]-this.mitte[1];
      var neuy=this.poslabel2[0]-this.mitte[0];
      var angleneu = Math.atan2(neuy, neux)* (180 / Math.PI)-90;
      if (angleneu <0){angleneu=angleneu+360};
      if (angleneu >360){angleneu=angleneu-360};
      this.winkelxy=Math.atan2(neuy, neux*1.5)* (180 / Math.PI)-90;;
      if (this.winkelxy <0){this.winkelxy=this.winkelxy+360};
      if (this.winkelxy >360){this.winkelxy=this.winkelxy-360};
      this.angle=angleneu;
      this.poslabel[1]=this.mitte[1]+((Math.sin(this.angle/180*Math.PI))*s_alt);
      this.poslabel[0]=this.mitte[0]-((Math.cos(this.angle/180*Math.PI))*s_alt);
      this.line.remove();
      this.drawLine();
      this.marker.remove();
      this.drawMarker();
    },
    streetChanged: function(){
      console.log("streetChanged");
    },
    housenumberChanged: function(){
      this.hnrtextmarker.remove();
      this.drawHnrtext();
      this.mittemarker.remove();
      this.drawMittemarker();
    },
    findStreet: function(){
      console.log("findStreet");
    },
    saveAddress: function(){
      console.log("saveAddress");
    }

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

.lc3 {
  width: 30px;
  height: 30px;
  /*background-color:rgb(197, 248, 213);*/
  color: #292268;
  transform: rotate(0deg);
  text-align: left;
  font-size: 1.2em;
  font-weight: bold;
}
</style>
