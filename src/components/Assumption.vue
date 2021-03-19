<template>
  <article class="assumption">

    <div class="label">
      <div class="value">{{formattedAssumption}}</div>
      <div class="description">{{assumption.description}}</div>
    </div>

    <SliderChart class="input"
      :graphData="assumption.graphData"
      :graphDimensions="graphDimensions"
      :sliderRange="assumption.sliderRange"
      :sliderValue="assumption.value"
      :name="name"/>

  </article>
</template>

<script>
const FORMATTER = new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD',
  minimumFractionDigits: 0,
});
const PAY_GRADE_MAP = ['JA', 'SA', 'Associate']

import { Bus } from '../bus.js'
import SliderChart from './SliderChart';

export default {
  name: 'Assumption',
  components: {
    SliderChart
  },
  data() {
    return {
      width: null,
    };
  },
  props: {
    assumption: {
      type: Object,
      default: () => {},
    },
    name: {
      type: String,
    }
  },
  computed: {
    graphDimensions() {
      return {
        w: this.width,
        h: 120,
        scale: 48,
      };
    },
    formattedAssumption() {
      let value = this.assumption.value;
      let format = this.assumption.format;
      
      if ('currency' === format)
        return FORMATTER.format(value);

      else if ('percent' === format)
        return Math.round(value) + '%';

      else if ('float' === format)
        return Math.round(value, 1);

      else if ('paygrade' === format)
        return PAY_GRADE_MAP[value];

      else if ('int' === format)
        return Math.round(value);

      else
        return value;
    }
  },
  mounted() {
    let vm = this;
    Bus.$on('resize', (width)=> {
      vm.width = width;
    });
  }
}
</script>

<style scoped lang="scss">
.assumption {
  display: flex;
  flex-flow: row wrap;
  align-items: center;
  max-width: 960px;
  margin: auto;

  .label {
    flex-basis: 160px;
    flex-grow: 1;
    padding-right:30px;
    // padding: 50px 30px 0 40px;

    
    .value {
      font-size: 36px;
      font-weight: normal;
      border-bottom: 1px solid #19264b;
      margin: 0 0 3px;
    }

    .description {
      color: #777;
    }
  }

  .input {
    position: relative;
    flex-basis: 300px;
    flex-grow: 3;
    box-sizing: border-box;
    // padding-left: 30px;
    margin: 0;
  }
}
</style>
