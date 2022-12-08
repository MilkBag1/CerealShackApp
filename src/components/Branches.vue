<template>
  <div class="text-center q-mt-md">
    <div class="text-h4 text-primary">Branches</div>
    <div class="text-center">
      <q-avatar class="q-mb-md" size="100px" square>
        <img :src="`/img/icon.png`" />
      </q-avatar>
    </div>
    <div class="text-h6 text-secondary">Find 3 Closest Branches To:</div>
    <div>
      <q-input
        class="q-ma-lg text-h5"
        placeholder="enter address"
        id="address"
        v-model="state.address"
      />
      <br />
    </div>
    <q-btn label="Find 3" @click="genMap" class="q-mb-md" style="width: 30vw" />
    <div
      style="height: 50vh; width: 90vw; margin-left: 5vw; border: solid"
      ref="mapRef"
      v-show="state.showmap === true"
    ></div>
  </div>
</template>
<script>
import { ref, reactive } from "vue";
import { fetcher } from "../utils/apiutil";
export default {
  // eslint-disable-next-line vue/multi-word-component-names
  name: "Map",
  setup() {
    const mapRef = ref(null);
    let state = reactive({
      status: "",
      address: "",
      showmap: false,
      threeBranches: [],
    });
    const genMap = async () => {
      try {
        state.showmap = true;
        const tt = window.tt;
        let url = `https://api.tomtom.com/search/2/geocode/${state.address}.json?key=AGTkEAL7UinnkNlBIwyYNcGwpaomB3CB`;
        let response = await fetch(url);
        let payload = await response.json();
        let lat = payload.results[0].position.lat;
        let lon = payload.results[0].position.lon;
        payload = await fetcher(`branch/${lat}/${lon}`);
        state.threeBranches = payload;
        let map = tt.map({
          key: "AGTkEAL7UinnkNlBIwyYNcGwpaomB3CB",
          container: mapRef.value,
          source: "vector/1/basic-main",
          center: [lon, lat],
          zoom: 8,
        });
        map.addControl(new tt.FullscreenControl());
        map.addControl(new tt.NavigationControl());
        state.threeBranches.forEach((branch) => {
          let marker = new tt.Marker()
            .setLngLat([branch.longitude, branch.latitude])
            .addTo(map);
          let popupOffset = 25;
          let popup = new tt.Popup({ offset: popupOffset });
          popup.setHTML(
            `<div id="popup">Branch#: ${branch.id}</div><div>${
              branch.street
            }, ${branch.city}
              <br/>${branch.distance.toFixed(2)} km</div>`
          );
          marker.setPopup(popup);
        });
      } catch (err) {
        state.status = err.message;
      }
    };
    return {
      mapRef,
      state,
      genMap,
    };
  },
};
</script>
