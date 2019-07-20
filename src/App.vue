<script>
import ChooserHorizScroll from './components/ChooserHorizScroll.vue'
// import {genGradients} from './mixins/genGradients.js'
export default {
  name: 'DateSortDrag',
  components: {ChooserHorizScroll},
  // mixins: [genGradients],
  data: function () {
    return {
      mouseExited: false,
      selectedTarg: null, // the selected value
      tempTarg: null, // temp selected value while dragging
      lastSelected: null // previous selected value
      }
  },
  props: {
    currentDate: Object,
    sortYearsRange: {
      default () { return [1990, 2010]},
      type: Array
    }
  },
  computed: {
    timeYears: function () { // Takes the first and last year from sortYearsRange and fills in the rest
      var years = []
      for (var i = this.sortYearsRange[0]; i <= this.sortYearsRange[1]; i++) {
        years.push(i)
      }
      return years
    }
  },
  methods: {
    mouseLeaveMethod() {
      this.mouseExited = true
    },
    mouseEnterMethod() {
      this.mouseExited = false
    },
    updateTempTarg(targ) {
      this.tempTarg = targ
    },
    acceptTarget(targ) {
      this.lastSelected = this.selectedTarg
      this.selectedTarg = targ
    }
  }
}
</script>

<template>
<div id="test-div" @mouseenter="mouseEnterMethod" @mouseleave="mouseLeaveMethod">
  <div id="readout-div">
    <div id="display-last-selected">Last selection: {{lastSelected}} </div>
    <div id="display-temp-selected">Temporary selection: {{tempTarg}} </div>
    <div id="display-selected">Current selection: {{selectedTarg}} </div>
  </div>
  <ChooserHorizScroll :mouseOutOfBounds="mouseExited" :targets="timeYears" @select-temptarg="updateTempTarg" @select-target="acceptTarget"></ChooserHorizScroll>
</div>
</template>

<style>
#sort-div {
  display: flex;
  flex-direction: column;
  padding: 0 0.2rem;
  overflow: hidden;
  justify-content: flex-start;
}
#test-div {
  display: flex;
  flex-direction: column;
  overflow: hidden;
  justify-content: flex-start;
}
</style>
