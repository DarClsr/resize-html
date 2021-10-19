<template>
  <div class="resizable-box">

    <div class="resizers">
      <div class="resizer-border" @mousedown="(e) => resizerDown(e, i)" @mouseup="(e) => resizerUp(e, i)" :class="`${border.key}`" v-for="(border, i) in borders" :key="border.key"></div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    minWidth: {
      type: Number,
      default: 50,
    },
    maxWidth: {
      type: Number,
      default: 500,
    },
    minHeight: {
      type: Number,
      default: 10,
    },
    maxHeight: {
      type: Number,
      default: 500,
    },
  },
  data () {
    return {
      minimum_size: 100,
      original_width: 0, //拖动div的宽度
      original_height: 0, //拖动div的高度
      original_x: 0, //拖动div的left值
      original_y: 0, //拖动div的top值
      original_mouse_x: 0, //鼠标的left值
      original_mouse_y: 0, //鼠标的top值
      reszieParentElement: null, //拖动div
      currentResizers: [], //存储拖动div的边
      currentResieIndex: 0, //拖动div边的下标
      borders: [
        {
          key: "top",
          style: "top:0px",
        },
        {
          key: "bottom",
          style: "bottom:0px",
        },
        {
          key: "left",
          style: "left:0px",
        },
        {
          key: "right",
          style: "right:0px",
        },
      ],
    };
  },
  mounted () {
    this.initdivs();
  },
  computed: {
    direction () {
      return this.borders[this.currentResieIndex].key;
    },
    currentResizer () {
      return this.currentResizers[this.currentResieIndex];
    },
  },
  methods: {
    initdivs () {
      this.reszieParentElement = document.querySelector(".resizable-box");
      const resizers = document.querySelectorAll(".resizer-border");
      for (let i = 0; i < resizers.length; i++) {
        this.$set(this.currentResizers, i, resizers[i]);
      }
    },
    initParentDiv (e) {
      this.original_width = parseFloat(
        getComputedStyle(this.reszieParentElement, null)
          .getPropertyValue("width")
          .replace("px", "")
      );
      this.original_height = parseFloat(
        getComputedStyle(this.reszieParentElement, null)
          .getPropertyValue("height")
          .replace("px", "")
      );
      this.original_x = this.reszieParentElement.getBoundingClientRect().left;
      this.original_y = this.reszieParentElement.getBoundingClientRect().top;
      this.original_mouse_x = e.pageX;
      this.original_mouse_y = e.pageY;
    },
    resizerDown (e, i) {
      this.currentResieIndex = i;
      e.preventDefault();
      // 改变拖动div的class
      this.currentResizer.classList.add("actived");
      // 初始化拖动DIV的宽高 距离 以及 坐标值
      this.initParentDiv(e);
      window.addEventListener("mousemove", this.resize);
      window.addEventListener("mouseup", this.stopResize);
    },
    getResizeInfo (e) {
      let width = this.original_width + (e.pageX - this.original_mouse_x),
        height = this.original_height + (e.pageY - this.original_mouse_y);
      switch (this.direction) {
        case "right":
          width = this.original_width + (e.pageX - this.original_mouse_x);
          break;
      }
      return { width, height };
    },

    resize (e) {
      this.resizerMove(e);
    },
    stopResize () {
      window.removeEventListener("mousemove", this.resize);
      this.currentResizer.classList.remove("actived");
    },
    resizerMove (e) {
      const mouseX = e.pageX;
      const mouseY = e.pageY;
      let resizeBorderLeft = 0,
        resizeBorderTop = 0;
      switch (this.direction) {
        case "right":
          resizeBorderLeft = mouseX - this.original_x;
          this.currentResizer.style.left = resizeBorderLeft + "px";
          break;
        case "left":
          resizeBorderLeft = mouseX - this.original_x;
          this.currentResizer.style.left = (resizeBorderLeft > 0 ? resizeBorderLeft : 0) + "px";
          break;
        case "top":
          resizeBorderTop = mouseY - this.original_y;
          this.currentResizer.style.top = (resizeBorderTop > 0 ? resizeBorderTop : 0) + "px";
          break;

      }
    },
    initBorderPlace () {
      for (let i in this.borders) {
        const border = this.borders[i];
        this.currentResizers[i].style = border.style;
      }
    },
    resizerUp (e, i) {
      this.currentResieIndex = i;
      e.preventDefault();
      const currentBorderLeft = this.currentResizer.offsetLeft;
      let parentWidth = this.reszieParentElement.offsetWidth;
      let parentHeight = this.reszieParentElement.offsetHeight;
      let parentLeft = this.reszieParentElement.offsetLeft;
      let parentTiop = this.reszieParentElement.offsetTop;
      let spaceNum = 0;
      switch (this.direction) {
        case "right":
          spaceNum = currentBorderLeft + this.currentResizer.offsetWidth;
          parentWidth = spaceNum;
          break;
        case "left":
          parentWidth = (parentLeft + parentWidth) - e.clientX
          parentLeft = e.clientX > 0 ? e.clientX : 0
          break;
        case "top":
          parentHeight = (parentTiop + parentHeight) - e.clientY;
          parentTiop = e.clientY > 0 ? e.clientY : 0
          break;
      }



      // 限制宽
      if (parentWidth < this.minWidth) {
        console.log(`min width is ${this.minWidth}px`)
        parentWidth = this.minWidth;
      } else if (parentWidth > this.maxWidth) {
        console.log(`max width is ${this.maxWidth}px`)
        parentWidth = this.maxWidth;
      }

      // 限制宽
      if (parentHeight < this.minHeight) {
        console.log(`min width is ${this.minWidth}px`)
        parentHeight = this.minHeight;
      } else if (parentHeight > this.maxHeight) {
        console.log("max width is 500px")
        parentHeight = this.maxHeight;
      }

      this.reszieParentElement.style.width = parentWidth + "px";
      this.reszieParentElement.style.height = parentHeight + "px";
      this.reszieParentElement.style.left = parentLeft + "px";
      this.reszieParentElement.style.top = parentTiop + "px";

      // console.log(width)
      this.currentResizer.classList.remove("actived");

      this.initBorderPlace();
    },
  },
};
</script>

<style lang="css" scoped>
* {
  box-sizing: border-box;
}
.resizable-box {
  background: white;
  width: 300px;
  height: 300px;
  position: absolute;
  top: 300px;
  left: 0;
}

.resizable-box .resizers {
  width: 100%;
  height: 100%;
  border: 20px solid #4286f4;
  box-sizing: border-box;
}

.resizable-box .resizers .resizer-border {
  position: absolute;
  background: red;
}

.resizer-border.top,
.resizer-border.bottom {
  width: calc(100% - 40px);
  height: 20px;
  cursor: n-resize;
}

.resizer-border.left,
.resizer-border.right {
  height: calc(100% - 40px);
  width: 20px;
  cursor: e-resize;
}

.resizer-border.top {
  top: 0;
}
.resizer-border.left {
  left: 0;
}
.resizer-border.right {
  right: 0;
}
.resizer-border.bottom {
  bottom: 0;
}

.resizer-border.actived {
  background: red;
}
</style>