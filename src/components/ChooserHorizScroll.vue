<script>
let uuid = 0 // this is for keeping track of instances of the component
export default {
  name: 'ChooserHorizScroll',
  props: {
    mouseOutOfBounds: { // this allows the parent some control over what happens if the user lets go of the bar with the mouse outside of the component
      type: Boolean,
      default: false
    },
    targets: {
      type: Array,
      default () {return [ 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13 ]}
    }
  },
  data: function () {
    return {
      halfDeltaW: {}, // half of the difference between the frame element and the bar with all items in it
      xShift: 0, // this is the offset in pixels of the bar from its rest position
      barStartX: 0, // position of the bar when grabbed
      mouseXStart: null, // position of the mouse when bar is grabbed
      trackRecord: [],
      // animation speed parameters:
      relaxStepSize: 1,
      speedScale: 1,
      frictionFac: 0.1,
      lowSpeedLimit: 0.1,
      stepSizeLimit: 10,
      stillGoing: false,
      dx: 0
    }
  },
  computed: {
    xTransformStyle () {
      return {transform: 'translateX(' + this.xShift + 'px)'}
    },
    frameID () {
      return ("chooser-horiz-scroll-frame" + uuid.toString())
    },
    scrollerID () {
      return ("horiz-chooser-scrolling-bar" + uuid.toString())
    }
  },
  watch: {
    mouseOutOfBounds: function() {
      if (this.mouseOutOfBounds) {
        this.throwBar()
      }
    },
    // When the targets prop changes, recentre sort-scroll-bar
    targets: function() {
      this.xShift = 0
      this.barStartX = 0
    }
  },
  beforeCreate() { // Create a unique ID for this instance of this component
      this.uuid = uuid.toString();
      //console.log(this.uuid)
      uuid += 1;
  },
  mounted() {
    // this.$root.$on('mouseLeft',this.throwBar),
    this.compareWidths()
  },
  methods: {
    compareWidths () {
      var viewWidth
      var barWidth
      if (document.getElementById(this.frameID)) {
        //console.log("this.frameID = " + this.frameID)
        viewWidth = document.getElementById(this.frameID).offsetWidth
        }
      if (document.getElementById(this.scrollerID)) {
        barWidth = document.getElementById(this.scrollerID).offsetWidth
      }
      var halfDiff = (viewWidth - barWidth)/2
      //console.log("barWidth: " + barWidth + "; viewWidth: " + viewWidth)
      if (halfDiff >= 0) {
        this.halfDeltaW = { needsScroll: false, magnitude: 0 }
      } else {
        this.halfDeltaW = { needsScroll: true, magnitude: Math.abs(halfDiff) }
      }
      //console.log("this.halfDeltaW.magnitude = " + this.halfDeltaW.magnitude)
    },
    tempSelect (targ) {
      //console.log("invoked tempSelect with target " + key)
      //this.tempTarg = targ
      this.$emit('select-temptarg', targ)
    },
    selectWithClick (targ) {
      //this.selectedTarget = targ
      this.$emit('select-target', targ)
      this.$emit('select-temptarg', null)
    },
    grabBar (ev) {
      // Starts moving the bar
      this.compareWidths()
      this.trackRecord = []
      this.mouseXStart = ev.pageX
      //console.log("grabBar: mousedown at x = " + this.mouseXStart + "; this.barStartX = " + this.barStartX)
      document.addEventListener('mousemove', this.dragBar, true)
      document.addEventListener('mouseup', this.throwBar, true)
    },
    dragBar (ev) {
      // Keeps moving the bar while mouse is down
      ev.stopPropagation()
      var mousex = ev.pageX
      var mouseOffset = mousex - this.mouseXStart
      var dragShift = mouseOffset + this.barStartX
      var maxShift = this.halfDeltaW.magnitude
      var stretchShift = maxShift + 20;
      //console.log("dragbar(): Math.abs(dragShift): "+ Math.abs(dragShift) + "; maxShift: " + maxShift)
      if (Math.abs(dragShift) > stretchShift) { //Dragged mouse past stretch limit
        if (dragShift < stretchShift) {
          this.xShift = -stretchShift
        }
        if (dragShift > stretchShift) {
          this.xShift = stretchShift
        }
        //console.log("dragBar: dragShift = " + dragShift)
        //console.log("this.xShift adjusted to" + this.xShift)
      } else {
        this.xShift = dragShift
        this.trackRecord.push(mousex)
        if (this.trackRecord.length > 4) {
          this.trackRecord.shift()
        }
      }
    },
    throwBar () {
      // Releases the bar, and starts it gliding
      document.removeEventListener('mousemove', this.dragBar, true)
      document.removeEventListener('mouseup', this.throwBar, true)
      var len = this.trackRecord.length
      if (len > 1) { // Only change anything if the mouse has moved since the grabBar()
        this.stillGoing = true
        var dx = (this.trackRecord[len - 1] - this.trackRecord[0])/(len - 1) //
        //console.log("throwBar: this.trackRecord: " + this.trackRecord.toString() + "; mouseup at x = " + ev.pageX + "; step size = " + dx + "; this.xShift = " + this.xShift)
        var limit = this.stepSizeLimit
        if ( dx > limit ) {
          //console.log("Too fast: step size = " + dx+ "; stepSizeLimit = " + limit)
          dx = limit
        }
        this.dx = dx
        if (Math.abs(this.xShift) > this.halfDeltaW.magnitude) { // if let go outside of stretch range, relax to edge
          window.requestAnimationFrame(this.relaxStep)
        }
        else { // start gliding
          this.barStartX = this.xShift
          window.requestAnimationFrame(this.glideStep)
        }
      }
      this.trackRecord = []
    },

    glideStep () {
      // Callback for requestAnimationFrame
      // Moves the bar a bit, reduces its speed gradually based on animation speed parameters in data
      // Allows for hitting end of travel and relaxing back (using this.relaxStep)
      //console.log("glideStep: frictionFac = " + this.frictionFac + "; dx = " + dx )
      if (this.stillGoing) {
        var dx = this.dx
        var stepSize = Math.abs(dx)
        this.xShift = this.xShift + dx
        if (stepSize < this.lowSpeedLimit ) {
          //console.log('hit lowSpeedLimit')
          this.stillGoing = false
          this.barStartX = this.xShift
        }
        var pos = this.xShift
        var maxShift = this.halfDeltaW.magnitude
        if (pos < -maxShift || pos > maxShift ) {
          //console.log("glided past endpoint: xShift = " + this.xShift)
          window.requestAnimationFrame(this.relaxStep)
        }
        else {
          // decrease speed for next loop
          this.dx = dx*(1-this.frictionFac)
          window.requestAnimationFrame(this.glideStep)
        }
      }
    },
    relaxStep () {
      // Callback for requestAnimationFrame()
      // Brings bar to within range if dragged too far. Purely aesthetic to allow it to be dragged a bit past end position and relax to a stop
      var pos = this.xShift
      var maxShift = this.halfDeltaW.magnitude
      var dx = this.relaxStepSize
      var stepSize = Math.abs(dx)
      if (Math.abs(pos) > maxShift)  {
        if (pos < -maxShift){
          dx = stepSize
        }
        if (pos > maxShift) {
          dx = -stepSize
        }
        this.xShift = pos + dx
        window.requestAnimationFrame(this.relaxStep)
      }
      else
      {
        this.stillGoing = false
        if (pos < 0) {this.xShift = -maxShift}
        else {this.xShift = maxShift}
        this.barStartX = this.xShift
      }

    }
  }
}
</script>

<template>
  <div class="chooser-horiz-scroll-frame" :id="frameID" @mousedown.prevent="grabBar">
    <div class="chooser-horiz-scroller" :id="scrollerID" :style="xTransformStyle">
      <div v-for="(targ, index) in targets" :key="index" @mousedown.prevent="tempSelect(targ)" @click.prevent="selectWithClick(targ)" @mouseup.prevent class="chooser-horiz-scroll-item">{{targ}}</div>
    </div>
  </div>
</template>

<style>

  .chooser-horiz-scroll-frame {
    width: 50%;
    align-self: center;
    padding: 5px;
    background: #fff;
    overflow: hidden;
    text-align: center;
    border: 1px solid;
  }

  .chooser-horiz-scroller {
    max-width: 5000%; /* to counter anything set in the embedding document */
    position: relative;
    left: 0;
    top: 0;
    /* max-width: 5rem; */
    margin: 0 -1000%;
    display: inline-flex;
    flex-direction: row;
    justify-content: center;
    /*outline: 1px dashed red;*/
  }

  .chooser-horiz-scroll-item {
    padding: 0 0.2em;
  }
</style>
