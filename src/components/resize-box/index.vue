<template>
  <div class="sizeBox" :style="defaultStyle">

    <resize-div :maxWidth="currentMax" />
  </div>
</template>

<script>
import ResizeDiv from "@/components/resize-div/index.vue"
export default {
  data () {
    return {
      boxWidth: 800,
      boxHeight: 800,
    }
  },
  computed: {
    currentMax () {
      return this.boxWidth
    },
    defaultStyle () {
      return `width:${this.boxWidth};height:${this.boxHeight}`;
    }
  },
  components: {
    ResizeDiv
  }
}
</script>
<style scoped>
.sizeBox {
  width: 800px;
  height: 800px;
  display: grid;
  background-color: #f5f5f5;
  grid-template-columns: 100%;
  grid-template-rows: 100%;
  margin: 0 auto;
  position: relative;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.2);
}
</style>