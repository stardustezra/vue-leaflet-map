<template>
  <div id="map" style="height: 400px"></div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import "leaflet/dist/leaflet.css";
import * as L from "leaflet";

const initialMap = ref(null);

onMounted(() => {
  // Create the map
  initialMap.value = L.map("map").setView([0, 0], 16); // Centered at [0, 0] with a zoom level of 16

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
        initialMap.value.setView([latitude, longitude], 16); // Zoom in with a zoom level of 16
        L.marker([latitude, longitude])
          .addTo(initialMap.value)
          .bindPopup("You are here", { maxWidth: "auto" })
          .openPopup();

        // Calculate coordinates for treasure area 50 meters away
        const { lat: newLat, lng: newLng } = calculateOffsetCoordinates(
          latitude,
          longitude,
          50
        );

        // Show treasure area at calculated coordinates
        showTreasureArea(newLat, newLng);
      },
      (error) => {
        console.error("Error getting geolocation:", error);
      }
    );
  } else {
    console.error("Geolocation is not supported by your browser");
  }

  // Function to calculate coordinates offset by distance (in meters)
  function calculateOffsetCoordinates(lat, lng, distance) {
    const earthRadius = 6378137; // Earth's radius in meters
    const latOffset = (distance / earthRadius) * (180 / Math.PI);
    const lngOffset =
      ((distance / earthRadius) * (180 / Math.PI)) /
      Math.cos((lat * Math.PI) / 180);
    const newLat = lat + latOffset;
    const newLng = lng + lngOffset;
    return { lat: newLat, lng: newLng };
  }

  // Function to show treasure area at a specific location
  function showTreasureArea(lat, lng) {
    const treasureCenter = [lat, lng];
    const treasureAreaCircle = L.circle(treasureCenter, {
      color: "blue",
      fillColor: "#add8e6",
      fillOpacity: 0.5,
      radius: 30, // Adjust radius as needed
    }).addTo(initialMap.value);
    treasureAreaCircle.bindPopup("Treasure area!");

    // Define the point within the area for the popup
    const popupPoint = L.latLng(lat, lng); // Adjust as needed

    // Bind popup to the specific point inside the area
    L.popup({
      maxHeight: "auto",
      maxWidth: "auto",
    })
      .setLatLng(popupPoint)
      .setContent("<div class='popup-content'>Treasure here!</div>")
      .openOn(initialMap.value);
  }
});
</script>
