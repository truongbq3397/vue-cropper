<template>
  <div
    class="v-cropper"
    :style="{
      height: `${height}px`,
      width: `${width}px`,
      transform: `scale(${zoom})`,
      'transform-origin': zoom > 1 ? 'left top' : 'center center',
    }"
    @mousedown="startCrop"
    @mousemove="cropMove"
    @mouseup="endCrop"
  >
    <!-- image -->
    <img
      class="v-crop-image"
      :width="width"
      :height="height"
      :src="src"
      id="image-crop"
      alt="image-crop"
    />
    <div class="v-cropper-overlay" v-show="mouseIsDown" />
    <div
      class="v-cropper-crop"
      :style="{
        width: `${boxEndWidth}px`,
        height: `${boxEndHeight}px`,
        transform: `translateX(${boxLeft}px) translateY(${boxTop}px)`,
      }"
    >
      <span class="v-cropper-view-box">
        <!-- image crop view box -->
        <img
          :style="{
            height: `${height}px`,
            width: `${width}px`,
            transform: `translateX(-${boxLeft}px) translateY(-${boxTop}px)`,
          }"
          class="v-crop-image-box"
          :src="src"
          alt="image-crop"
        />
      </span>
    </div>
  </div>
</template>
<script>
export default {
  props: {
    src: String,
    width: Number,
    height: Number,
    zoom: Number,
  },
  data() {
    return {
      mouseIsDown: false,
      endDrop: true,
      // Used to calculate where to start showing the dragging area
      startX: 0,
      startY: 0,
      endX: 0,
      endY: 0,

      // The box that contains the border and all required numbers.
      boxTop: 0,
      boxLeft: 0,
      boxEndWidth: 0,
      boxEndHeight: 0,
      offsetX: 0,
      offsetY: 0,
      imgEl: null,
    };
  },
  methods: {
    startCrop(event) {
      let { width, height, zoom } = this;
      this.startX = event.layerX / zoom;
      this.startY = event.layerY / zoom;
      this.offsetX = (width - width * zoom) / 2 / zoom;
      this.offsetY = (height - height * zoom) / 2 / zoom;
      this.mouseIsDown = true;
      this.endDrop = false;
    },
    cropMove: function (event) {
      //   const screenshotElem = document.getElementById("screenshot");
      const { endX, endY, startX, startY } = this;

      if (!this.endDrop) {
        this.endX = event.layerX / this.zoom;
        this.endY = event.layerY / this.zoom;
        // Calculating the values differently depending on how the user start's dragging.
        if (endX >= startX && endY >= startY) {
          this.boxTop = startY;
          this.boxLeft = startX;
          this.boxEndWidth = endX - startX;
          this.boxEndHeight = endY - startY;
        } else if (endX <= startX && endY >= startY) {
          this.boxLeft = endX;
          this.boxTop = startY;
          this.boxEndWidth = startX - endX;
          this.boxEndHeight = endY - startY;
        } else if (endX >= startX && endY <= startY) {
          this.boxLeft = startX;
          this.boxTop = endY;
          this.boxEndWidth = endX - startX;
          this.boxEndHeight = startY - endY;
        } else if (endX <= startX && endY <= startY) {
          this.boxLeft = endX;
          this.boxTop = endY;
          this.boxEndWidth = startX - endX;
          this.boxEndHeight = startY - endY;
        }
        if (this.zoom < 1) {
          this.boxTop -= this.offsetY;
          this.boxLeft -= this.offsetX;
        }
        if (this.zoom > 1) {
          let domMain = document.getElementById("screenshot");
          let offSetB = domMain.offsetHeight + domMain.offsetTop;
          let offSetL = domMain.offsetWidth + domMain.offsetLeft;
          console.log({ event, domMain });
          if (event.clientY >= offSetB - 40) {
            domMain.scrollTop += 20;
            console.log("--scroll bottom");
          }
          if (event.clientX >= offSetL - 40) {
            domMain.scrollLeft += 20;
            console.log("--scroll left");
          }
          if (event.clientX <= domMain.offsetLeft + 40) {
            domMain.scrollLeft -= 20;
            console.log("--scroll left");
          }
          if (event.clientY <= domMain.offsetTop + 40) {
            domMain.scrollTop -= 20;
            console.log("--scroll left");
          }
        }
      }
    },
    endCrop() {
      this.endDrop = true;
      //   this.mouseIsDown = false;
      //   this.boxLeft = 0;
      //   this.boxTop = 0;
      //   this.boxEndWidth = 0;
      //   this.boxEndHeight = 0;
      this.$emit("endCrop");
    },
    async crop() {
      const { boxLeft, boxTop, boxEndWidth, boxEndHeight, width, height } =
        this;
      let imgEl = this.$el.querySelector("#image-crop");
      // let's store the width and height of our image
      const inputWidth = imgEl.naturalWidth;
      const inputHeight = imgEl.naturalHeight;
      const scaleX = inputWidth / width;
      const scaleY = inputHeight / height;
      const outputWidth = boxEndWidth * scaleX;
      const outputHeight = boxEndHeight * scaleY;
      const outputX = boxLeft * scaleX;
      const outputY = boxTop * scaleY;
      // create a canvas that will present the output image
      const outputImage = document.createElement("canvas");
      // set it to the same size as the image
      outputImage.width = outputWidth;
      outputImage.height = outputHeight;
      // draw our image at position 0, 0 on the canvas
      const ctx = outputImage.getContext("2d");
      ctx.drawImage(
        imgEl,
        outputX,
        outputY,
        outputWidth,
        boxEndHeight * scaleY,
        0,
        0,
        outputWidth,
        outputHeight
      );
      let outputUrl = outputImage.toDataURL();
      const blob = await this.$b64ToBlob(outputUrl);
      return blob;
    },
  },
};
</script>
<style  scoped>
.v-cropper {
  user-select: none;
  cursor: crosshair;
}
.v-crop-image,
.v-crop-image-box {
  -webkit-user-select: none;
  -khtml-user-select: none;
  -moz-user-select: none;
  -o-user-select: none;
  user-select: none;
  pointer-events: none;
}
.v-cropper-overlay {
  background-color: #000;
  opacity: 0.5;
  bottom: 0;
  left: 0;
  position: absolute;
  right: 0;
  top: 0;
  user-select: none;
  pointer-events: none;
}
.v-cropper-view-box {
  display: block;
  height: 100%;
  outline: 2px solid #39f;
  outline-color: rgba(51, 153, 255, 0.75);
  overflow: hidden;
  width: 100%;
  user-select: none;
  pointer-events: none;
}
.v-cropper-crop {
  position: absolute;
  user-select: none;
  pointer-events: none;
  bottom: 0;
  left: 0;
  position: absolute;
  right: 0;
  top: 0;
}
</style>