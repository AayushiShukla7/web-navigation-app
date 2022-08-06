<template>
  <div class="h-screen relative">
    <div id="map" class="h-full z-[1]"></div>
  </div>
</template>

<script>
import leaflet from "leaflet";
import { onMounted, ref } from "vue";

export default {
  name: 'HomeView',
  components: {},
  setup() {
    let map;
    onMounted(() => {
      // init map
      map = leaflet.map('map').setView([28.538336, -81.379234], 10);

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

    //const coords = ref(null);
    const fetchCoords = ref(null);
    //const geoMarker = ref(null);

    const getGeoLocation = () => {
      fetchCoords.value = true;
      navigator.geolocation.getCurrentPosition(setCoords, getLocError);
    };

    const setCoords = (pos) => {
      //console.log(pos);

      // stop fetching coords
      fetchCoords.value = null;

      // set coords in session storage
      
    };

    const getLocError = (err) => {
      console.log(err);
    };
  }
}
</script>
