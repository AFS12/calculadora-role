<template>
  <v-app>
    <v-app-bar
      app
      color="primary"
      dark
    >
      <div class="d-flex align-center">
        <h1>Calculadora de role</h1>
      </div>

      <v-spacer></v-spacer>
    </v-app-bar>

    <v-main>
      <Calculadora/>
    </v-main>
  </v-app>
</template>

<script>
import Calculadora from './components/Calculadora.vue';

export default {
  name: 'App',

  components: {
    Calculadora,
  },

  data: () => ({
    //
  }),
};
</script>
