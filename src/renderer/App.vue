<template>
  <div id="app">
    <router-view></router-view>
  </div>
</template>

<script>
  export default {
    name: 'flisto'
  }
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css?family=Manjari&display=swap');
body {
  margin: 0;
  font-family: 'Manjari', sans-serif;
}
#app {
    background: #000;

    .draggable-header {
      height: 22px;
      width: 100vw;
      background: #232323;
      -webkit-app-region: drag;
      flex-shrink: 0;
    }
  }
</style>
