<template>
  <div 
    class="panel" 
    :class="'panel' + panel.id + (panel.fired ? ' panel_fired' : '') + (panel.failed ? ' panel_failed' : '')" 
    v-on:click="$emit('panelclick', panel.id)"
  > 
  </div>
</template>

<script>
export default {
  name: "Panel",
  props: {
    panel: Object
  }
};
</script>

<style scoped>
.panel {
  width: 200px;
  height: 200px;
  float: left;
  cursor: pointer;
}

.panel.panel_failed {
  background-color: black;
}

.top_right, .panel2 {
  background-color: darkred;
  border-top-right-radius: 100%;
  border-left: 5px solid #2c3e50;
  border-bottom: 5px solid #2c3e50;
}
.panel2.panel_fired {
  background-color: rgb(228, 47, 47);
}
.panel2.panel_failed {
  background-color: black;
}
.bottom_right, .panel4 {
  background-color: darkblue;
  border-bottom-right-radius: 100%;
  border-left: 5px solid #2c3e50;
  border-top: 5px solid #2c3e50;
}
.panel4.panel_fired {
  background-color: lightskyblue;
}

.bottom_left, .panel3 {
  background-color: goldenrod;
  border-bottom-left-radius: 100%;
  border-right: 5px solid #2c3e50;
  border-top: 5px solid #2c3e50;
}
.panel3.panel_fired {
  background-color: yellow;
}

.top_left, .panel1 {
  background-color: darkgreen;
  border-top-left-radius: 100%;
  border-right: 5px solid #2c3e50;
  border-bottom: 5px solid #2c3e50;
}
.panel1.panel_fired {
  background-color: rgb(136, 228, 61);
}
</style>
