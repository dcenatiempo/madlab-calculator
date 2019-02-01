<template>
  <article class="assumption">

    <div class="label">
      <div class="value">{{assumption.value}}</div>
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
        scale: 36,
      };
    },
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

  .label {
    flex-basis: 160px;
    flex-grow: 1;
    padding-right:30px;
    // padding: 50px 30px 0 40px;

    
    .value {
      font-size: 36px;
      font-weight: normal;
      color: #000;
      border-bottom: 1px solid #000;
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
