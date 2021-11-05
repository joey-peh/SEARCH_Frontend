<template>
  <v-app>
    <v-main>
      <v-container fluid>
        <SearchRepository />
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import SearchRepository from "./components/SearchRepository";

export default {
  name: "App",

  components: {
    SearchRepository,
  },

  data: () => ({
    //
  }),
};
</script>
