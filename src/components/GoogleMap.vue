<template>
  <div>
    <div>
      <h2>Search and add a pin</h2>
      <label>
        <gmap-autocomplete
          id="autocomplete"
          @place_changed="setPlace">
        </gmap-autocomplete>
        <button @click="addMarker">Add</button>
      </label>

      <br/>
    <form action="">
        <input type="checkbox" id="checkbox" v-model="isChecked" @click= "toggleKMLs">
        <label for="checkbox">Toggle Zipcode Layer</label>
        <input type="checkbox" id="checkbox-zone" @click= "toggleZones">
        <label for="checkbox-zone">Toggle HUB Layer</label>
    </form>
    </div>
    <br>
    <gmap-map
      ref = "mapRef"
      :center="center"
      :zoom="15"
      style="width:100%;  height: 700px;"
      map-type-id="roadmap"
    >
      <gmap-marker
        ref = "markers"
        :key="index"
        v-for="(m, index) in markers"
        :position="m.position"
        :icon="m.icon"
        :animation = "m.animation"
        @click="center=m.position"
      ></gmap-marker>
    </gmap-map>
  </div>
</template>
<script>
import axios from 'axios';
const validZips = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 15, 16, 17, 18, 19, 20, 24, 32, 36, 37, 52, 57, 64];//, 202, 317, 319, 373, 390, 510, 593];
let styledMap;
let visible = false;
let visibleHUB = false;
let HUBs = [];
let kmls = [];
let buildingCount = 0;
export default {
  name: "GoogleMap",
  data() {
    return {
      // default to Montreal to keep it simple
      // change this to whatever makes sense
      center: { lat: 38.8938, lng: 77.0310 },
      markers: [],
      places: [],
      currentPlace: null,
      isChecked: ''
    };
  },

  mounted() {
    this.geolocate();
    this.setStyles();
    this.drawHubZones();
  },

  methods: {
    // receives a place object via the autocomplete component

    setPlace(place) {
      this.currentPlace = place;
    },
    addMarker() {
      if (this.currentPlace) {
        const marker = {
          lat: this.currentPlace.geometry.location.lat(),
          lng: this.currentPlace.geometry.location.lng()
        };
        const building = ['one', 'two', 'three'];
        this.markers.push({ position: marker, icon: `https://raw.githubusercontent.com/akinhwan/smallbizweek/master/src/assets/building_${building[buildingCount]}.png`, animation: google.maps.Animation.DROP });
        this.center = marker;
        ++buildingCount;
        if (buildingCount > 2) buildingCount = 0;
        this.places.push(this.currentPlace);
        this.currentPlace = null;
      }
    },
    geolocate: function() {
      navigator.geolocation.getCurrentPosition(position => {
        this.center = {
          lat: position.coords.latitude,
          lng: position.coords.longitude
        };
      });
    },

    setStyles() {
      this.$refs.mapRef.$mapPromise.then((map) => {
        styledMap = new google.maps.StyledMapType(
          [{
            "featureType": "administrative",
            "elementType": "geometry",
            "stylers": [
              {
                "visibility": "off"
              }
            ]
          },
          {
            "featureType": "administrative.land_parcel",
            "stylers": [
              {
                "visibility": "off"
              }
            ]
          },
          {
            "featureType": "administrative.neighborhood",
            "stylers": [
              {
                "visibility": "off"
              }
            ]
          },
          {
            "featureType": "poi",
            "stylers": [
              {
                "visibility": "off"
              }
            ]
          },
          {
            "featureType": "road",
            "elementType": "labels",
            "stylers": [
              {
                "visibility": "off"
              }
            ]
          },
          {
            "featureType": "road",
            "elementType": "labels.icon",
            "stylers": [
              {
                "visibility": "off"
              }
            ]
          },
          {
            "featureType": "road.arterial",
            "elementType": "labels",
            "stylers": [
              {
                "visibility": "off"
              }
            ]
          },
          {
            "featureType": "road.highway",
            "elementType": "labels",
            "stylers": [
              {
                "visibility": "off"
              }
            ]
          },
          {
            "featureType": "road.local",
            "stylers": [
              {
                "visibility": "off"
              }
            ]
          },
          {
            "featureType": "transit",
            "stylers": [
              {
                "visibility": "off"
              }
            ]
          },
          {
            "featureType": "water",
            "elementType": "labels.text",
            "stylers": [
              {
                "visibility": "off"
              }
            ]
          }]
        );
        map.mapTypes.set('styled_map', styledMap);
        map.setMapTypeId('styled_map');
      });
    },
    toggleKMLs(){
      if (visible){
        this.hideKMLs();
      } else {
        this.loadKMLs();
      }
      visible = !visible;
    },
    drawHubZones(){
        this.$refs.mapRef.$mapPromise.then((map) => {
            var censusTractCoords = [
                {lat: 38.896012, lng: -77.047979},
                {lat: 38.892046, lng: -77.040748},
                {lat: 38.892079, lng: -77.039482},
                {lat: 38.898793, lng: -77.039460},
                {lat: 38.902249, lng: -77.049355},
                {lat: 38.901991, lng: -77.050110},
                {lat: 38.896023, lng: -77.050114}
            ];

            var downtownRedesigCoords = [
                {lat: 38.902279, lng: -77.049364},
                {lat: 38.902963, lng: -77.049708},
                {lat: 38.909143, lng: -77.043903},
                {lat: 38.909381, lng: -77.042712},
                {lat: 38.907342, lng: -77.036866},
                {lat: 38.907017, lng: -77.036528},
                {lat: 38.900199, lng: -77.036555},
                {lat: 38.900199, lng: -77.037950},
                {lat: 38.898742, lng: -77.037961},
                {lat: 38.898801, lng: -77.039452},
            ]

            var vernonRedesigCoords = [
                {lat: 38.902946, lng: -77.021923},
                {lat: 38.905357, lng: -77.014777},
                {lat: 38.908559, lng: -77.015905},
                {lat: 38.908551, lng: -77.021903}
            ]
            var vernonCensusCoords = [
                {lat: 38.902936, lng: -77.021912},
                {lat: 38.902519, lng: -77.021923},
                {lat: 38.902516, lng: -77.009062},
                {lat: 38.907335, lng: -77.009054}
            ]
            var eastendRedesigCoords = [
                {lat: 38.902513, lng: -77.021910},
                {lat: 38.902104, lng: -77.021910},
                {lat: 38.901123, lng: -77.018933},
                {lat: 38.894719, lng: -77.018939},
                {lat: 38.894402, lng: -77.019893},
                {lat: 38.892761, lng: -77.019899},
                {lat: 38.891778, lng: -77.016382},
                {lat: 38.892083, lng: -77.013168},
                {lat: 38.893942, lng: -77.010656},
                {lat: 38.894794, lng: -77.010946},
                {lat: 38.894781, lng: -77.009487},
                {lat: 38.895090, lng: -77.009058},
                {lat: 38.902514, lng: -77.009065}
            ]

            var censusTract = new google.maps.Polygon({
                paths: censusTractCoords,
                strokeColor: '#FF0000',
                strokeOpacity: 0.8,
                strokeWeight: 2,
                fillColor: '#FF0000',
                fillOpacity: 0.35
            });
            let downtownRedesig = new google.maps.Polygon({
                paths: downtownRedesigCoords,
                strokeColor: '#FF0000',
                strokeOpacity: 0.8,
                strokeWeight: 2,
                fillColor: '#ff9696',
                fillOpacity: 0.35
            });
            let vernonRedesig = new google.maps.Polygon({
                paths: vernonRedesigCoords,
                strokeColor: '#FF0000',
                strokeOpacity: 0.8,
                strokeWeight: 2,
                fillColor: '#ff9696',
                fillOpacity: 0.35
            });
            let vernonCensus = new google.maps.Polygon({
                paths: vernonCensusCoords,
                strokeColor: '#FF0000',
                strokeOpacity: 0.8,
                strokeWeight: 2,
                fillColor: '#FF0000',
                fillOpacity: 0.35
            });
            let eastendRedesig = new google.maps.Polygon({
                paths: eastendRedesigCoords,
                strokeColor: '#FF0000',
                strokeOpacity: 0.8,
                strokeWeight: 2,
                fillColor: '#ff9696',
                fillOpacity: 0.35
            });
            HUBs.push(censusTract, downtownRedesig,vernonRedesig,vernonCensus,eastendRedesig);
        })
    },
    toggleZones(){
      if (visibleHUB){
        HUBs.forEach((hub)=> hub.setMap(null))
      } else {
        HUBs.forEach((hub)=> hub.setMap(this.$refs.mapRef.$mapObject));
      }
      visibleHUB = !visibleHUB
    },
  loadKMLs(){
    if(kmls.length > 0){
      kmls.forEach((kml)=>kml.setMap(this.$refs.mapRef.$mapObject));
      return;
    }
    validZips.forEach ((zip)=>{
      const link = `https://raw.githubusercontent.com/akinhwan/smallbizweek/master/src/assets/zip${20000 + zip}.kml`;
      axios.get(link).then(()=>{
          this.$refs.mapRef.$mapPromise.then((map) => {
            kmls.push (new google.maps.KmlLayer({
              url: link,
              visibility: visible ? 'visible': 'off',
              preserveViewport: true,
              map: map
            }));
          })
        }
      ).catch(()=> console.log('not found'));
    });
  },
  hideKMLs(){
    kmls.forEach((kml)=>kml.setMap(null));
  }
}
};
</script>

<style scoped>
h2{
    margin: 0;
}
#autocomplete{
    width: 35%;
}
</style>
