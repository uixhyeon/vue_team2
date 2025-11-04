<script setup>
import Banner from "./mains/Banner.vue";
import Faq from "./mains/Faq.vue";
import About from "./mains/About.vue";
import Info from "./mains/Info.vue";
import Serch from "./mains/Serch.vue";
import Review from "./mains/Review.vue";
</script>
<template>
  <img class="thumb" src="/public/images/mains/thumb.png" alt="썸네일">
  <Banner />
  <About />
  <Review />
  <Info />
  <Serch />
  <Faq />
</template>
<style scoped>
.thumb{
  display: none;
}
</style>
