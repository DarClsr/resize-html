<template>
  <div class="sizeBox" :style="defaultStyle">
    <resize-div :index="index" :maxHeight="boxHeight" :gutters="gutters" :width="item.width" :height="item.height" :maxWidth="boxWidth" :classId="item.id" v-for="(item, index) in grids" :key="index" :leftNum="item.left" :topNum="item.top" @change="setResize" />
  </div>
</template>

<script>
import ResizeDiv from "@/components/resize-div/index.vue";
export default {
  data () {
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
        //   height: 800,
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
          top: 0,
          width: 800,
          height: 800,
        },
      ],
    };
  },
  computed: {
    isLarge(){
      return (prev,width)=>{
        return prev.width>width;
      }

    },
    currentMaxWidth() {
      return (item) => {
        const blocks = this.grids.filter(
          (v) => v.top == item.top && v.id != item.id
        );
        const width = blocks.reduce((total, cur) => {
          return (total += cur.width);
        }, 0);
        console.log(width, "max", item.id);
        return this.boxWidth - width;
      };
    },
    currentMaxHeight () {
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
    randomId () {
      return (i) => {
        return Math.floor(Math.random() * 10000000 * (i + 1));
      };
    },
    defaultStyle () {
      return `width:${this.boxWidth};height:${this.boxHeight}`;
    },
  },
  components: {
    ResizeDiv,
  },
  created () {
    this.initGrids();
  },
  methods: {
    initGrids () {
      this.grids = this.grids.map((v, index) => {
        return {
          ...v,
          id: this.randomId(index),
        };
      });
    },
    isAdd (left, direction, top, prev, width, height) {
      console.log(width, height);
      let prevleft = prev.left;
      let prevTop = prev.top;
      let is = true;
      let blanaceWidth = 0;
      let blanaceHeight = 0;
      switch (direction) {
        case "left":
          is = left > this.limit;
          if (prevleft > this.limit) {
            is = false;
          }
          blanaceWidth = this.grids
            .filter((v) => v.top == top)
            .reduce((total, cur) => {
              return (total = total - cur.width);
            }, this.boxWidth);
          break;
        case "right":
          prevleft = prevleft + prev.width;
          console.log("right",prevleft)
          if (prevleft < (this.boxWidth-this.limit)) {
            is = false;
          }
          blanaceWidth = this.grids
            .filter((v) => v.top == top)
            .reduce((total, cur) => {
              return (total = total - cur.width);
            }, this.boxWidth);
          if (blanaceWidth <= 0) {
            is = false;
          }

          break;
        case "top":
          console.log(top, "top", prevTop);
          is = top > this.limit;
          if (prevTop != 0) {
            is = false;
          }
          blanaceHeight = this.grids
            .filter((v) => v.left == left)
            .reduce((total, cur) => {
              return (total = total - cur.height);
            }, this.boxHeight);

          if (blanaceHeight <= 0) {
            is = false;
          }
          break;
        case "bottom":
          prevTop = prevTop + prev.height;
          console.log("lllllll","bottom",prevTop)

          if (prevTop < this.boxHeight-this.limit) {
            is = false;
          }

          console.log(is,"bottom1")
          blanaceHeight = this.grids
            .filter((v) => v.left == left)
            .reduce((total, cur) => {
              return (total = total - cur.height);
            }, this.boxHeight);
          if (blanaceHeight <= 0) {
            is = false;
          }

          console.log(is,"bottom2")


          break;
      }

      return is;
    },
    isReduce (left, direction, top, width, height) {
      let is = false;
      switch (direction) {
        case "left":
          is = left < this.limit;
          break;
        case "right":
          is = left > (this.boxWidth - this.limit - width);
          break;
        case "top":
          is = top < this.limit;
          break;
        case "bottom":
          is = top >= (this.boxHeight - this.limit - height);
          break;
      }
      return is;
    },
    reduceGrid (direction, id) {
      const currentIndex = this.grids.findIndex((v) => v.id == id);
      const { top, left, height } = this.grids[currentIndex];
      let removeIndex = 0;
      console.log("remove", direction)
      switch (direction) {
        case "left":
          removeIndex = this.grids.findIndex((v) => {
            return v.top == top && v.left < left;
          });
          break;
        case "right":
          removeIndex = this.grids.findIndex((v) => {
            return v.top == top && v.left > left;
          });
          break;
        case "top":
          removeIndex = this.grids.findIndex((v) => {
            return v.left == left && v.top < top
          });
          break;
        case "bottom":
          removeIndex = this.grids.findIndex((v) => {
            return v.left == left && (v.top + v.height) > (top + height)
          })
          break;
      }

      console.log(removeIndex, "remopve", direction)
      if (removeIndex > -1) {
        this.grids.splice(removeIndex, 1);
      }
    },
    addGrid (direction, id) {
      console.log("添加 不改变");
      const currentIndex = this.grids.findIndex((v) => v.id == id);
      const { top, height, width, left } = this.grids[currentIndex];
      let gridWidth = this.grids
        .filter((v) => v.top == top)
        .reduce((result, cur) => {
          return result - cur.width;
        }, this.boxWidth);

      let gridHeight = this.grids
        .filter((v) => v.left == left)
        .reduce((result, cur) => {
          return result - cur.height;
        }, this.boxHeight);
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
        case "right":
          gridIttem = {
            id: this.randomId(Math.floor(Math.random() * 100000)),
            left: this.boxWidth - gridWidth,
            top: top,
            width: gridWidth,
            height,
          };
          this.grids.push(gridIttem);
          break;
        case "top":
          gridIttem = {
            id: this.randomId(Math.floor(Math.random() * 100000)),
            left: left,
            top: 0,
            width: width,
            height: gridHeight,
          };
          this.grids.push(gridIttem);
          break;
           case "bottom":
          gridIttem = {
            id: this.randomId(Math.floor(Math.random() * 100000)),
            left: left,
            top: this.boxHeight-gridHeight,
            width,
            height:gridHeight,
          };
          this.grids.push(gridIttem);
          break;
      }
    },
    initRowGrid (direction, id) {
      console.log("改变 不添加");
      const currentIndex = this.grids.findIndex((v) => v.id == id);
      const { top, left,width,height } = this.grids[currentIndex];
      console.log(this.grids[currentIndex],currentIndex)
      let changeGrid = null;
      let changeGridIndex = 0;
      let gridWidth = 0;
      let gridHeight = 0;
      let prevChild='';
      let changeIndex=0;
      // let prevLeft=""

      console.log(direction, "change size")

      switch (direction) {
        case "left":
          changeGridIndex = this.grids.findIndex((v) => v.left < left);
          changeGrid = this.grids[changeGridIndex];
          gridWidth = changeGrid && this.grids
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
        case "right":
           changeIndex = this.grids.findIndex((v) => v.left > (left+width));
          changeGridIndex=changeIndex>-1?changeIndex:0;
          changeGrid = this.grids[changeGridIndex];
           prevChild=changeGridIndex>0?this.grids[changeGridIndex-1]:this.grids[0];
          gridWidth = this.grids
            .filter((v) => v.top == top)
            .reduce((result, cur) => {
              return (result =
                result - (cur.id == changeGrid.id ? 0 : cur.width));
            }, this.boxWidth);
          this.$set(this.grids, changeGridIndex, {
            ...this.grids[changeGridIndex],
            left: changeGridIndex>0? (prevChild.left+prevChild.width) :width,
            width: gridWidth,
          });
          break;
        case "top":
          changeGridIndex = this.grids.findIndex((v) => v.top < top);
          changeGrid = this.grids[changeGridIndex];
          gridHeight = this.grids
            .filter((v) => v.left == left)
            .reduce((result, cur) => {
              return (result =
                result - (cur.id == changeGrid.id ? 0 : cur.height));
            }, this.boxHeight);
          this.$set(this.grids, changeGridIndex, {
            ...this.grids[changeGridIndex],
            height: gridHeight,
          });
          break;
          case "bottom":
           changeIndex = this.grids.findIndex((v) => (v.top+v.height) > (top+height));
           console.log("changeindex",changeIndex)
          changeGridIndex=changeIndex>-1?changeIndex:0;
          changeGrid = this.grids[changeGridIndex];
           prevChild=changeGridIndex>0?this.grids[changeGridIndex-1]:this.grids[0];
             console.log("bottom",changeGridIndex,"init row grid")

          gridHeight = this.grids
            .filter((v) => v.left == left)
            .reduce((result, cur) => {
              return (result =
                result - (cur.id == changeGrid.id ? 0 : cur.height));
            }, this.boxHeight);
            console.log(gridHeight,"init size")
          this.$set(this.grids, changeGridIndex, {
            ...this.grids[changeGridIndex],
            top: changeGridIndex>0? (prevChild.top+prevChild.height) :height,
            height: gridHeight,

          });
          break;
      }
    },
    isChange (id, left, direction, top, width, height, prev) {
      const prevLeft = prev.left;
      const prevTop = prev.top;
      let currentLeft = left;
      let currentTop = top;
      let is = true;
      let childLength = 0;
      let child = null;
      switch (direction) {
        case "left":
          child = this.grids.find(v => {
            return (
              v.top == top &&
              v.left >= left &&
              v.id != id &&
              v.left >= this.limit
            )
          })
          childLength = this.grids.filter(v => {
            return (
              v.top == top &&
              v.left <= prevLeft &&
              v.id != id
            )
          }).length;

          break;
        case "right":
          currentLeft = left + width;
          child = this.grids.find(v => {
            return (
              v.top == top &&
              v.left <= currentLeft &&
              v.id != id &&
              (v.left + v.width) >= (this.boxWidth - this.limit)
            )
          })
          childLength = this.grids.filter(v => {
            return (
              v.top == top &&
              v.left > prevLeft &&
              v.id != id
            )
          }).length;

          break;
        case "top":
          child = this.grids.find(v => {
            return (
              v.left == left &&
              v.top >= top &&
              v.id != id &&
              v.top >= this.limit
            )
          })

          childLength = this.grids.filter(v => {
            return (
              v.left == left &&
              v.top <= prevTop &&
              v.id != id
            )
          }).length;
          break;
        case "bottom":
          currentTop = top + height;
          console.log(prevTop, "is change bottom")
          child = this.grids.find(v => {
            return (
              v.left == left &&
              v.top <= currentTop &&
              v.id != id &&
              (v.top + v.height) >= (this.boxHeight - this.limit)
            )
          })
          childLength = this.grids.filter(v => {
            return (
              v.left == left &&
              v.top >= prevTop &&
              v.id != id
            )
          }).length;
          break;
      }
      is = (child && childLength >= 2) ? false : true;
      console.log(is, "change", child, childLength);
      return is;
    },
    setResize ({ id, width, height, left, top, direction }) {
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
      const isChange = this.isChange(
        id,
        currentLeft,
        direction,
        currentTop,
        width,
        height,
        prev
      );
      if (!isChange) {
        console.log("不改变", width, height, currentTop, currentLeft, prev);
        return false;
      }
      this.$nextTick(() => {
        console.log("set grid",height);
        this.$set(this.grids, index, {
          ...currentGrid,
          width,
          height,
          left: currentLeft,
          top: currentTop,
        });

        const isAdd = this.isAdd(
          currentLeft,
          direction,
          currentTop,
          prev,
          width,
          height
        );
        const isReduce = this.isReduce(
          currentLeft,
          direction,
          currentTop,
          width,
          height,
          prev
        );
        console.log(isAdd, "add");
        console.log(isReduce, "reduce");
        if (isAdd) {
          this.addGrid(direction, id);
        } else if (isReduce) {
          this.reduceGrid(direction, id, currentLeft, currentTop);
        } else {
          this.initRowGrid(direction, id,index);
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