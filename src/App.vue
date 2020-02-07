<template>
  <div id="app">
    <scroll-slide>
      <template v-slot="{ onReady }">
        <img
          src="https://assets.website-files.com/5b3fdf61d4f8bbc26c34275c/5cab56e31aed3617e272fdc8_lala.svg"
          @load="onReady"
        >
      </template>
    </scroll-slide>
  </div>
</template>

<script>
import ScrollSlide from './components/ScrollSlide';

export default {
    components: {
        ScrollSlide,
    },
};
</script>

<style>
body {
    margin: 0;
}

img {
    object-fit: contain;
    height: 80vh;
}
</style>
