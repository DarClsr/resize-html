<template>
  <div class="sizeBox" :style="defaultStyle">
    <resize-div :maxHeight="currentMaxHeight(item)" :gutters="gutters" :width="item.width" :height="item.height" :maxWidth="currentMaxWidth(item)" :classId="randomId(index)" v-for="(item,index) in grids" :key="index" :leftNum="item.left" :topNum="item.top" />
  </div>
</template>

<script>
import ResizeDiv from "@/components/resize-div/index.vue"
export default {
  data () {
    return {
      boxWidth: 800,
      boxHeight: 800,
      gutters: 10,

      grids: [
        {
          id: Math.random() * 10000,
          left: 0,
          width: 300,
          height: 590,
          top: 0,
        },

        // {
        //   id: Math.random() * 10000,
        //   left: 500,
        //   width: 300,
        //   height: 590,
        //   top: 0,
        // },
        // {
        //   left: 0,
        //   id: Math.random() * 10000,
        //   top: 600,
        //   width: 800,
        //   height: 200,
        // }
      ]
    }
  },
  computed: {
    currentMaxWidth () {
      return (item) => {
        const blocks = this.grids.filter(v => v.top == item.top && v.id != item.id);
        const width = blocks.reduce((total, cur) => {
          return total += cur.width;
        }, 0)
        console.log(this.boxWidth - width, "lllll");
        console.log(item)
        return this.boxWidth - width;
      }
    },
    currentMaxHeight () {
      return (item) => {
        const blocks = this.grids.filter(v => v.left == item.left && v.id != item.id);
        const width = blocks.reduce((total, cur) => {
          return total += cur.height;
        }, 0)
        return this.boxHeight - width;
      }
    },
    randomId () {
      return (i) => {
        return Math.floor(Math.random() * 10000000 * (i + 1));
      }
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
  background-color: #f5f5f5;
  margin: 0 auto;
  position: relative;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.2);
}
</style>