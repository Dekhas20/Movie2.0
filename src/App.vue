<template>
  <div id="app">
    <RouterView />
  </div>
</template>

<script>
import { RouterView } from "vue-router";

export default {
  components: {},
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: Arial, Helvetica, sans-serif;
}

body {
  margin-bottom: 20px;
}

.title {
  width: 100%;
  text-align: center;
  margin: 20px 0;
}
</style>
