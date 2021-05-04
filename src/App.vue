
<template>
  <div id="q-app">
    <CardList />
    <CardList />
    <CardList />
    <router-view />
  </div>
</template>
<script>
import CardList from "./components/CardList.vue";
export default {
  name: 'App',
  components: {
  CardList
 }
}
</script>
<style>
body {
 background: #f6f9fc;
}
#app {
 display: flex;
 justify-content: space-evenly;
 align-items: center;
 font-family: "Roboto", sans-serif;
 -webkit-font-smoothing: antialiased;
 -moz-osx-font-smoothing: grayscale;
}
:root {
 font-size: 10px;
}
</style>
