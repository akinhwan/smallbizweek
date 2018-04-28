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
        @click="center=m.position"
      ></gmap-marker>
    </gmap-map>
  </div>
</template>
<script>
import axios from 'axios';
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
        this.markers.push({ position: marker });
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
    loadKMLs(){
      let that = this;
      for(let i = 1; i <= 55; i++){
        const link = `https://raw.githubusercontent.com/akinhwan/smallbizweek/master/src/assets/zip${20000 + i}.kml`;
        console.log(link);
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
      }

    }
  }
};
</script>

<style scoped>
h2{
    margin: 0;
}
</style>
