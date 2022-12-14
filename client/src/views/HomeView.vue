<template>
  <div class="h-screen relative">
    <GeoErrorModal 
    v-if="geoError" 
    :geoErrorMsg="geoErrorMsg" 
    @closeGeoError="closeGeoError" />

    <MapFeatures />

    <div id="map" class="h-full z-[1]"></div>
  </div>
</template>

<script>
import leaflet from "leaflet";
import { onMounted, ref } from "vue";
import GeoErrorModal from '@/components/GeoErrorModal.vue';
import MapFeatures from '@/components/MapFeatures.vue';

export default {
  name: 'HomeView',
  components: {
    GeoErrorModal,
    MapFeatures
  },
  setup() {
    let map;
    onMounted(() => {
      // init map
      map = leaflet.map('map').setView([28.094080, 76.991920], 10);

      // add tile layer 
      leaflet.tileLayer(
          `https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=${process.env.VUE_APP_API_KEY}`,
          {
            maxZoom: 18,
            id: "mapbox/streets-v11",
            tileSize: 512,
            zoomOffset: -1,
            accessToken: process.env.VUE_APP_API_KEY,
          }
        )
        .addTo(map);

        getGeoLocation();
    }) 

    const coords = ref(null);
    const fetchCoords = ref(null);
    const geoMarker = ref(null);
    const geoError = ref(null);
    const geoErrorMsg = ref('testing v-bind on modal');

    const getGeoLocation = () => {
      // check session storage for coords
      if(sessionStorage.getItem('coords')) {
        coords.value = JSON.parse(sessionStorage.getItem('coords'));
        plotGeoLocation(coords.value);
        return;
      }

      fetchCoords.value = true;
      navigator.geolocation.getCurrentPosition(setCoords, getLocError);
    };

    const setCoords = (pos) => {
      console.log(pos);

      // stop fetching coords
      fetchCoords.value = null;

      // set coords in session storage
      const setSessionCoords = {
        lat: pos.coords.latitude,
        lng: pos.coords.longitude,
      }

      sessionStorage.setItem('coords', JSON.stringify(setSessionCoords));

      // set ref coords value
      coords.value = setSessionCoords;

      plotGeoLocation(coords.value);
    };

    const getLocError = (err) => {
      fetchCoords.value = null;
      geoError.value = true;
      geoErrorMsg.value = err.message;
    };

    const plotGeoLocation = (coords) => {
      // create custom marker
      const customMarker = leaflet.icon({
        iconUrl: require('../assets/map-marker-red.svg'),
        iconSize: [35, 35],
      });

      //create new marker with coords and custom icon
      geoMarker.value = leaflet
      .marker([coords.lat, coords.lng], {icon: customMarker})
      .addTo(map);

      // set map view to current location
      map.setView([coords.lat, coords.lng], 10);
    };

    const closeGeoError = () => {
      geoError.value = null;
      geoErrorMsg.value = null;
    };

    return { coords, geoMarker, closeGeoError, geoError, geoErrorMsg };
  },
};
</script>
