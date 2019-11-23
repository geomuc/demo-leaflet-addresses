<template>
<v-container>
  <v-row no-gutters>
    <v-col cols="12" sm="12">
      <v-card class="pa-2" outlined tile height="500px" width="100%">
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
    longitude: 11.615,
    latitude: 48.1329,
    lastedited: {feature:""},
    labelmarker: L.latLng(48.13,11.615),
    angle: 30,
  }),
  mounted(){

      this.map = L.map('map5',{editable: true, doubleClickZoom: false}).setView([this.latitude, this.longitude], 16);
      /*this.tileLayer = L.tileLayer.wms('https://geoportal.muenchen.de/geoserver/gsm/wms?', {
          maxZoom: 30,
          layers: 'g_stadtkarte_gesamt',
          attribution: '&copy; <a href="https://www.muenchen.de/rathaus/Stadtverwaltung/Kommunalreferat/geodatenservice/geobasisdaten.html">GeodatenService München</a>'
      }).addTo(this.map);*/
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
          var angle=geojson.properties.angle;
          var strl=geojson.properties.streetlabel;
          var id = geojson.id;
          return L.marker(latlng, {
            rotationAngle: angle,
            draggable: false,
            icon: L.divIcon({
              iconSize: null,
              className: 'label',
              iconAnchor:   [0,0],
              html: '<div class="sl" id="dsl'+id+'"><input id="sl'+id+'" type="text" name="lname" value="'+strl+'"></div>',
            })
          });
        },
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
        var id=this.marker.feature.id;
        // Falls Label aktuell editierbar
        if (this.marker.editable == true)
        {
          this.beendeEdit(this.marker, "neu");
           this.lastedited=this.marker;
          this.lastpkt=this.pkt;
          this.lastangle=this.angle;
          this.lastpos=this.poslabel;
        // Falls Label aktuell nicht editierbar
        } else 
        {
          this.angle = this.marker.options.rotationAngle;
          //{
          //  this.beendeEdit(this.lastedited);
          //}
          if (this.lastedited.feature!="")
          {
            if (this.lastedited.feature.id != id) 
            {
              this.beendeEdit(this.lastedited, "alt");
            }
          }

          this.startEditing();
          document.getElementById("dsl"+id).style.backgroundColor="yellow";
          this.lastedited=this.marker;
          this.lastpkt=this.pkt;
          this.lastangle=this.angle;
          this.lastpos=this.poslabel;
          this.marker.editable = true;
          this.marker.dragging.enable();
        };
      });
  },
  methods: {
    startEditing: function(){
      this.pkt = L.marker([this.poslabel[1],this.poslabel[0]], {
        rotationAngle: this.angle,
        draggable: true,
        icon: this.icon
      }).addTo(this.map);

      this.pkt.on("dragend",(e) => {
        var diffx=e.sourceTarget._newPos.x-e.sourceTarget._startPos.x;
        var diffy=e.sourceTarget._newPos.y-e.sourceTarget._startPos.y;
        var angle = e.target.options.rotationAngle;
        var neux=Math.cos(angle/180*Math.PI)*this.laengepx+diffx;
        var neuy=Math.sin(angle/180*Math.PI)*this.laengepx+diffy;
        var angleneu = Math.atan2(neuy, neux)* (180 / Math.PI); // In radians
        if (angleneu <0){angleneu=angleneu+360};
        this.marker.setRotationAngle(angleneu);
        this.pkt.setLatLng([this.poslabel[1],this.poslabel[0]]);
        this.pkt.setRotationAngle(angleneu);
        this.angle=angleneu;
        this.lastangle=angleneu;
      });

      this.marker.on("dragend",(e) => {
        var pos=e.target.getLatLng();
        this.pkt.setLatLng(pos);
        this.poslabel[1]=pos.lat;
        this.poslabel[0]=pos.lng;
      });

    },
    beendeEdit: function(layer, altneu){
      if (altneu=="neu") {
        layer.feature.geometry.coordinates=[this.poslabel[0],this.poslabel[1]];
        layer.feature.properties.angle=this.angle;
        layer.options.rotationAngle=this.angle;
      } else {
        layer.feature.geometry.coordinates=[this.lastpos[0],this.lastpos[1]];
        layer.feature.properties.angle=this.lastangle;
        layer.options.rotationAngle=this.lastangle;
      }
      var sl=document.getElementById('sl'+layer.feature.id).value;
      layer.feature.properties.streetlabel=sl;
      document.getElementById("dsl"+layer.feature.id).style.backgroundColor="white";
      this.map.removeLayer(this.lastpkt);
      layer.editable=false;
      layer.options.draggable=false;
      this.streetlabels.updateFeature(layer.toGeoJSON(), function (error, response) {
        if (error) {
          console.log('error updating feature' + error.message);
        } else {
          console.log('Successfully updated feature ');
        }
      });
    }
  }
};
</script>

<style>

.leaflet-fade-anim .leaflet-tile,.leaflet-zoom-anim .leaflet-zoom-animated { will-change:auto !important; }

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
