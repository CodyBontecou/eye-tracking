<template>
  <div class="w-screen h-screen">
    <!-- <h1>Hey</h1> -->
    <div id="gazecloudopts" class="centered-element">
      <button
        id="gc_start"
        class="
          px-4
          py-2
          rounded
          bg-blue-300
          shadow-lg
          hover:bg-blue-200
          hover:shadow
          active:shadow-none
        "
        type="button"
        onclick="GazeCloudAPI.StartEyeTracking();"
      >
        Start tracking
      </button>
      <button
        id="gc_stop"
        class="
          px-4
          py-2
          rounded
          bg-blue-300
          shadow-lg
          hover:bg-blue-200
          hover:shadow
          active:shadow-none
        "
        type="button"
        onclick="GazeCloudAPI.StopEyeTracking();"
      >
        Stop tracking
      </button>
    </div>
    <div id="gaze" class="gaze absolute border border-2 border-grey-200"></div>
    <div id="container">
      <!-- <img src='regression.jpg'> -->
    </div>

    <vue-draggable-resizable
      :class-name="resizableClass"
      :parent="true"
      @dragging="onDrag"
      @resizing="onResize"
    >
      <p>You can drag me around and resize me as you wish.</p>
    </vue-draggable-resizable>
    <!-- <vue-draggable-resizable
      v-for="element in elements"
      :key="element.id"
      class-name-active="my-active-class"
      :x="element.x"
      :y="element.y"
      :w="200"
      :h="200"
      :resizable="true"
      :parent="true"
      @dragging="(left, top) => dragging(element.id, left, top)"
      @dragstop="(left, top) => dragstop(element.id, left, top)"
    > -->
    <!-- </vue-draggable-resizable> -->
  </div>
</template>

<script>
import 'vue-draggable-resizable/dist/VueDraggableResizable.css'
export default {
  components: {
    VueDraggableResizable: () => import('vue-draggable-resizable'),
  },
  data() {
    return {
      width: 0,
      height: 0,
      x: 0,
      y: 0,
      calibrated: false,
      inZone: false,
      elements: [
        { id: 1, x: 0, y: 0 },
        { id: 2, x: 0, y: 0 },
      ],
    }
  },
  computed: {
    draggingElement() {
      if (!this.draggingId) return

      return this.elements.find((el) => el.id === this.draggingId)
    },
    resizableClass() {
      if (this.inZone) {
        return 'active-state'
      }
      return 'inactive-state'
    },
  },
  watch: {
    inZone() {
      if (this.inZone === true) {
        console.log('log user is in zone')
      }
    },
  },
  mounted() {
    // eslint-disable-next-line no-undef
    GazeCloudAPI.OnCalibrationComplete = function () {
      const gaze = document.getElementById('gaze')
      if (gaze.style.display === 'none') gaze.style.display = 'block'
      this.calibrated = true
    }
    // eslint-disable-next-line no-undef
    GazeCloudAPI.OnCamDenied = function () {}
    // eslint-disable-next-line no-undef
    GazeCloudAPI.OnError = function (msg) {
      console.log('err: ' + msg)
    }
    // eslint-disable-next-line no-undef
    GazeCloudAPI.UseClickRecalibration = true
    // eslint-disable-next-line no-undef
    GazeCloudAPI.OnResult = this.PlotGaze

    // Kalman Filter defaults to on. Can be toggled by user.
    window.applyKalmanFilter = true

    // Set to true if you want to save the data even if you reload the page.
    window.saveDataAcrossSessions = true
  },
  methods: {
    onResize(x, y, width, height) {
      this.x = x
      this.y = y
      this.width = width
      this.height = height
    },
    onDrag(x, y) {
      this.x = x
      this.y = y
    },
    PlotGaze(GazeData) {
      /*
        GazeData.state // 0: valid gaze data; -1 : face tracking lost, 1 : gaze uncalibrated
        GazeData.docX // gaze x in document coordinates
        GazeData.docY // gaze y in document cordinates
        GazeData.time // timestamp
    */

      let docx = GazeData.docX
      let docy = GazeData.docY

      const gaze = document.getElementById('gaze')
      docx -= gaze.clientWidth / 2
      docy -= gaze.clientHeight / 2

      if (
        docx >= this.x &&
        docy >= this.y &&
        docx <= Math.abs(this.x + this.width) &&
        docy <= Math.abs(this.y + this.height)
      ) {
        this.inZone = true
      } else {
        this.inZone = false
      }

      gaze.style.left = docx + 'px'
      gaze.style.top = docy + 'px'
    },
    // dragging(id, left, top) {
    //   this.draggingId = id

    //   if (!this.sync) return

    //   const offsetX = left - this.draggingElement.x
    //   const offsetY = top - this.draggingElement.y

    //   const deltaX = this.deltaX(offsetX)
    //   const deltaY = this.deltaY(offsetY)

    //   this.elements.map((el) => {
    //     if (el.id !== id) {
    //       el.x += deltaX
    //       el.y += deltaY
    //     }

    //     return el
    //   })
    // },
    // dragstop(id, left, top) {
    //   this.elements.map((el) => {
    //     if (el.id === id) {
    //       el.x = left
    //       el.y = top
    //     }

    //     return el
    //   })

    //   this.draggingId = null
    //   this.prevOffsetX = 0
    //   this.prevOffsetY = 0
    // },
    // deltaX(offsetX) {
    //   const ret = offsetX - this.prevOffsetX

    //   this.prevOffsetX = offsetX

    //   return ret
    // },
    // deltaY(offsetY) {
    //   const ret = offsetY - this.prevOffsetY

    //   this.prevOffsetY = offsetY

    //   return ret
    // },
  },
}
</script>

<style>
.centered-element {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translateX(-50%) translateY(-50%);
}
.isZone {
  @apply border-2 border-green-500;
}
.active-state {
  border: 2px solid rgb(44, 129, 44);
  background-color: rgba(0, 128, 0, 0.2);
}
.inactive-state {
  border: 2px solid white;
}
.gaze {
  width: 100px;
  height: 100px;
  border-radius: 50%;
  box-shadow: 0 0 100px 3px rgba(125, 125, 125, 0.5);
  pointer-events: none;
  z-index: 999999;
}
</style>
