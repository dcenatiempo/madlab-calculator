<template>
  <div class="slider-chart">
    <div class="slider-container">
      <vue-slider
        v-model="value"
        :min="sliderRange.min"
        :max="sliderRange.max"
        :interval="sliderRange.interval"/>
    </div>
    <BarGraph
      :graphDimensions="graphDimensions"
      :d3-data="graphData"/>
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
      value: undefined,
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
        h: 0,
        w: 0,
        scale: 0,
      }),
    },
    sliderRange: {
      type: Object,
      default: () => {}
    },
    sliderValue: {
      type: Number,
    },
    name: {
      type: String,
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
      Bus.$emit('slider-update', { sliderValue, name: this.name });
    }
  }
}
</script>

<style scoped lang="scss">
.slider-chart {
  // overflow: hidden;
  position: relative;

  .slider-container {
    position: absolute;
    top: 70px;
    width: 100%;

    .vue-slider-component.vue-slider-horizontal {
      padding: 0 8px !important;
    }
  }
}
</style>
