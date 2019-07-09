<template>
  <div id="app">
    <swipe :autoplay="true" :width="width" :height="height" :items="items"></swipe>
  </div>
</template>

<script>
import swipe from "./swipe.vue";
import image1 from "./1.jpg";
import image2 from "./2.jpg";
export default {
  name: "app",
  components: {
    swipe
  },
  data() {
    return {
      width: window.innerWidth,
      height: 200,
      items: [image1, image2]
    };
  }
};
</script>

<style lang="less">
* {
  margin: 0;
  padding: 0;
}
</style>
