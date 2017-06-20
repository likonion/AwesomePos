<template>
  <div id="app" class="app">
    <leftNav></leftNav>
    <router-view></router-view>
  </div>
</template>

<script>
  import leftNav from '@/components/common/leftNav'
  export default {
    name: 'app',
    components: {
      leftNav
    }
  }
</script>
<style lang="scss">
  @import "assets/css/common";
  .app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    height: 100%;
    display: flex;
    flex-flow: row nowrap;
  }
</style>
