<template>
  <div class="resizable">
    <div class="resizers">
      <div class="resizer top-left"></div>
      <div class="resizer top-right"></div>
      <div class="resizer bottom-left"></div>
      <div class="resizer bottom-right"></div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      minimum_size: 100,
      original_width: 0,
      original_height: 0,
      original_x: 0,
      original_y: 0,
      original_mouse_x: 0,
      original_mouse_y: 0,
      reszieParentElement: null,
      currentResizers: [],
      currentResieIndex: 0,
    };
  },
  mounted() {
    this.initReizeDivs();
  },
  computed: {
    currentResizer() {
      return this.currentResizers[this.currentResieIndex];
    },
  },
  methods: {
    initReizeDivs() {
      this.reszieParentElement = document.querySelector(".resizable");
      const resizers = document.querySelectorAll(".resizer");
      for (let i = 0; i < resizers.length; i++) {
        this.$set(this.currentResizers, i, resizers[i]);
        this.currentResizers[i].addEventListener("mousedown", (e) => {
          this.currentResieIndex = i;
          this.resizerDown(e);
        });
      }
    },

    resizerDown(e) {
      e.preventDefault();
      console.log("mouse down");
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
      window.addEventListener("mousemove", this.resize);
      window.addEventListener("mouseup", this.stopResize);
    },
    stopResize() {
      window.removeEventListener("mousemove", this.resize);
    },
    resize(e) {
      if (this.currentResizer.classList.contains("bottom-right")) {
        const width = this.original_width + (e.pageX - this.original_mouse_x);
        const height = this.original_height + (e.pageY - this.original_mouse_y);
        if (width > this.minimum_size) {
          this.reszieParentElement.style.width = width + "px";
        }
        if (height > this.minimum_size) {
          this.reszieParentElement.style.height = height + "px";
        }
      } else if (this.currentResizer.classList.contains("bottom-left")) {
        const height = this.original_height + (e.pageY - this.original_mouse_y);
        const width = this.original_width - (e.pageX - this.original_mouse_x);
        if (height > this.minimum_size) {
          this.reszieParentElement.style.height = height + "px";
        }
        if (width > this.minimum_size) {
          this.reszieParentElement.style.width = width + "px";
          this.reszieParentElement.style.left =
            this.original_x + (e.pageX - this.original_mouse_x) + "px";
        }
      } else if (this.currentResizer.classList.contains("top-right")) {
        const width = this.original_width + (e.pageX - this.original_mouse_x);
        const height = this.original_height - (e.pageY - this.original_mouse_y);
        if (width > this.minimum_size) {
          this.reszieParentElement.style.width = width + "px";
        }
        if (height > this.minimum_size) {
          this.reszieParentElement.style.height = height + "px";
          this.reszieParentElement.style.top =
            this.original_y + (e.pageY - this.original_mouse_y) + "px";
        }
      } else {
        const width = this.original_width - (e.pageX - this.original_mouse_x);
        const height = this.original_height - (e.pageY - this.original_mouse_y);
        if (width > this.minimum_size) {
          this.reszieParentElement.style.width = width + "px";
          this.reszieParentElement.style.left =
            this.original_x + (e.pageX - this.original_mouse_x) + "px";
        }
        if (height > this.minimum_size) {
          this.reszieParentElement.style.height = height + "px";
          this.reszieParentElement.style.top =
            this.original_y + (e.pageY - this.original_mouse_y) + "px";
        }
      }
    },
  },
};
</script>

<style lang="css" scoped>
.resizable {
  background: white;
  width: 100px;
  height: 100px;
  position: absolute;
  top: 100px;
  left: 100px;
}

.resizable .resizers {
  width: 100%;
  height: 100%;
  border: 3px solid #4286f4;
  box-sizing: border-box;
}

.resizable .resizers .resizer {
  width: 10px;
  height: 10px;
  border-radius: 50%; /*magic to turn square into circle*/
  background: white;
  border: 3px solid #4286f4;
  position: absolute;
}

.resizable

.resizable .resizers .resizer.top-left {
  left: -5px;
  top: -5px;
  cursor: nwse-resize; /*resizer cursor*/
}
.resizable .resizers .resizer.top-right {
  right: -5px;
  top: -5px;
  cursor: nesw-resize;
}
.resizable .resizers .resizer.bottom-left {
  left: -5px;
  bottom: -5px;
  cursor: nesw-resize;
}
.resizable .resizers .resizer.bottom-right {
  right: -5px;
  bottom: -5px;
  cursor: nwse-resize;
}
</style>