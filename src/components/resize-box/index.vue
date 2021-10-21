<template>
  <div class="sizeBox" :style="defaultStyle">
    <resize-div :maxHeight="currentMaxHeight(item)" :gutters="gutters" :width="item.width" :height="item.height" :maxWidth="currentMaxWidth(item)" :classId="item.id" v-for="(item,index) in grids" :key="index" :leftNum="item.left" :topNum="item.top" @change="setResize" />
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
          left: 0,
          width: 300,
          height: 590,
          top: 0,
        },

        {
          left: 500,
          width: 300,
          height: 590,
          top: 0,
        },
        {
          left: 0,
          top: 600,
          width: 800,
          height: 200,
        }
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
  },
  created () {
    this.initGrids();
  },
  methods: {
    initGrids () {
      this.grids = this.grids.map((v, index) => {
        return {
          ...v,
          id: this.randomId(index)
        }
      })
    },
    setResize ({ id, width, height, left, top }) {
      const index = this.grids.findIndex(v => v.id == id);
      const currentGrid = this.grids[index];
      const currentLeft = left + currentGrid.left;
      const currentTop = top + currentGrid.top;


      this.$set(this.grids, index, {
        ...currentGrid,
        width,
        height,
        left: currentLeft,
        top: currentTop,

      })
    }
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