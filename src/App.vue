<template>
  <Simon />
</template>

<script>
import Simon from "@/components/Simon.vue";

export default {
  name: "App",
  components: {
    Simon
  }
};
</script>

<style>
#app {
  font-family: Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  height: 100vh;
  width: 100%;
}
</style>
