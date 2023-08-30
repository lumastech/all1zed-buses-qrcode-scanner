<template>
  <v-app>
    <v-app-bar
      app
      color="grey"
      dark
    >
      <div class="d-flex align-center">
        <v-img src="img/icons/full-logo.png" width="150"></v-img>
      </div>

      <v-spacer></v-spacer>

      
    </v-app-bar>

    <v-main>
      <router-view/>
    </v-main>
  </v-app>
</template>

<script>

export default {
  name: 'App',

  data: () => ({
    //
  }),
};
</script>
