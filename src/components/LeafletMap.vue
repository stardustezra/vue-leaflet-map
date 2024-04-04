<template>
  <div>
    <h3>An interactive leaflet map</h3>
    <div id="map" style="height: 90vh"></div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import "leaflet/dist/leaflet.css";
import * as L from "leaflet";

const initialMap = ref(null);

onMounted(() => {
  // Create the map
  initialMap.value = L.map("map").setView([51.505, -0.09], 13);

  // Add tile layer from OpenStreetMap
  L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
    attribution:
      "&copy; <a href='https://www.openstreetmap.org/copyright'>OpenStreetMap</a>",
  }).addTo(initialMap.value);

  // Add geolocation functionality
  if ("geolocation" in navigator) {
    navigator.geolocation.getCurrentPosition(
      (position) => {
        const { latitude, longitude } = position.coords;
        initialMap.value.setView([latitude, longitude], 13);
        L.marker([latitude, longitude])
          .addTo(initialMap.value)
          .bindPopup("You are here")
          .openPopup();

        // Show treasure area at current location for testing
        showTreasureArea(latitude, longitude);
      },
      (error) => {
        console.error("Error getting geolocation:", error);
      }
    );
  } else {
    console.error("Geolocation is not supported by your browser");
  }

  // Function to show treasure area at a specific location
  function showTreasureArea(lat, lng) {
    const treasureCircle = L.circle([lat, lng], {
      color: "blue",
      fillColor: "#add8e6",
      fillOpacity: 0.5,
      radius: 100, // Adjust radius as needed
    }).addTo(initialMap.value);
    treasureCircle.bindPopup("Treasure area!");
  }
});
</script>
