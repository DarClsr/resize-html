<template>
  <div class="grid-item" :style="{left:leftNum+'px',top:topNum+'px'}">
    <div class="resizable-box" :class="`resize-box-${classId}`" :style="{width:width+'px',height:height+'px'}">
      <div class="resizers">
        {{classId}}
        {{'width--'+ width}}
        <div class="resizer-border" @mousedown="(e) => resizerDown(e, i)" @mouseup="(e) => resizerUp(e, i)" :class="`${border.key}`" v-for="(border, i) in borders" :key="border.key"></div>
      </div>
    </div>
  </div>

</template>

<script>
export default {
  props: {
    minWidth: {
      type: Number,
      default: 100,
    },
    maxWidth: {
      type: Number,
      default: 500,
    },
    minHeight: {
      type: Number,
      default: 100,
    },
    width: {
      type: Number,
      default: 300,
    },
    height: {
      type: Number,
      default: 300,
    },
    maxHeight: {
      type: Number,
      default: 500,
    },
    leftNum: {
      type: Number,
      default: 0,
    },
    topNum: {
      type: Number,
      default: 0,
    },

    gutters: {
      type: Number,
      default: 10,
    },
    classId: {
      type: Number || String,
      default: "",
    },
  },
  data () {
    return {
      minimum_size: 100,
      isChange: false,
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
      this.reszieParentElement = document.querySelector(`.resize-box-${this.classId}`);
      const resizers = this.reszieParentElement.querySelectorAll(".resizer-border");
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
      this.isChange = false;
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
          this.currentResizer.style.left = resizeBorderLeft + "px";
          break;
        case "top":
          resizeBorderTop = mouseY - this.original_y;
          this.currentResizer.style.top = resizeBorderTop + "px";
          break;
        case "bottom":
          resizeBorderTop = this.original_mouse_y - mouseY;
          this.currentResizer.style.bottom = resizeBorderTop + "px";
          break;
      }
    },
    initBorderPlace () {
      for (let i in this.borders) {
        const border = this.borders[i];
        this.currentResizers[i].style = border.style;
      }
    },
    isAllow () {
      let parentWidth = this.reszieParentElement.offsetWidth;
      let parentHeight = this.reszieParentElement.offsetHeight;
      let parentLeft = this.reszieParentElement.offsetLeft;
      let parentTiop = this.reszieParentElement.offsetTop;
      const currentBorderLeft = this.currentResizer.offsetLeft;
      const currentBorderTop = this.currentResizer.offsetTop;
      let allow = true;
      switch (this.direction) {
        case "left":
          allow = currentBorderLeft + parentLeft > parentLeft + parentWidth ? false : true;
          break;
        case "top":
          allow = currentBorderTop + parentTiop > parentTiop + parentHeight ? false : true;
          break;
        case "right":
          allow = currentBorderLeft < 0 ? false : true;
          break;
        case "bottom":
          allow = currentBorderTop < 0 ? false : true;
          break;
      }
      return allow;
    },
    resizerUp (e, i) {
      this.currentResieIndex = i;
      e.preventDefault();
      let currentBorderLeft = this.currentResizer.offsetLeft;
      let currentBorderTop = this.currentResizer.offsetTop;
      let parentWidth = this.reszieParentElement.offsetWidth;
      let parentHeight = this.reszieParentElement.offsetHeight;
      let parentLeft = this.reszieParentElement.offsetLeft;
      let parentTiop = this.reszieParentElement.offsetTop;
      let spaceNum = 0;
      let cloumnNum = 0;
      let leftNumber = 0;
      this.isChange = true;

      switch (this.direction) {
        case "right":
          spaceNum = currentBorderLeft + this.currentResizer.offsetWidth;
          parentWidth = spaceNum;
          break;
        case "left":
          spaceNum = currentBorderLeft;
          if (parentWidth <= this.minWidth) {
            currentBorderLeft = 0
          }
          leftNumber = parentLeft + currentBorderLeft;
          parentWidth =
            spaceNum > 0 ? parentWidth - spaceNum : parentWidth + (-spaceNum);
          parentLeft = leftNumber;
          break;
        case "top":
          cloumnNum = currentBorderTop;
          parentHeight =
            cloumnNum > 0 ? parentHeight - cloumnNum : parentHeight - cloumnNum;
          parentTiop = cloumnNum + parentTiop;

          break;
        case "bottom":
          cloumnNum = currentBorderTop > 0 ? currentBorderTop : 0;
          parentHeight = cloumnNum;

          break;
      }


      // 限制宽
      if (parentWidth < this.minWidth) {
        console.log(`min width is ${this.minWidth}px`);
        parentWidth = this.minWidth;
      } else if (parentWidth > this.maxWidth) {
        console.log(`max width is ${this.maxWidth}px`);
        parentWidth = this.maxWidth;
      }

      // 限制宽
      if (parentHeight < this.minHeight) {
        console.log(`min width is ${this.minWidth}px`);
        parentHeight = this.minHeight;
      } else if (parentHeight > this.maxHeight) {
        console.log("max width is 500px");
        parentHeight = this.maxHeight;
      }

      const isAllow = this.isAllow();
      if (isAllow && this.isChange) {
        // this.reszieParentElement.style.width = parentWidth + "px";
        // this.reszieParentElement.style.height = parentHeight + "px";

        // this.reszieParentElement.style.left = parentLeft + "px";
        // this.reszieParentElement.style.top = parentTiop + "px";
        this.$emit("change", {
          id: this.classId,
          left: parentLeft,
          top: parentTiop,
          width: parentWidth,
          height: parentHeight,
          direction: this.direction,
        })

        this.isChange = false;

      }

      // console.log(width)
      this.currentResizer.classList.remove("actived");



      this.initBorderPlace();
    },
  },
};
</script>

<style lang="css" scoped>
.grid-item {
  width: auto;
  height: auto;
  position: absolute;
}
.resizable-box {
  background: white;
  width: 300px;
  height: 300px;
  position: relative;
}

.resizable-box .resizers {
  width: 100%;
  height: 100%;
  box-sizing: border-box;
  border: 20px solid #4286f4;
}

.resizable-box .resizers .resizer-border {
  position: absolute;
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
  z-index: 99999;
}
</style>