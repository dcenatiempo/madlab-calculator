<template>
  <div id="results">
    <div class="results-table">
      <span></span>
      <span>{{goalMonth}}</span>
      <span>{{selectedMonth}}</span>
      <span>Clients</span>
      <span>{{results.tableData.clients[0]}}</span>
      <span>{{results.tableData.clients[1]}}</span>
      <span>Floor Hours/week</span>
      <span>{{results.tableData.hrsPerWeek[0]}}</span>
      <span>{{results.tableData.hrsPerWeek[1]}}</span>
      <span>$/Month</span>
      <span>{{FORMATTER.format(Math.round(results.tableData.dollarsPerMonth[0]))}}/month</span>
      <span>{{FORMATTER.format(Math.round(results.tableData.dollarsPerMonth[1]))}}/month</span>
      <span>$/hour</span>
      <span>{{FORMATTER.format(results.tableData.dollarsPerHour[0])}}/hour</span>
      <span>{{FORMATTER.format(results.tableData.dollarsPerHour[1])}}/hour</span>
      <span>Total Income</span>
      <span>{{FORMATTER.format(Math.round(results.tableData.totalIncome[0]))}}</span>
      <span>{{FORMATTER.format(Math.round(results.tableData.totalIncome[1]))}}</span>
      <span>Pay Grade</span>
      <span>{{results.tableData.payGrade[0]}}</span>
      <span>{{results.tableData.payGrade[1]}}</span>
    </div>
    <SliderChart
      :graphData="results.graphData"
      :graphDimensions="graphDimensions"
      :sliderRange="results.sliderRange"
      :sliderValue="results.selectedMonth"
      name="results"/>
  </div>
</template>

<script>
import { Bus } from '../bus.js';
import SliderChart from './SliderChart';

export default {
  name: 'MadlabResults',
  components: {
    SliderChart
  },
  data() {
    return {
      width: null,
      FORMATTER: new Intl.NumberFormat('en-US', {
        style: 'currency',
        currency: 'USD',
        minimumFractionDigits: 0,
      }),
    };
  },
  props: {
    results: {
      type: Object,
      default: () => {}
    }
  },
  computed: {
    graphDimensions() {
      return {
        w: this.width,
        h: 120,
        scale: Math.max(...this.results.graphData.y),
      };
    },
    goalMonth() {
      let gm = this.results.tableData.goalMonth;
      return 0 == gm ? 'Longer than 36 months' : `Month ${gm}`;
    },
    selectedMonth() {
      let sm = this.results.selectedMonth;
      return `Month ${sm}`;
    }
  },
  mounted() {
    let vm = this;
    Bus.$on('resize', ()=> {
      vm.width = document.querySelector('#results').clientWidth;
    });
  }
}
</script>

<style scoped lang="scss">
  #results {
    position: sticky;
    // height: 150px;
    width: 100%;
    max-width: 960px;
    background: rgba(255, 255, 255, 0.8);
    top: 0;
    z-index: 10;

  .results-table {
    display: grid;
    grid-template-columns: max-content max-content max-content;
    grid-column-gap: 5px;
    grid-row-gap: 1px;

    span {
        padding: 2px 20px;

      &:nth-child(3n+1) {
        // background: transparent;
        text-align: right;
        border-bottom: 1px solid #adadad;
      }
      &:nth-child(3n+2) {
        background: rgb(100, 181, 246);;
        color: white;
        text-align: center;
      }
      &:nth-child(3n+3) {
        // background: rgb(255, 231, 187);
        // color: white;
        border-bottom: 1px solid #adadad;
        text-align: center;
      }
    }
  }
  .slider-chart {
    margin-top: 20px;
  }
}
</style>
