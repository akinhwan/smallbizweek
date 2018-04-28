<template>
  <div>
    <div>
      <h2>Search and add a pin</h2>
      <label>
        <gmap-autocomplete
          @place_changed="setPlace">
        </gmap-autocomplete>
        <button @click="addMarker">Add</button>
      </label>
      <br/>

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
        :key="index"
        v-for="(m, index) in markers"
        :position="m.position"
        :icon="m.icon"
        @click="center=m.position"
      ></gmap-marker>
    </gmap-map>
  </div>
</template>
<script>
import axios from 'axios';
const validZips = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 15, 16, 17, 18, 19, 20, 24, 32, 36, 37, 52, 57, 64];//, 202, 317, 319, 373, 390, 510, 593];
let styledMap;
export default {
  name: "GoogleMap",
  data() {
    return {
      // default to Montreal to keep it simple
      // change this to whatever makes sense
      center: { lat: 38.8938, lng: 77.0310 },
      markers: [],
      places: [],
      currentPlace: null
    };
  },

  mounted() {
    this.geolocate();
    this.setStyles();
    this.loadKMLs();
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
        this.markers.push({ position: marker, icon: 'https://raw.githubusercontent.com/akinhwan/smallbizweek/master/src/assets/building_one.svg' });
        this.places.push(this.currentPlace);
        this.center = marker;
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
    loadKMLs(){
      let that = this;

      validZips.forEach ((zip)=>{
        const link = `https://raw.githubusercontent.com/akinhwan/smallbizweek/master/src/assets/zip${20000 + zip}.kml`;
        axios.get(link).then(()=>{
            this.$refs.mapRef.$mapPromise.then((map) => {
              that.kml = new google.maps.KmlLayer({
                url: link,
                visibility: 'visible',
                preserveViewport: true,
                map: map
              })
            })
          }
        ).catch(()=> console.log('not found'));
      });

    }
  }
};
</script>

<style scoped>
h2{
    margin: 0;
}
</style>
