<template>
  <div class="hello">
    <div class="image-container">
      <img :src="image">
    </div>
    <div>
      <h1>WeeaBingo!</h1>
      <input type="text" placeholder="nimi tähän">
    </div>
    <div class="image-container">
      <img class="img-reverse" :src="image">
    </div>
  </div>
</template>

<script>
export default {
  name: "Header",
  props: {
    image: String
  },
  data: function() {
    return {
   };
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.hello {
  display: flex;
  justify-content: space-between;
  max-width: 1000px;
  margin: 0 auto;
  margin-bottom: 20px;
  align-items: center;
}

img {
  width: 100%;
  max-width: 200px;
  display: block;
  height: auto;
}

.img-reverse {
  transform: rotateY(180deg);
}

h1 {
  font-size: 30px;
}
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

input {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  border: none;
  border-bottom: 2px solid black;
  font-size: 25px;
  width: 250px;
  text-align: center;
  font-weight: bold;
  border-radius: 0;
  &:focus {
    outline: 1px solid black;
  }
}
</style>
