<template>
  <div id="app">
    <resize-box/>
  </div>
</template>

<script>
import ResizeBox from '@/components/resize-box/index.vue'
export default {
  name: 'App',
  components: {
    ResizeBox
  }
}
</script>

<style>
*{
  margin: 0;
  padding: 0;
}
</style>
