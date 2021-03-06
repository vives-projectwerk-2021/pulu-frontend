<template>
  <div id="container">
    <div id="mapContainer" class="mx-1;"></div>
  </div>
</template>

<script>
//import mapState from "vuex"
import "leaflet/dist/leaflet.css";
import L from "leaflet";
import { Icon } from "leaflet";
import { config } from "@/config.js";

export default {
  name: "Map",
  data() {
    return {
      center: [51.209348, 3.2246995],
      data: [],
      markerarray: [],
      map: null,
      loaded: false,
      icon: [],
      anchor: [],
      currentZoom: null,
      manIcon: "",
      maxBounds: [
        //south west
        [-90, -180],
        //north east
        [90, 180],
      ],
    };
  },
  methods: {
    setupLeafletMap() {
      this.map = L.map("mapContainer", { center: this.center, zoom: 14 });
      L.tileLayer(
        "https://api.mapbox.com/styles/v1/{username}/{style_id}/tiles/256/{z}/{x}/{y}@2x?access_token={access_token}",
        {
          minZoom: 2,
          maxZoom: 18,
          username: "arthur2",
          style_id: "ckxepad6y03n814n07tgt0jhy",
          access_token: config.VUE_APP_MAPBOX_TOKEN,
          projection: "naturalEarth",
        }
      ).addTo(this.map);

      this.map.setMaxBounds(this.maxBounds);
      delete Icon.Default.prototype._getIconUrl;
      Icon.Default.mergeOptions({
        iconRetinaUrl: require("leaflet/dist/images/marker-icon-2x.png"),
        iconUrl: require("leaflet/dist/images/marker-icon.png"),
        shadowUrl: require("leaflet/dist/images/marker-shadow.png"),
      });

      if (this.currentZoom > 9) {
        this.refreshLivemarker();
        this.icon = [16, 26];
        this.anchor = [8, 25];
      } else {
        this.refreshLivemarker();
        this.icon = [40, 50];
        this.anchor = [20, 49];
      }

      var LeafIcon = L.Icon.extend({
        options: {
          iconSize: this.icon,
          iconAnchor: this.anchor,
        },
      });

      this.manIcon = new LeafIcon({
        iconUrl: "img/map-man-marker.png",
      });

      this.map.on("zoomend", () => {
        this.currentZoom = this.map.getZoom();
      });

      this.addPoints();
    },
    refreshLivemarker() {
      var liveMarker;
      this.map
        .locate({
          setView: true,
        })
        .on("locationfound", (e) => {
          if (!liveMarker) {
            liveMarker = new L.marker(e.latlng, { icon: this.manIcon }).addTo(
              this.map
            );
          } else {
            liveMarker.setLatLng(e.latlng);
          }
          liveMarker.bindTooltip("You are here.");
          this.markerarray.push(liveMarker);
          this.map.fitBounds(
            L.latLngBounds(this.markerarray.map((marker) => marker.getLatLng()))
          );
        });
    },
    async addPoints() {
      await this.$store.dispatch("getAllSensors");
      this.$store.getters.devicelist.forEach((device) => {
        if (device.location.lat && device.location.long) {
          const marker = L.marker([device.location.lat, device.location.long])
            .addTo(this.map)
            .bindTooltip(device.devicename)
            .bindPopup(
              `<b>${device.devicename}</b><br><a href="${window.location.href}sensors/${device.deviceid}">See sensor data</a>`
            );

          this.markerarray.push(marker);
        }
      });
      if(this.markerarray){
        this.map.fitBounds(
          L.latLngBounds(this.markerarray.map((marker) => marker.getLatLng()))
        );
      }
    },
  },
  mounted() {
    this.setupLeafletMap();
  },
};
</script>

<style>
#mapContainer {
  height: 70vh;
  z-index: 0;
}
</style>