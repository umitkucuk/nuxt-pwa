<template>
  <div class="shown-image">
    <img :src="images[shownImage]" />
  </div>
</template>

<script>
export default {
  name: 'ImageList',
  components: {},
  props: {
    images: {
      type: Array
    }
  },
  data() {
    return {
      shownImage: 0
    }
  },
  watch: {
    images(value) {
      this.shownImage = value.length - 1
    }
  },
  created() {},
  methods: {}
}
</script>

<style lang="scss" scoped>
.shown-image {
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  height: 100vh;

  img {
    width: auto;
    height: auto;
    max-height: 100vh;
  }
}
</style>
