<template>
  <div class="slider-chart">
    <div class="slider-container">
      <vue-slider
        v-model="value"
        :min="sliderRange.min"
        :max="sliderRange.max"
        tooltip-style="border-color: #19264b; background-color: #19264b"
        :interval="sliderRange.interval"/>
    </div>
    <BarGraph
      :graphDimensions="graphDimensions"
      :d3-data="graphData"
      :desiredIncome="desiredIncome"/>
  </div>
</template>

<script>
import { Bus } from '../bus.js';
import vueSlider from 'vue-slider-component/src/vue2-slider.vue';
import BarGraph from './BarGraph';

export default {
  name: 'SliderChart',
  components: {
    'vue-slider': vueSlider,
    BarGraph,
  },
  data() {
    return {
      value: 1,
      initial: true,
      // height: 150,
      // width: null,
    };
  },
  props: {
    graphData: {
      type: Object,
      default: () => ({
        x: [],
        y: [],
        tooltip: []
      }),
    },
    graphDimensions: {
      type: Object,
      default: () => ({
        h: 1,
        w: 1,
        scale: 1,
      }),
    },
    sliderRange: {
      type: Object,
      default: () => ({
        min: 1,
        max: 2,
        step: 1, 
        interval: 1,
      })
    },
    sliderValue: {
      type: Number,
      default: 1,
    },
    name: {
      type: String,
    },
    desiredIncome: {
      type: Number,
      default: 0,
    }
  },
  computed: {},
  methods: {
  },
  mounted() {
    this.value = this.sliderValue;
  },
  watch: {
    value(sliderValue) {
      Bus.$emit('slider-update', { sliderValue, name: this.name, initial: this.initial });
      this.initial = false;
    }
  }
}
</script>

<style lang="scss">
.slider-chart {
  // overflow: hidden;
  position: relative;

  .slider-container {
    position: absolute;
    top: 70px;
    width: 100%;

    .vue-slider-component .vue-slider-process {
      background-color: #0757c3;
    }
    .vue-slider-component.vue-slider-horizontal {
      padding: 0 8px !important;
    }
  }
}
</style>
