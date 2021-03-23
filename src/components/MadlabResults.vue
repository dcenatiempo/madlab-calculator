<template>
  <div id="results" :class="{'with-transition': withTransition, expanded }" :style="`top: -${top}px`">
    <div id="results-sizing-template">
      <div class="left"></div>
      <div class="right"></div>
    </div>
    <div class="wrapper">
      <div class="results-table" :class="{'no-goal': 0 === results.tableData.goalMonth}">
        <span></span>
        <span>{{goalMonthText}}</span>
        <span>{{selectedMonth}}</span>
        <span>Clients</span>
        <span>{{format(results.tableData.clients[0])}}</span>
        <span>{{format(results.tableData.clients[1])}}</span>
        <span>Floor hrs/wk</span>
        <span>{{format(results.tableData.hrsPerWeek[0])}}</span>
        <span>{{format(results.tableData.hrsPerWeek[1])}}</span>
        <span>$/month</span>
        <span>{{format(FORMATTER.format(Math.round(results.tableData.dollarsPerMonth[0])))}}</span>
        <span>{{format(FORMATTER.format(Math.round(results.tableData.dollarsPerMonth[1])))}}</span>
        <span>$/hour</span>
        <span>{{format(FORMATTER.format(results.tableData.dollarsPerHour[0]))}}</span>
        <span>{{format(FORMATTER.format(results.tableData.dollarsPerHour[1]))}}</span>
        <span>Total Income</span>
        <span>{{format(FORMATTER.format(Math.round(results.tableData.totalIncome[0])))}}</span>
        <span>{{format(FORMATTER.format(Math.round(results.tableData.totalIncome[1])))}}</span>
        <span>Pay Grade</span>
        <span>{{format(results.tableData.payGrade[0])}}</span>
        <span>{{format(results.tableData.payGrade[1])}}</span>
        <span>Est. Attrition</span>
        <span>{{format(results.tableData.attrition[0])}}%</span>
        <span>{{format(results.tableData.attrition[1])}}%</span>
      </div>
      <SliderChart
        :graphData="results.graphData"
        :graphDimensions="graphDimensions"
        :sliderRange="results.sliderRange"
        :sliderValue="results.selectedMonth"
        name="results"
        :desiredIncome="assumptions.desiredIncomePerMonth.value"/>
    </div>
    <div class="handle" :class="{expanded}" @click="expandResults"><div>&raquo;</div></div>
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
      top: 0,
      ref: undefined,
      withTransition: true,
      expanded: true,
    };
  },
  props: {
    results: {
      type: Object,
      default: () => {}
    },
    assumptions: {
      type: Object,
      default: () => {},
    }
  },
  computed: {
    graphDimensions() {
      return {
        w: this.width,
        h: 120,
        // scale: Math.max(...this.results.graphData.y, this.assumptions.desiredIncomePerMonth.value),
        // scale: this.assumptions.desiredIncomePerMonth.value,
        scale: 10000,
      };
    },
    goalMonthText() {
      let gm = this.results.tableData.goalMonth;
      return 0 == gm ? '3+ Years' : `Month ${gm}`;
    },
    selectedMonth() {
      let sm = this.results.selectedMonth;
      return `Month ${sm}`;
    },
  },
  methods: {
    format(val) {
      if (!val) return '?';
      return val.toString().includes('N/A') || val.toString().includes('NaN')
        ? '?'
        : val;
    },
    expandResults(shouldToggle = true) {
      let isHidden = this.top !== 0;

      if (shouldToggle) {
        this.top = Math.abs(this.top - this.ref.clientHeight + 16);
        this.expanded = !this.expanded;
      }
      else if (isHidden) {
        this.top = this.ref.clientHeight - 16;
      }
      window.scrollTo(window.scrollX, window.scrollY);
    }
  },
  mounted() {
    let vm = this;
    this.ref = document.querySelector("#results");
    this.width = document.querySelector('#results-sizing-template .right').clientWidth;
    setTimeout(() => { window.dispatchEvent(new Event('resize'));}, 0);
    Bus.$on('resize', ()=> {
      vm.width = document.querySelector('#results-sizing-template .right').clientWidth;
      vm.expandResults(false)
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
  background: rgba(255, 255, 255, 0.9);
  top: 0;
  z-index: 10;
  margin-top: -10px;
  padding-top: 10px;
 
  &.with-transition {
    // TODO: currently transitioning top property is very laggy
    // Need to switch to a different method which will be difficult with a sticky header...
    transition: all ease .2s;
  }

  .wrapper {
    display: flex;
    flex-flow: row wrap;
  }

  .results-table {
    display: grid;
    grid-template-columns: max-content max-content max-content;
    grid-column-gap: 5px;
    grid-row-gap: 1px;
    padding-right: 30px;
    margin: auto;

    &.no-goal {
      span:nth-child(3n+2) {
        // background: rgb(243, 151, 151);
        background: #ccc;
      }
    }

    span {
        padding: 2px 12px;

      &:nth-child(3n+1) {
        // background: transparent;
        text-align: right;
        border-bottom: 1px solid #adadad;
      }
      &:nth-child(3n+2) {
        background: #1861ae;
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
    bottom: -50px;
    align-self: flex-end;
  }
  .handle {
    height: 30px;
    width: 100%;
    background: rgba(255, 255, 255, 0.9);
    border: none;
    position: relative;
    bottom: -16px;
    display: flex;
    justify-content: center;
    cursor: pointer;

    div {
      font-size: 48px;
      line-height: 24px;
      transform: rotate(90deg);
    }
    &.expanded div {
      transform: rotate(-90deg);
    }
  }
  #results-sizing-template {
  display: flex;
  flex-flow: row wrap;
  max-width: 960px;
    .left {
      height: 0px;
      // background-color: forestgreen;
      flex-basis: 360px;
      padding-right: 30px;
    }
    .right {
      height: 0px;
      // background-color: orangered;
      flex-basis: 300px;
      flex-grow: 4;
    }
  }
}
</style>
