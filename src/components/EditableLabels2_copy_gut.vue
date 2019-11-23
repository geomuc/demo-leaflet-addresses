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
          id="map5"
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
    lastedited: {feature:""},
    labelmarker: L.latLng(48.13,11.6),
    angle: 30,
  }),
  mounted(){

      this.map = L.map('map5',{editable: true, doubleClickZoom: false}).setView([this.latitude, this.longitude], 16);
      this.tileLayer = L.tileLayer(
        'https://cartodb-basemaps-{s}.global.ssl.fastly.net/rastertiles/voyager/{z}/{x}/{y}.png',
        {
          maxZoom: 30,
          attribution: '&copy; <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a>, &copy; <a href="https://carto.com/attribution">CARTO</a>',
        }
      );     
      this.tileLayer.addTo(this.map);

      this.streetlabels = FeatureLayer({
        url: 'https://services6.arcgis.com/o35AqnOAAmCIYvxP/ArcGIS/rest/services/streetlabels/FeatureServer/0',
        pointToLayer: function (geojson, latlng) {
          console.log(geojson);
          var angle=geojson.properties.angle;
          var strl=geojson.properties.streetlabel;
          var id = geojson.id;
          console.log(id);
          return L.marker(latlng, {
            rotationAngle: angle,
            draggable: false,
            icon: L.divIcon({
              iconSize: null,
              className: 'label',
              iconAnchor:   [0,0],
              //html: '<div class="sl" id="sl'+id+'">' +strl + '</div>'
              html: '<div class="sl" id="sl'+id+'"><input type="text" name="lname" value="'+strl+'"></div>',
            })
          });
        }
      }).addTo(this.map);

      this.laengepx = 200;
      this.icon =  L.divIcon({
              iconSize: 50,
              className: 'rp',
              iconAnchor:   [this.laengepx*(-1),0],
              html: '⚫'
      });

this.streetlabels.on('dblclick', (e) => {
        this.marker=e.layer;
        this.poslabel=[e.latlng.lng,e.latlng.lat];
        console.log("Doppelklick auf Label");
        console.log(e);
        console.log(this.poslabel);
        var id=this.marker.feature.id;
        // Falls Label aktuell editierbar
        if (this.marker.editable == true)
        {
          this.marker.editable = false;
          this.marker.dragging.disable();
          document.getElementById("sl"+id).style.backgroundColor="white";
          this.map.removeLayer(this.lastpkt);
          console.log("am Ende:");
          console.log(this.marker);
        // Falls Label aktuell nicht editierbar
        } else 
        {
          console.log("zu Beginn:");
          console.log(this.marker);
          console.log(this.marker.feature.properties.angle);
          //this.poslabel = this.marker.feature.geometry.coordinates;
          //this.poslabel = this.marker.getLatLng();
          this.angle = this.marker.options.rotationAngle;
          console.log(this.angle);
          //this.marker = e.layer;
          // Falls vorhanden, vorheriges Editierlabel zuruecksetzen
          if (this.lastedited.feature!="")
          {
            console.log("bin drin");
            console.log(this.lastedited.feature.id);
            document.getElementById("sl"+this.lastedited.feature.id).style.backgroundColor="white";
            this.map.removeLayer(this.lastpkt);
            this.lastedited.editable=false;
            //this.lastedited.options.draggable=false;
            this.lastedited.dragging.disable();
          }
            this.startEditing();
         this.marker.editable = true;
          //e.layer.options.draggable=true;
          this.marker.dragging.enable();
          document.getElementById("sl"+id).style.backgroundColor="yellow";
          this.lastedited=this.marker;
          this.lastpkt=this.pkt;
        };
        //console.log(e);

      });

      /*this.streetlabels.on('dblclick', (e) => {
        console.log("Doppelklick auf Label");
        console.log(e);
        var id=e.layer.feature.id;
        // Falls Label aktuell editierbar
        if (e.layer.editable == true)
        {
          e.layer.editable = false;
          //e.layer.options.draggable=false;
          //e.layer.options.setDragging(false);
          e.layer.dragging.disable();
          document.getElementById("sl"+id).style.backgroundColor="white";
          this.map.removeLayer(this.lastmarker);
        // Falls Label aktuell nicht editierbar
        } else 
        {
          this.poslabel = e.layer.feature.geometry.coordinates;
          this.angle = e.layer.feature.properties.angle;
          this.marker = e.layer;
          // Falls vorhanden, vorheriges Editierlabel zuruecksetzen
          if (this.lastedited.feature!="")
          {
            console.log("bin drin");
            console.log(this.lastedited.feature.id);
            document.getElementById("sl"+this.lastedited.feature.id).style.backgroundColor="white";
            this.map.removeLayer(this.lastmarker);
            this.lastedited.editable=false;
            //this.lastedited.options.draggable=false;
            this.lastedited.dragging.disable();
          }
            this.startEditing();
          e.layer.editable = true;
          //e.layer.options.draggable=true;
          e.layer.dragging.enable();
          document.getElementById("sl"+id).style.backgroundColor="yellow";
          this.lastedited=e.layer;
          this.lastmarker=this.pkt;
        };
        //console.log(e);

      });*/

      /*this.pkt = L.marker(this.labelmarker, {
        rotationAngle: this.angle,
        //rotationAngle: 0,
        draggable: true,
        icon: this.icon
      }).addTo(this.map);*/

      /*this.marker.on("dragend",(e) => {
        center=e.target.getLatLng();
        this.pkt2.setLatLng(center);
      });*/

      /*this.pkt.on("dragend",(e) => {
        console.log("hat sich bewegt");
        console.log(e);
        console.log(e.sourceTarget._startPos.x);
        console.log(e.sourceTarget._newPos.x);
        var diffx=e.sourceTarget._newPos.x-e.sourceTarget._startPos.x;
        var diffy=e.sourceTarget._newPos.y-e.sourceTarget._startPos.y;
        console.log(diffx);
        console.log(diffy);
        var angle = e.target.options.rotationAngle;
        var neux=Math.cos(angle/180*Math.PI)*this.laengepx+diffx;
        var neuy=Math.sin(angle/180*Math.PI)*this.laengepx+diffy;
        var angleneu = Math.atan2(neuy, neux)* (180 / Math.PI); // In radians
        if (angleneu <0){angleneu=angleneu+360};
        console.log("neuer Winkel");
        console.log(angleneu);
        //this.marker.setRotationAngle(angleneu);
        this.pkt.setLatLng(this.labelmarker);
        this.pkt.setRotationAngle(angleneu);
      });*/



      
  },
  methods: {
    startEditing: function(){
      console.log("Beginne mit Editieren ...");
      //if (this.marker) { // check
      //  this.map.removeLayer(this.marker); // remove
      //};
      this.pkt = L.marker([this.poslabel[1],this.poslabel[0]], {
        rotationAngle: this.angle,
        //rotationAngle: 0,
        draggable: true,
        icon: this.icon
      }).addTo(this.map);

      this.pkt.on("dragend",(e) => {
        console.log("hat sich bewegt");
        console.log(e);
        console.log(e.sourceTarget._startPos.x);
        console.log(e.sourceTarget._newPos.x);
        var diffx=e.sourceTarget._newPos.x-e.sourceTarget._startPos.x;
        var diffy=e.sourceTarget._newPos.y-e.sourceTarget._startPos.y;
        console.log(diffx);
        console.log(diffy);
        var angle = e.target.options.rotationAngle;
        var neux=Math.cos(angle/180*Math.PI)*this.laengepx+diffx;
        var neuy=Math.sin(angle/180*Math.PI)*this.laengepx+diffy;
        var angleneu = Math.atan2(neuy, neux)* (180 / Math.PI); // In radians
        if (angleneu <0){angleneu=angleneu+360};
        console.log("neuer Winkel");
        console.log(angleneu);
        this.marker.setRotationAngle(angleneu);
        this.pkt.setLatLng([this.poslabel[1],this.poslabel[0]]);
        //e.setLatLng([this.poslabel[1],this.poslabel[0]]);
        this.pkt.setRotationAngle(angleneu);
      });

      this.marker.on("dragend",(e) => {
        console.log("Marker Drag");
        console.log(e);
        var pos=e.target.getLatLng();
        console.log(pos);
        this.pkt.setLatLng(pos);
        //this.poslabel=e.target.getLatLng();
        this.poslabel[1]=pos.lat;
        this.poslabel[0]=pos.lng;
      });

    }
  }
};
</script>


<style>

.leaflet-container {
    background-color:rgba(255,0,0,0.0);
}

.sl {
  width: 200px;
  height: 30px;
  background-color: white;
  color: #bd924d;
  text-align: left;
  font-size: 16px;
  font-weight: bold;
}
</style>
