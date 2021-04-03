<template>
  <v-app>
    <v-main>
      <div id="app" class="app">
          <MapContainer />
      </div>
    </v-main>
  </v-app>
</template>

<script>
import MapContainer from "./components/MapContainer";

export default {
  name: "App",

  components: {
    MapContainer,
  },

  data: () => ({
    //
  }),
};
</script>

<style>
.app {
  padding: 40px 20px;
  max-width: 1000px;
  margin: 0 auto;
}
</style>