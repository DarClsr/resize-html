<template>
  <div class="sizeBox" :style="defaultStyle">
    <resize-div
      :maxHeight="currentMaxHeight(item)"
      :gutters="gutters"
      :width="item.width"
      :height="item.height"
      :maxWidth="boxWidth"
      :classId="item.id"
      v-for="(item, index) in grids"
      :key="index"
      :leftNum="item.left"
      :topNum="item.top"
      @change="setResize"
    />
  </div>
</template>

<script>
import ResizeDiv from "@/components/resize-div/index.vue";
export default {
  data() {
    return {
      boxWidth: 800,
      boxHeight: 800,
      gutters: 10,
      limit: 20,
      addNum: 0,

      grids: [
        // {
        //   left: 0,
        //   width: 300,
        //   height: 590,
        //   top: 0,
        // },

        // {
        //   left: 500,
        //   width: 300,
        //   height: 590,
        //   top: 0,
        // },
        {
          left: 0,
          top: 600,
          width: 800,
          height: 200,
        },
      ],
    };
  },
  computed: {
    currentMaxWidth() {
      return (item) => {
        const blocks = this.grids.filter(
          (v) => v.top == item.top && v.id != item.id
        );
        const width = blocks.reduce((total, cur) => {
          return (total += cur.width);
        }, 0);
        console.log(width,"max",item.id)
        return this.boxWidth - width;
      };
    },
    currentMaxHeight() {
      return (item) => {
        const blocks = this.grids.filter(
          (v) => v.left == item.left && v.id != item.id
        );
        const width = blocks.reduce((total, cur) => {
          return (total += cur.height);
        }, 0);
        return this.boxHeight - width;
      };
    },
    randomId() {
      return (i) => {
        return Math.floor(Math.random() * 10000000 * (i + 1));
      };
    },
    defaultStyle() {
      return `width:${this.boxWidth};height:${this.boxHeight}`;
    },
  },
  components: {
    ResizeDiv,
  },
  created() {
    this.initGrids();
  },
  methods: {
    initGrids() {
      this.grids = this.grids.map((v, index) => {
        return {
          ...v,
          id: this.randomId(index),
        };
      });
    },
    isAdd(left, direction, top, prev) {
      const prevleft = prev.left;
      let is = left > this.limit;
      let blanaceWidth = 0;
      switch (direction) {
        case "left":
          if (prevleft != 0) {
            is = false;
          }
          blanaceWidth = this.grids
            .filter((v) => v.top == top)
            .reduce((total, cur) => {
              return (total = total - cur.width);
            }, this.boxWidth);
          break;
      }

      if (blanaceWidth <= 0) {
        is = false;
      }

      return is;
    },
    isReduce(left, direction, top, prev) {
      const prevleft = prev.left;
      let is = left < this.limit;

      console.log(direction,top,"reduce",prevleft)

      return is;
    },
    reduceGrid(direction,id){
      console.log("删除 不改变");
      const currentIndex = this.grids.findIndex((v) => v.id == id);
      const { top,left } = this.grids[currentIndex];

      let removeIndex=0;

      switch(direction){
        case "left":
          removeIndex=this.grids.findIndex(v=>{
            return v.top==top&&v.left<left;
          })
          break;
      }

      if(removeIndex>-1){
        this.grids.splice(removeIndex,1)
      }

    },
    addGrid(direction, id) {
      console.log("添加 不改变");
      const currentIndex = this.grids.findIndex((v) => v.id == id);
      const { top, height } = this.grids[currentIndex];
      let gridWidth = this.grids
        .filter((v) => v.top == top)
        .reduce((result, cur) => {
          return result - cur.width;
        }, this.boxWidth);
      let gridIttem = {};
      switch (direction) {
        case "left":
          gridIttem = {
            id: this.randomId(Math.floor(Math.random() * 100000)),
            left: 0,
            top: top,
            width: gridWidth,
            height,
          };
          this.grids.push(gridIttem);
          break;
      }
    },
    initRowGrid(direction, id) {
      console.log("改变 不添加");
      const currentIndex = this.grids.findIndex((v) => v.id == id);
      const { top, left } = this.grids[currentIndex];
      let changeGrid = null;
      let changeGridIndex = 0;
      let gridWidth = 0;
      switch (direction) {
        case "left":
          changeGridIndex = this.grids.findIndex((v) => v.left < left);

          changeGrid = this.grids[changeGridIndex];

          gridWidth = this.grids
            .filter((v) => v.top == top)
            .reduce((result, cur) => {
              return (result =
                result - (cur.id == changeGrid.id ? 0 : cur.width));
            }, this.boxWidth);
          this.$set(this.grids, changeGridIndex, {
            ...this.grids[changeGridIndex],
            width: gridWidth,
          });
          break;
      }
    },
    setResize({ id, width, height, left, top, direction }) {
      const index = this.grids.findIndex((v) => v.id == id);
      const currentGrid = this.grids[index];
      let currentLeft = left + currentGrid.left;
      let currentTop = top + currentGrid.top;
      // let currentWidth=left>0?width:currentLeft+width;
      if (currentLeft <= 0) {
        currentLeft = 0;
      }
      if (currentTop <= 0) {
        currentTop = 0;
      }

      const prev = this.grids[index];
      this.$nextTick(() => {
        this.$set(this.grids, index, {
          ...currentGrid,
          width,
          height,
          left: currentLeft,
          top: currentTop,
        });
        const isAdd = this.isAdd(currentLeft, direction, currentTop, prev);
        const isReduce=this.isReduce(currentLeft, direction, currentTop, prev)
        if (isAdd) {
          this.addGrid(direction, id);
        } else if(isReduce) {
          this.reduceGrid(direction,id)
        }else{
          this.initRowGrid(direction, id);
        }
      });
    },
  },
};
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