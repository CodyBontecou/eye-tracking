<template>
  <div id="container">
    <div
      class="drag-zone"
      :class="{
        active: draggable,
      }"
      :draggable="draggable"
      @drag="onDrag($event)"
      @drop="onDrop($event)"
      @dragstart="startDrag($event)"
      @dragover.prevent
      @dragenter.prevent
    >
      <div class="drag-el" @click="draggable = !draggable">+</div>
      <div class="area" :style="rectangle"></div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      draggable: false,
      tempLeft: 0,
      tempTop: 0,
      rectangle: {
        left: 0,
        top: 0,
        right: 0,
        bottom: 0,
        'z-index': -1,
      },
    }
  },
  methods: {
    startDrag(event) {
      this.tempLeft = event.x
      this.tempTop = event.y

      const background = document.getElementById('container').style.background
      document.getElementById('container').style.background = 'blue'
      setTimeout(
        (function (background) {
          return function () {
            // Set style here
            document.getElementById('container').style.background = background
          }
        })(background),
        1
      )
    },
    onDrop(event) {
      event.target.style.opacity = 1
      this.rectangle['z-index'] = 1
    },
    onDrag(event) {
      event.target.style.opacity = 0.5
      this.rectangle['z-index'] = 1
      this.rectangle.left = this.tempLeft + 'px'
      this.rectangle.top = this.tempTop + 'px'
      this.rectangle.right = window.innerWidth - event.x + 'px'
      this.rectangle.bottom = window.innerHeight - event.y + 'px'
    },
  },
}
</script>

<style scoped>
.area {
  border: 1px solid white;
  background-color: black;
  opacity: 0.1;
  position: absolute;
}
.active {
  cursor: crosshair;
}
#container {
  background-color: #eee;
  height: 100vh;
  width: 100vw;
}
.drag-el {
  position: absolute;
  border-radius: 100%;
  width: 1rem;
  height: 1rem;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: white;
  margin: 1rem;
  padding: 1rem;
  cursor: pointer;
  z-index: 100;
  box-shadow: rgba(149, 157, 165, 0.2) 0px 8px 24px;
}
.drag-zone {
  height: 100%;
  width: 100%;
}
</style>
