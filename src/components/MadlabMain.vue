<template>
  <main>
    <h1>Assumptions:</h1>
    <section>
      <h2>Goals</h2>
      <p>Without a destination, it doesn't matter where you steer the ship. Set some realistic figures to shoot for, and the calculator will tell you how long it will take to get there.</p>
      <Assumption
        name="desiredIncomePerMonth"
        :assumption="assumptions.desiredIncomePerMonth"/>
      <Assumption
        name="floorHours"
        :assumption="assumptions.floorHours"/>
    </section>

    <section>
      <h2>Gym's Determinants</h2>
      <p>Not everything is in direct control of the coach. The following items are determined by the gym owners and your progress in the PCDP as you become a professional coach.</p>
      <Assumption
        name="startingPayGrade"
        :assumption="assumptions.startingPayGrade"/>
        <a class="paygrade-link" @click="showPaygradeModal">view details</a>
      <Assumption
        name="monthlyRate"
        :assumption="assumptions.monthlyRate"/>
      <Assumption
        name="hourlyRate"
        :assumption="assumptions.hourlyRate"/>
      <Assumption
        name="classHours"
        :assumption="assumptions.classHours"/>
    </section>

    <section>
      <h2>New Clients</h2>
      <p>Are you starting with any clients? Great! Now how many clients can you bring in each month? No clients equals no money, no matter how much you charge!</p>
      <Assumption
        name="startClients"
        :assumption="assumptions.startClients"/>
      <Assumption
        name="newClients1"
        :assumption="assumptions.newClients1"/>
      <Assumption
        name="newClients2"
        :assumption="assumptions.newClients2"/>
      <Assumption
        name="newClients3"
        :assumption="assumptions.newClients3"/>
      <Assumption
        name="sourceClients1"
        :assumption="assumptions.sourceClients1"/>
      <Assumption
        name="sourceClients2"
        :assumption="assumptions.sourceClients2"/>
      <Assumption
        name="sourceClients3"
        :assumption="assumptions.sourceClients3"/>
    </section>

    <section>
      <h2>Retention / Conversion</h2>
      <p>Your retention and conversion rates play a huge role in how many hours you work compared with your revenue produced. Better retention equals more money with less work.</p>
      <Assumption
        name="averagePT"
        :assumption="assumptions.averagePT"/>
      <Assumption
        name="salesmanship"
        :assumption="assumptions.salesmanship"/>
      <!-- Sizing template is serving double duty:
      1) provide right sizing for graphs
      2) custom salesmanship x-axis label -->
      <div id="sizing-template">
        <div class="left">
          <div :class="{hide}"><br>Intro ⇨ Fund<br>Fund ⇨ Group<br>Yearly Attrition Base</div> 
        </div>
        <div class="right">
          <div>Poor</div>
          <div>Average</div>
          <div>Awesome</div>
          <div :class="{hide}">25%<br>50%<br>40%</div>
          <div :class="{hide}">60%<br>70%<br>25%</div>
          <div :class="{hide}">90%<br>90%<br>10%</div>
        </div>
      </div>
      <Assumption
        name="superstar"
        :assumption="assumptions.superstar"/>
    </section>
    <PaygradeModal :is-visible="paygradeModalVisible" @close="paygradeModalVisible = false"/>

  </main>
</template>

<script>
import Assumption from './Assumption';
import PaygradeModal from './PaygradeModal';
import { Bus } from '../bus.js';

export default {
  name: 'MadlabMain',
  components: {
    Assumption,
    PaygradeModal,
  },
  props: {
    assumptions: {
      type: Object,
      default: () => {},
    }
  },
  data() {
    return {
      d3Data: {},
      right: undefined,
      left: undefined,
      hide: true,
      paygradeModalVisible: false,
    }
  },
  methods: {
    resize() {
      if (this.right.clientWidth === this.left.clientWidth)
        this.hide = true
      else this.hide = false
    },
    showPaygradeModal() {

      console.log('pizza')
      this.paygradeModalVisible = true;
    }
  },
  mounted() {
    this.right = document.querySelector("#sizing-template .right");
    this.left = document.querySelector("#sizing-template .left");
    Bus.$on('resize', this.resize);
    this.resize();
  },
}
</script>

<style lang="scss">
#sizing-template {
  display: flex;
  flex-flow: row wrap;
  max-width: 960px;
  margin-top: -30px;
  color: #777;
  .hide {
    height: 0;
    overflow: hidden;
  }

  .left {
    // background-color: red;
    flex-basis: 160px;
    flex-grow: 1;
    padding-right: 30px;
  }
  .right {
    // background-color: green;
    flex-basis: 300px;
    flex-grow: 4;
    display: grid;
    grid-template-columns: max-content max-content max-content;
    justify-content: space-between;
    > div {
      text-align: center;
    }
  }
}
.paygrade-link {
  position: relative;
  top: -24px;
  height: 0;
  display: block;
  cursor: pointer;
}

</style>
