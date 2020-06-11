<template>
  <div id="app">
    <Bingo/>
    <Footer/>
  </div>
</template>

<script>
import Bingo from "./components/Bingo";
import Footer from "./components/Footer";

export default {
  name: "App",
  components: {
    Bingo,
    Footer
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

* {
  box-sizing: border-box;
}
</style>
