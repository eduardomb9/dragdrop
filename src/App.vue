<template>
  <v-app>

    <v-main class="container">
      <DragDropList/>
    </v-main>
  </v-app>
</template>

<script>
import DragDropList from './components/DragDropList';

export default {
  name: 'App',

  components: {
    DragDropList,
  },

  data: () => ({
    //
  }),
};
</script>
