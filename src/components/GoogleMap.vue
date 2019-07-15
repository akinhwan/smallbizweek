<template>
  <div>
    <div id="topbar">
      <div id="wordmark">
        <img id="logo" src="../assets/logo.png" alt="">
        <h2>DataBridge</h2>
      </div>
      <label>
        <gmap-autocomplete
          id="autocomplete"
          @place_changed="setPlace">
        </gmap-autocomplete>
        <button @click="addMarker">Add</button>
      </label>

    <form action="">
        <input type="checkbox" id="checkbox" v-model="isChecked" @click= "toggleKMLs">
        <label for="checkbox">Zipcode Layer</label>
        &nbsp;&nbsp;
        <input type="checkbox" id="checkbox-zone" @click= "toggleZones">
        <label for="checkbox-zone">HUBZone Layer</label>
    </form>
    </div>
    <div id="legend">
      <div class="circle"></div><p></p>
    </div>

    <gmap-map
      ref = "mapRef"
      :center="center"
      :zoom="15"
      style="width:100%;  height: 91vh;"
      map-type-id="roadmap"
    >
      <gmap-marker
        ref = "markers"
        :key="index"
        v-for="(m, index) in markers"
        :position="m.position"
        :icon="m.icon"
        :animation = "m.animation"
      >

      </gmap-marker>
    </gmap-map>
  </div>
</template>

<script>
import axios from 'axios';
import {merchants} from './merchants.js';
const validZips = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 15, 16, 17, 18, 19, 20, 24, 32];//, 36, 37, 52, 57, 64, 202, 317, 319, 373, 390, 510, 593];
let styledMap;
let visible = false;
let visibleHUB = false;
let HUBs = [];
let kmls = [];
let buildingCount = 0;
let infoWindows = [];

export default {
  name: "GoogleMap",
  data() {
    return {
      // default to Montreal to keep it simple
      // change this to whatever makes sense
      center: { lat: 38.8938, lng: 77.0310 },
      markers: [],
      places: [],
      merchantMeasurements: [],
      currentPlace: null,
      isChecked: '',
      isOpened: false,
      errors: [],
      merchants: merchants
    };
  },
  created(){
    var baseURL = 'http://ec2-34-238-116-47.compute-1.amazonaws.com'
    var zipcodes= ['20004', '20005', '20001', '20024']

    //SEARCH BY ZIP CODE
    zipcodes.forEach((zc) => {
      axios.get(`${baseURL}/api/merchant/searchByZipCode/${zc}`, {
      headers:{
        "Access-Control-Allow-Origin": "http://localhost:8080",
        "Content-Type": "application/json",
        "charset": "utf-8"
        }
      })
      .then(response => {
        // JSON responses are automatically parsed.
        // this.merchantMeasurements = response.data
        var actf = response.data[0].response.responseData[1].avgCardTranFreq;
        console.log(response.data);
        // console.log(response.data.response.responseData[1].avgCardTranFreq);
        this.merchantMeasurements.push(actf);
      })
      .catch(e => {
        this.errors.push(e)
      })
    })

    //GET ALL MERCHANT DATA
    axios.get(`${baseURL}/api/merchant/getAllMerchantData`, {
      headers:{
        "Access-Control-Allow-Origin": "http://localhost:8080",
        "Content-Type": "application/json",
        "charset": "utf-8"
      }
    })
    .then(resp => {
      console.log("All Merchant Data", resp.data);
    })
    .catch(e => {
      this.errors.push(e);
    })

  },
  mounted() {
    var merchants20004 = merchants.filter(mch => mch.address.includes('20004'));
    var merchants20005 = merchants.filter(mch => mch.address.includes('20005'));
    var merchants20001 = merchants.filter(mch => mch.address.includes('20001'));
    var merchants20024 = merchants.filter(mch => mch.address.includes('20024'));

    const m4_shuffle = merchants20004.sort(() => .5 - Math.random());
    let m4_selected =m4_shuffle.slice(0,10) ;
    const m5_shuffle = merchants20005.sort(() => .5 - Math.random());
    let m5_selected =m5_shuffle.slice(0,10) ;
    const m1_shuffle = merchants20001.sort(() => .5 - Math.random());
    let m1_selected =m1_shuffle.slice(0,10) ;
    const m24_shuffle = merchants20024.sort(() => .5 - Math.random());
    let m24_selected =m24_shuffle.slice(0,10) ;

    // console.log("ZIP 4", merchants20004);
    // console.log("ZIP 5", merchants20005);
    // console.log("ZIP 1", merchants20001);
    // console.log("ZIP 24", merchants20024);

    this.$refs.mapRef.$mapPromise.then(()=>{
      this.geolocate();
      this.setStyles();
      this.drawHubZones();

      // this.infoWindows();
      for (var x = 0, ln = merchants20004.length; x < ln; x++) {
        setTimeout((y) => {    
          this.findLatLong(merchants20004[y]);
        }, x * 500, x); 

      for (var x = 0, ln = m4_selected.length; x < ln; x++) {
        setTimeout((y) => {
          this.findLatLong(m4_selected[y]);
        }, x * 1000, x);
      }
      for (var x = 0, ln = m5_selected.length; x < ln; x++) {
        setTimeout((y) => {
          this.findLatLong(m5_selected[y]);
        }, x * 1000, x);
      }
      for (var x = 0, ln = m1_selected.length; x < ln; x++) {
        setTimeout((y) => {
          this.findLatLong(m1_selected[y]);
        }, x * 1000, x);
      }
      for (var x = 0, ln = m24_selected.length; x < ln; x++) {
        setTimeout((y) => {
          this.findLatLong(m24_selected[y]);
        }, x * 1000, x);
      }

    });

  },

  methods: {
    // receives a place object via the autocomplete component
    generateName(){
      return merchants[Math.floor(Math.random() * (2358 - 1) + 1)].name;
    },
    generateContact(){
      return merchants[Math.floor(Math.random() * (2358 - 1) + 1)].contact;
    },
    categoryGroup(){
      var categories = ['Gambling', 'Liquor Stores', 'Adult Entertainment', 'Bail Bonds', 'Pawn Shops', 'Durable Goods', 'Miscellaneous', 'Fuel', 'Supermarket', 'Health Care', 'Travel', 'Dining', 'Entertainment', 'General Retail', 'Drug Store', 'Electronic Stores', 'Housing', 'Automotive', 'Coffee Stores'];
      return categories[Math.floor(Math.random() * (19 - 1) + 1)];
    },
    setPlace(place) {
      this.currentPlace = place;
    },
    // infowWindows(){
    //   this.$refs.mapRef.$mapPromise.then((map) => {
        //     var contentString = `<div id="content">
        //     <div id="siteNotice">
        //     </div>
        //     <h1 id="firstHeading" class="firstHeading">Uluru</h1>
        //     <div id="bodyContent">
        //     <p><b>Uluru</b>, also referred to as <b>Ayers Rock</b>, is a large  
        //     sandstone rock formation in the southern part of the 
        //     Northern Territory, central Australia. It lies 335&#160;km (208&#160;mi) 
        //     south west of the nearest large town, Alice Springs; 450&#160;km 
        //     (280&#160;mi) by road. Kata Tjuta and Uluru are the two major 
        //     features of the Uluru - Kata Tjuta National Park. Uluru is 
        //     sacred to the Pitjantjatjara and Yankunytjatjara, the 
        //     Aboriginal people of the area. It has many springs, waterholes, 
        //     rock caves and ancient paintings. Uluru is listed as a World 
        //     Heritage Site.</p>
        //     <p>Attribution: Uluru, <a href="https://en.wikipedia.org/w/index.php?title=Uluru&oldid=297882194">
        //     https://en.wikipedia.org/w/index.php?title=Uluru</a> 
        //     (last visited June 22, 2009).</p>
        //     </div>
        //     </div>`;

        // var infowindow = new google.maps.InfoWindow({
        //   content: contentString
        // });

        // var marker = new google.maps.Marker({
        //   position: uluru,
        //   map: map,
        //   title: 'Uluru (Ayers Rock)'
        // });

        // marker.addListener('click', function() {
        //   infowindow.open(map, marker);
        // });
    //   })
    // },
    addMarker(scriptMarker) {
      let marker;
      if (scriptMarker.type !== 'click'){
        marker = scriptMarker;
      } else if (this.currentPlace) {
        marker = {
          lat: this.currentPlace.geometry.location.lat(),
          lng: this.currentPlace.geometry.location.lng()
        };
      }
      const building = ['one', 'two', 'three'];
      this.markers.push({ position: marker, icon: `https://raw.githubusercontent.com/akinhwan/smallbizweek/master/src/assets/building_${building[buildingCount]}.png`, animation: google.maps.Animation.DROP });

      //marker infoWindow event Listener
      // var contentString=`<div>helloooooooooooo</div>`
      // var infowindow = new google.maps.InfoWindow({
      //     content: contentString
      //   });
      // this.markers.addListener('hover', (markerEvent)=>{
      //   infowindow.open(this.$refs.mapRef, this.marker);
      // });
      //end infoWindow

      if (this.currentPlace) {
        this.places.push(this.currentPlace);
      }

      infoWindows.push(new google.maps.InfoWindow({
        content: `          <p>${this.generateName()}</p>
                  <p>${this.generateContact()}</p>
                  <p>${ this.categoryGroup()}</p>`
      }));

      this.$refs.mapRef.$mapPromise.then((map) => {
        const that = this;
        let last = that.markers.length -1;
        that.$refs.markers[that.markers.length-1].$markerPromise.then((marker)=>{
          marker.addListener('click', ()=>{
            infoWindows[last].open(this.$refs.mapRef.$mapObject, marker);
            });
          });
        });
      this.currentPlace = null;
      ++buildingCount;
      if (buildingCount > 2) buildingCount = 0;
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
            kmls[kmls.length - 1].addListener('click', (kmlEvent)=>{
              kmlEvent.featureData.infoWindowHtml = `<div style="font-family: Roboto,Arial,sans-serif; font-size: small"><div style="font-weight: 500; font-size: medium; margin-bottom: 0em"></div><div><center>
              <table><tbody><tr><th colspan="2" align="center"><em>VISA Data</em></th></tr><tr bgcolor="#E3E3F3">
              <th>Zip code</th>
              <td>${20000 + zip}</td>
              </tr>
              <tr bgcolor="">
              <th>Avg Card Transaction Freq</th>
              <td>${this.merchantMeasurements[Math.floor(Math.random()*4)]}</td>
              </tr>
              <tr bgcolor="#E3E3F3">
              <th>Sales Volume Growth MoM</th>
              <td>${this.merchantMeasurements[Math.floor(Math.random()*4)] * 3}</td>
              </tr>
              <tr >
              <th>Total Spend Percentage</th>
              <td>${parseFloat(this.merchantMeasurements[Math.floor(Math.random()*4)] * .1).toFixed(2) + '%'}</td>
              </tr>
              </tbody></table></center></div></div>`;
            });
          })
        }
      ).catch(()=> console.log('not found'));
    });
  },
  hideKMLs(){
    kmls.forEach((kml)=>kml.setMap(null));
  },
  findLatLong({address}){
    this.$refs.mapRef.$mapPromise.then((map) => {
      let geocoder = new google.maps.Geocoder();
      geocoder.geocode({address: address}, (result, status) => {
        if (status === 'OK'){
          this.addMarker(result[0].geometry.location);
        } else {
          console.log(`Failed: ${status}`)
        }
      });
    });
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
#topbar{
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-around;
  padding-top: 10px;
}
#topbar > label{
      width: 35%;
    display: flex;

}

#topbar > label > input{
  width: 85%;
}

#logo{
  width: 7vmin;
}

#wordmark{
  display:flex;
  flex-direction: row;
  align-items: center;
}
</style>
