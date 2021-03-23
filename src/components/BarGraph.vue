<template>
  <div class="bar-graph" :id="id">
  </div>
</template>

<script>
import * as d3 from 'd3';

const SLIDER_PADDING = 16;
const CANVAS = {
  margin: {
    top: 0,
    right: SLIDER_PADDING/2,
    bottom: 50,
    left: SLIDER_PADDING/2
  }
}


export default {
  name: 'BarGraph',

  data () {
    return {
      axis: {
        x: {},
        y: {}
      },
      ddd: {},
      id: 'd3-' + Math.round(Math.random() * 1000000),
      chart: {
        width: 0,
        height: 100,
        scale: 48,
        background: '',
        blue: '#1861ae',
        red: 'rgb(234 112 94)',
        gray: '#d0d3d4',
        barOffset: 0,
        barPadding: 2,
      }
    };
  },

  props: {
    d3Data: {
      type: Object,
      default: () => ({
        x: [],
        y: [],
        tooltip: []
      })
    },
    graphDimensions: {
      type: Object,
      default: () => ({
        h: 0,
        w: 0,
        scale: 0,
      }),
    },
    desiredIncome: {
      type: Number,
      default: 0,
    }
  },

  computed: {

  },

  methods: {
    makeGraph() {
      this.clearGraph();
      this.initializeGraph();
      this.refreshGraph();
    },

    clearGraph() {
      d3.selectAll(`#${this.id} svg`).remove();
    },

    initializeGraph () {
      d3.select(`#${this.id}`)
        .append('svg')
        .attr('width', this.chart.width)
        .attr('height', this.chart.height + CANVAS.margin.top + CANVAS.margin.bottom)
        // .style('background', this.chart.background);

      this.ddd.svg = d3.select(`#${this.id} svg`);
      if (!this.ddd.tooltip) {
        this.ddd.tooltip = d3.select('body')
          .append('div')
          .attr('class', 'tooltip elevation-3')
          .style('opacity', 0);
      }
    },

    refreshGraph () {
      // X axis
      this.axis.x.values = d3.scaleLinear()
        .domain([d3.min(this.d3Data.x), d3.max(this.d3Data.x)])
        .range([0, this.chart.width-SLIDER_PADDING]);

      // How far apart are the ticks on x axis, e.g. 7 days apart
      this.axis.x.ticks = d3.axisBottom(this.axis.x.values)
        .ticks(5);

      // Setting first, last and gap between bars, note d3DataY is required
      this.axis.x.scale = d3.scaleBand()
        .domain(this.d3Data.y)
        .range([0, this.chart.width]);

      // transform(x, y) specifies where x axis begins, drawn from left to right
      this.ddd.svg.append('g')
        .attr('transform', `translate(${CANVAS.margin.left}, ${CANVAS.margin.top + this.chart.height})`)
        .call(this.axis.x.ticks);

      // Y axis
      // .range specifies value from top left (high number) to bottom left (0)
      this.axis.y.values = d3.scaleLinear()
        .domain([0, d3.max(this.d3Data.y)])
        .range([this.chart.height, 0]);

      // How many ticks are on the y axis
      this.axis.y.ticks = d3.axisLeft(this.axis.y.values)
        .ticks(10);

      this.axis.y.scale = d3.scaleLinear()
        .domain([0, d3.max(this.d3Data.y)])
        .range([0, this.chart.height]);

      this.draw();
      this.addListeners();
    },

    draw () {
      let vm = this;
      // translate(x, y) specifies where bar begins, +1 to move right of y axis
      this.ddd.chart = this.ddd.svg.append('g')
        .attr('transform', `translate(${CANVAS.margin.left + 1}, 0)`)
        .selectAll('rect')
        .data(this.d3Data.y)
        .enter()
        .append('rect');

      this.ddd.chart
        .attr('fill', data => vm.calculateColor(data))
        .attr('width', (data, i, array) => ((vm.chart.width-SLIDER_PADDING) / array.length) - 2)
        .attr('x', (data, i, array) => i * ((vm.chart.width-SLIDER_PADDING) / array.length))
        .attr('height', data => vm.calculateHeight(data))
        .attr('y', data => vm.calculateY(data));
    },

    addListeners () {
      let vm = this;
      this.ddd.chart.on('mouseover', function(yData, index) {
          let tooltipX = d3.event.pageX + 5;
          let tooltipY = d3.event.pageY -50;
          vm.ddd.tooltip.html(vm.d3Data.tooltip[index])
            .style('left', `${tooltipX}px`)
            .style('top', `${tooltipY}px`)
            .style('opacity', 1);

          d3.select(this)
            .style('opacity', .5)
        }).on('mouseout', function() {
          vm.ddd.tooltip.html('')
            .style('opacity', 0);

          d3.select(this)
            .transition()
            .duration(300)
            .style('opacity', 1)
        });
    },

    calculateDimension() {
      this.chart = Object.assign({}, this.chart, {
        width: this.graphDimensions.w,
        height: this.graphDimensions.h - CANVAS.margin.top - CANVAS.margin.bottom,
        scale: this.graphDimensions.scale,
      });
    },
    calculateColor(data) {
      // This is an Assumption graph
      if (48 === this.chart.scale)
        if (0 === data)
          return this.chart.red;
        else
          return this.chart.blue;

      // This is the Results graph 
      if (data >= this.desiredIncome)
        return this.chart.blue;
      else
        return this.chart.gray;
    },
    calculateHeight(data) {
      if (48 === this.chart.scale) {
        if (0 === data) data = 48;
      }
      return this.chart.height*data/this.chart.scale
    },
    calculateY(data) {
      if (48 === this.chart.scale) {
        if (0 === data) data = 48;
      }
      return this.chart.height - this.chart.height*data/this.chart.scale + CANVAS.margin.top
    }
  },

  mounted() {
  },
  watch: {
    graphDimensions() {
      this.calculateDimension();
      this.makeGraph();
    },
    d3Data() {
      this.calculateDimension();
      this.makeGraph();
    }
  }
}
</script>

<style lang="scss">
.bar-graph {
  width: 100%;
  // overflow: hidden;
}

.tooltip {
    position: absolute;
    top: 0px;
    left: 0px;
    padding: 10px 20px;
    border-radius: 3px;
    background: #1861ae;
    // #19264b; // dark blue
    // #d0d3d4; // gray
    color: white;
    z-index: 100;
  }
</style>
