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
          id="map6"
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
import 'leaflet-rotatedmarker';
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
    labelmarker: L.latLng(48.13,11.6),
  }),
  mounted(){

      this.map = L.map('map6',{editable: true, doubleClickZoom: false}).setView([this.latitude, this.longitude], 16);
      this.tileLayer = L.tileLayer.wms('https://geoportal.muenchen.de/geoserver/gsm/wms?', {
          maxZoom: 30,
          layers: 'g_stadtkarte_gesamt'
      }).addTo(this.map);
      /*this.tileLayer = L.tileLayer(
        'https://cartodb-basemaps-{s}.global.ssl.fastly.net/rastertiles/voyager/{z}/{x}/{y}.png',
        {
          maxZoom: 30,
          attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>, &copy; <a href="https://carto.com/attribution">CARTO</a>',
        }
      );     
      this.tileLayer.addTo(this.map);*/

      var strl = "Hans-Gustav-Strasse";
      var angle = 30;
      var center = this.labelmarker;
      var laengepx = 200;
      this.marker = L.marker(center, {
            //rotationAngle: e.feature.properties.angle,
            rotationAngle: angle,
            draggable: true,
            icon: L.divIcon({
              iconSize: null,
              className: 'label',
              iconAnchor:   [0,0],
              //html: '<div class="sl">' +strl + '</div>'
              html: '<div class="sl2"><input type="text" name="lname" value="Strasse"></div>',
              //html: '<div> Testlabel </div>'
            })
      }).addTo(this.map);

      /*var icon = L.icon({
        iconUrl: 'https://esri.github.io/esri-leaflet/img/earthquake-icon.png',
        iconSize: [10, 20],
        iconAnchor: [0, 0]

      });*/

      /*var icon2 = L.icon({
        iconUrl: 'https://esri.github.io/esri-leaflet/img/earthquake-icon.png',
        iconSize: [10, 20],
        iconAnchor: [laengepx*(-1),0]
      });⚫*/

      var icon2 =  L.divIcon({
              iconSize: 20,
              className: 'rp',
              iconAnchor:   [laengepx*(-1),0],
              html: '⚫'
      });



      /*var pkt1 = L.marker(center, {
        rotationAngle: 0,
        //rotationAngle: 30,
        //draggable: true,
        icon: icon
      }).addTo(this.map);*/

      this.pkt2 = L.marker(center, {
        rotationAngle: angle,
        //rotationAngle: 0,
        draggable: true,
        icon: icon2
      }).addTo(this.map);

      this.marker.on("dragend",(e) => {
        center=e.target.getLatLng();
        this.pkt2.setLatLng(center);
      });

      this.pkt2.on("dragend",(e) => {
        console.log("hat sich bewegt");
        console.log(e);
        console.log(e.sourceTarget._startPos.x);
        console.log(e.sourceTarget._newPos.x);
        var diffx=e.sourceTarget._newPos.x-e.sourceTarget._startPos.x;
        var diffy=e.sourceTarget._newPos.y-e.sourceTarget._startPos.y;
        console.log(diffx);
        console.log(diffy);
        var angle = e.target.options.rotationAngle;
        var neux=Math.cos(angle/180*Math.PI)*laengepx+diffx;
        var neuy=Math.sin(angle/180*Math.PI)*laengepx+diffy;
        var angleneu = Math.atan2(neuy, neux)* (180 / Math.PI); // In radians
        if (angleneu <0){angleneu=angleneu+360};
        console.log("neuer Winkel");
        console.log(angleneu);
        this.marker.setRotationAngle(angleneu);
        this.pkt2.setLatLng(center);
        this.pkt2.setRotationAngle(angleneu);
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
  background-color: white;
  /*color: #bd924d;*/
  color: black;
  text-align: left;
  font-size: 1.2em;
  font-weight: bold;
}
</style>
