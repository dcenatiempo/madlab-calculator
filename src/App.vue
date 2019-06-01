<template>
  <div id="app">
    <MadlabHeader />
    <h1>When Will I Meet My Goal?</h1>
    <p>The left column shows when your goal is reached.</p>
    <p>The right column show a snapshot of any selected month along your journey.</p>
    <MadlabResults :results="results" :assumptions="assumptions"/>
    <MadlabMain :assumptions="assumptions"/>
    <footer><span>&copy;2019 Madlab Group</span><span>Developed by &nbsp;<a href="http://dcenatiempo.com/" target="_blank">dcenatiempo.com</a></span></footer>
  </div>
</template>

<script>
import MadlabHeader from './components/MadlabHeader.vue';
import MadlabMain from './components/MadlabMain.vue';
import MadlabResults from './components/MadlabResults.vue';
import { Bus } from './bus.js';

const FORMATTER = new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD',
  minimumFractionDigits: 0,
});

const PAY_GRADE_MAP = ['JA1', 'JA2', 'SA1', 'SA2', 'Associate']

const defaultData = {
  x: [],
  y: [],
  tooltip: [],
};

export default {
  name: 'app',
  components: {
    MadlabHeader,
    MadlabMain,
    MadlabResults,
  },
  data() {
    return {
      Months: [],
      results: {
        selectedMonth: 1,
        sliderRange: {
            min: 1,
            max: 48,
            step: 1, 
            interval: 1,
          },
        graphData: defaultData,
        tableData: {
          goalMonth: 0,
          clients: [],
          hrsPerWeek: [],
          dollarsPerMonth: [],
          dollarsPerHour: [],
          totalIncome: [],
          payGrade: [],
          attrition: [],
        },
      },
      assumptions: {
        desiredIncomePerMonth: {
          description: 'Desired Income per Month',
          value: 4000,
          format: 'currency',
          sliderRange: {
            min: 0,
            max: 10000,
            step: 500, 
            interval: 500,
          },
          graphData: defaultData,
        },
        floorHours: {
          description: 'Max Floor Hours per Week',
          value: 30,
          format: 'int',
          sliderRange: {
            min: 5,
            max: 40,
            step: 2,
            interval: 1,
          },
          graphData: defaultData,
        },
        startingPayGrade: {
          description: 'Starting Pay Grade',
          value: 0,
          format: 'paygrade',
          sliderRange: {
            min: 0,
            max: 4,
            step: 1,
            interval: 1,
          },
          graphData: defaultData,
        },
        monthlyRate: {
          description: 'Average Monthly Membership Rate',
          value: 200,
          format: 'currency',
          sliderRange: {
            min: 100,
            max: 400,
            step: 10, // ? 5
            interval: 5,
          },
          graphData: defaultData,
        },
        hourlyRate: {
          description: 'Average PT Hourly Rate',
          value: 75,
          format: 'currency',
          sliderRange: {
            min: 0,
            max: 150,
            step: 5,
            interval: 5,
          },
          graphData: defaultData,
        },
        classHours: {
          description: 'Minimum Class Hours per Week',
          value: 1,
          format: 'int',
          sliderRange: {
            min: 1,
            max: 40,
            step: 2,
            interval: 2,
          },
          graphData: defaultData,
        },
        startClients: {
          description: 'Starting Clients',
          value: 0,
          format: 'int',
          sliderRange: {
            min: 0,
            max: 100,
            step: 5, 
            interval: 5,
          },
          graphData: defaultData,
        },
        newClients1: {
          description: 'Y1: New PT Clients/month',
          value: 2,
          format: 'float',
          sliderRange: {
            min: 0,
            max: 10,
            step: 1, 
            interval: .5,
          },
          graphData: defaultData,
        },
        newClients2: {
          description: 'Y2: New PT Clients/month',
          value: 3,
          format: 'float',
          sliderRange: {
            min: 0,
            max: 10,
            step: 1, 
            interval: .5,
          },
          graphData: defaultData,
        },
        newClients3: {
          description: 'Y3+: New PT Clients/month',
          value: 4,
          format: 'float',
          sliderRange: {
            min: 0,
            max: 10,
            step: 1, 
            interval: .5,
          },
          graphData: defaultData,
        },
        sourceClients1: {
          description: 'Y1: % of Clients Self Sourced',
          value: 10,
          format: 'percent',
          sliderRange: {
            min: 0,
            max: 100,
            step: 5,
            interval: 5,
          },
          graphData: defaultData,
        },
        sourceClients2: {
          description: 'Y2: % of Clients Self Sourced',
          value: 50,
          format: 'percent',
          sliderRange: {
            min: 0,
            max: 100,
            step: 5,
            interval: 5,
          },
          graphData: defaultData,
        },
        sourceClients3: {
          description: 'Y3+: % of Clients Self Sourced',
          value: 80,
          format: 'percent',
          sliderRange: {
            min: 0,
            max: 100,
            step: 5,
            interval: 5,
          },
          graphData: defaultData,
        },
        averagePT: {
          description: 'Average PT Sessions per Client',
          value: 10,
          format: 'int',
          sliderRange: {
            min: 3,
            max: 25,
            step: 1,
            interval: 1,
          },
          graphData: defaultData,
        },
        salesmanship: {
          description: 'Salesmanship',
          value: 1,
          format: 'int',
          sliderRange: {
            min: 1,
            max: 10,
            step: 1,
            interval: 1,
          },
          graphData: defaultData,
        },
        superstar: {
          description: 'Months to Awesome Salemanship',
          value: 36,
          format: 'int',
          sliderRange: {
            min: 1,
            max: 60,
            step: 4,
           interval: 1,
         },
         graphData: defaultData,
        },
      },
    };
  },
  computed: {
    // Months() {
    //   let mode = 'results'
    //   let params = this.getAssumptionSliderValues();
    //   return this.calculate(mode, params);
    // },
    
  },
  methods: {
    getAssumptionSliderValues(currentKey, tickValue) {
      const a = this.assumptions;
      const assumptionKeys = Object.keys(a);

      let values = {};
      assumptionKeys.forEach( key => {
          values[key] =
            key === currentKey
            ? tickValue
            : a[key].value         
      });
      return values;
    },
    getCommission(paygrade, clientsS) {
      let referral = (clientsS / 100) * .1;
      if (paygrade == 0) return 0;
      if (paygrade == 1) return .2 + referral;
      if (paygrade == 2) return .3 + referral;
      if (paygrade == 3) return .35 + referral;
      if (paygrade == 4) return .4 + referral;
      return .4 + referral;
      
    },
    getPaygrade(paygrade, clients) {
      let pg;
      if (clients >= 50) pg = 4;
      else if (clients >= 30) pg = 3;
      else if (clients >= 10) pg = 2;
      else if (clients >= 3) pg = 1;
      else pg = 0;
      return Math.max(pg, paygrade);
    },
    getClientsSource(sourceClients1, sourceClients2, sourceClients3, Months, i) {
      let clientsS;
      if (i+2 > 24) {
        clientsS = sourceClients3;
      }
      else if (i+2 > 12) {
        clientsS = Months[i].clientsS + Math.abs(sourceClients3-sourceClients1) / 12;
      }
      else {
        clientsS = Months[i].clientsS + Math.abs(sourceClients2-sourceClients1) / 12;
      }
      return clientsS
    },
    introToFun(salesmanship) {
      let start = 25; // iF1
      let end = 90; // iF10
      return this.getSalesmanshipValue(start, end, salesmanship);
    },
    funToGroup(salesmanship) {
      let start = 50; // fG1
      let end = 90; // fG10
      return this.getSalesmanshipValue(start, end, salesmanship);
    },
    // averagePT(salesmanship) {
    //   let start = 10; //pF1
    //   let end = 16; //pF10
    //   return this.getSalesmanshipValue(start, end, salesmanship);
    // },
    yearlyAttrition(salesmanship, averagePT) {
      let start = 40; // yA1
      let end = 10; // yA10
      let baseAttrition = this.getSalesmanshipValue(start, end, salesmanship);
      let ptRatio = Math.min(averagePT/20, 1);
      return Math.round(baseAttrition/ptRatio);

    },
    getSalesmanshipValue(start, end, salesmanship) {
      // salesmanship is a value from 1 - 10
      // this returns a value inbetween start and end based on salesmanship
      let increment = (salesmanship - 1) / 9;
      return start - ( (start - end) * increment );
    },
    getCurrentSalesmanship(startingSalesmanship, monthsToSuperstar, currentMonth) {
      let max = 10;
      let increment = (max - startingSalesmanship) / (monthsToSuperstar);
      let currentSalemanship = startingSalesmanship + ( ( currentMonth ) * increment );
      return Math.min(max, currentSalemanship);
    },
    formatGraphData(rawData, sliderRange) {
      let d3Data = {
        x: [],
        y: [],
        tooltip: [],
      };
      rawData.forEach((val, i, array) => {
        let interval = (sliderRange.max - sliderRange.min) / (array.length - 1)
        d3Data.x.push(sliderRange.min + (i * interval));
        d3Data.y.push(val);
        d3Data.tooltip.push(val);
      });
      return d3Data;
    },
    calculateAssumptionGraphData() {
      let vm = this;
      let a = this.assumptions;
      const assumptionKeys = Object.keys(a);

      // loop through each assumption
      assumptionKeys.forEach( key => {
        let start = a[key].sliderRange.min;
        let end = a[key].sliderRange.max;
        let step = a[key].sliderRange.step;
        let n = start;
        let xAxisTicks = [start];
        let rawData = [];
        
        while (n < end) {
            xAxisTicks.push(n + step);
            n += step;
        }

        // loop through each bar on the graph for the assumption to calculate the bar height
        xAxisTicks.forEach(tickValue => {
          let mode = 'assumptions';
          let params = vm.getAssumptionSliderValues(key, tickValue);
          rawData.push(vm.calculate(mode, params));
        });

        // format assumption graph data
        vm.assumptions[key].graphData = vm.formatGraphData(rawData, a[key].sliderRange);
        vm.assumptions[key].graphData.tooltip = vm.assumptions[key].graphData.tooltip.map(val => 0 === val ? '48+ Months' : `${val} Months`);
      });
    },
    // Based on assumptions, calculate 36 months worth of data
    calculate( mode, {
      desiredIncomePerMonth,
      floorHours,
      startingPayGrade,
      monthlyRate,
      hourlyRate,
      classHours,
      startClients,
      newClients1,
      newClients2,
      newClients3,
      sourceClients1,
      sourceClients2,
      sourceClients3,
      averagePT,
      salesmanship,
      superstar }) {

      let Months = [];
      let month = {
        introHours: undefined,
        introToFun: this.introToFun(salesmanship),
        funToGroup: this.funToGroup(salesmanship),
        yearlyAttrition: this.yearlyAttrition(salesmanship, averagePT),
        newClients: newClients1,
        fundamentalsClients: newClients1,
        classClients: startClients,
        ptBucket: 0,
        ptHours: undefined,
        hoursWeek: undefined,
        clientsS: sourceClients1,
        revenue: undefined,
        incomeM: undefined,
        incomeH: undefined,
        incomeT: 0,
        payGrade: undefined,
      }

      month.introHours = month.newClients / month.introToFun * 100;
      let availablePtHours = (floorHours - classHours)*4.3 - month.introHours;
      month.ptBucket += month.newClients * averagePT;
      month.ptHours = Math.min(availablePtHours, month.ptBucket);
      month.ptBucket -= month.ptHours;
      month.hoursWeek = (month.ptHours + classHours*4.3 + month.introHours)/4.3;
      let clientsFinishedPT = month.ptHours/averagePT;
      month.fundamentalsClients -= clientsFinishedPT;
      month.fundamentalsClients = Math.round(month.fundamentalsClients, 2);
      month.classClients += clientsFinishedPT * (month.funToGroup / 100);

      // income
      month.revenue = monthlyRate * month.classClients + hourlyRate * month.ptHours; 
      month.payGrade = this.getPaygrade(startingPayGrade, month.classClients);
      let commission = this.getCommission(month.payGrade, month.clientsS);
      month.incomeM = month.revenue * commission;
      month.incomeH = month.incomeM / (month.hoursWeek * 4.3);
      month.incomeT += month.incomeM;
      
      Months.push({...month});

      let i = 0;
      let goalMonth = 0;
      let checkGoals = 0;  // 'assumptions' = mode

      // Did we meet our goal in this month?
      if (goalMonth == 0 && month.incomeM > desiredIncomePerMonth) {
        goalMonth = i+1;
        checkGoals = 'results' === mode ? 0 : goalMonth;
      }
        
      //SUBSEQUENT MONTHS
      while (i < 47 && 0 === checkGoals) {
        let currentSalesmanship = this.getCurrentSalesmanship(salesmanship, superstar, i+1);
        month.introToFun = this.introToFun(currentSalesmanship);
        month.funToGroup = this.funToGroup(currentSalesmanship);
        month.yearlyAttrition = this.yearlyAttrition(currentSalesmanship, averagePT);
        
        if (i+2 > 24)
          month.newClients = newClients3;
        else if (i+1 == 12)
          month.newClients =  newClients2;
        else if (i+2 > 12)
          month.newClients =  (newClients3 - newClients2) / 12 + Months[i].newClients;
        else
          month.newClients =  (newClients2 - newClients1) / 12 + Months[i].newClients;
        
        month.fundamentalsClients += month.newClients;

        // possibly override class hours
        let currentClassHours = Math.max(Months[i].classClients / 6 , classHours);

        month.introHours = month.newClients / month.introToFun * 100;
        availablePtHours = (floorHours - currentClassHours)*4.3 - month.introHours;
        availablePtHours = availablePtHours <= 0 ? 0 : availablePtHours;
        month.ptBucket += month.newClients * averagePT;
        month.ptHours = Math.min(availablePtHours, month.ptBucket);
        month.ptBucket -= month.ptHours;

        month.hoursWeek = (month.ptHours + currentClassHours*4.3 + month.introHours)/4.3;
        clientsFinishedPT = month.ptHours/averagePT;
        month.fundamentalsClients -= clientsFinishedPT;
        month.fundamentalsClients = Math.round(month.fundamentalsClients, 2);

        // Move clients from fundamentals to class
        month.classClients += clientsFinishedPT * (month.funToGroup / 100);
        
        // Drop clients due to attrition
        let clientsLost = Months[i].classClients * (Months[i].yearlyAttrition/100) / 12;
        month.classClients -= clientsLost;        
        
        // income
        month.revenue = monthlyRate * month.classClients + hourlyRate * month.ptHours; 

        month.clientsS = this.getClientsSource(sourceClients1, sourceClients2, sourceClients3, Months, i);
        month.payGrade = this.getPaygrade(startingPayGrade, month.classClients);
        commission = this.getCommission(month.payGrade, month.clientsS);
        month.incomeM = month.revenue * commission;
        month.incomeH = month.incomeM / (month.hoursWeek * 4.3);
        month.incomeT += month.incomeM;
        
        Months.push({...month});	

        if (goalMonth == 0 && month.incomeM > desiredIncomePerMonth) {
          goalMonth = i+2;
          checkGoals = 'results' === mode ? 0 : goalMonth; // 'assumptions' = mode
        }
        if (goalMonth == 0 && i == 34) {  // 'assumptions' = mode
          checkGoals = 'results' === mode ? 0 : goalMonth;
        }
        
        i++;
      }

      if ('results' === mode) {
        this.results.tableData.goalMonth = goalMonth;
        this.Months = Months;
        this.setRawResultsGraphData(Months, this.results.selectedMonth);
      }
      if ('assumptions' === mode) {
        return checkGoals;
      }
      return 'Error';
    },
    setRawResultsGraphData(Months, selectedMonth) {
      let tD = this.results.tableData;
      let gM = tD.goalMonth >= 48 ? 47 : tD.goalMonth-1;
      let sM = selectedMonth - 1;
      // goal month
      tD.clients[0] = -1 === gM ? 'N/A' : Math.round(Months[gM].classClients);
      tD.hrsPerWeek[0] = -1 === gM ? 'N/A' : Math.round(Months[gM].hoursWeek * 10) / 10;
      tD.dollarsPerMonth[0] = -1 === gM ? 'N/A' : Months[gM].incomeM;
      tD.dollarsPerHour[0] = -1 === gM ? 'N/A' : Months[gM].incomeH;
      tD.totalIncome[0] = -1 === gM ? 'N/A' : Months[gM].incomeT;
      tD.payGrade[0] = -1 === gM ? 'N/A' : PAY_GRADE_MAP[Months[gM].payGrade];
      tD.attrition[0] = -1 === gM ? 'N/A' : Months[gM].yearlyAttrition;
      // month selected
      tD.clients[1] = Math.round(Months[sM].classClients);
      tD.hrsPerWeek[1] = Math.round(Months[sM].hoursWeek * 10) / 10;
      tD.dollarsPerMonth[1] = Months[sM].incomeM;
      tD.dollarsPerHour[1] = Months[sM].incomeH;
      tD.totalIncome[1] = Months[sM].incomeT;
      tD.payGrade[1] = PAY_GRADE_MAP[Months[sM].payGrade];
      tD.attrition[1] = Months[sM].yearlyAttrition;
        
      // build graph
      let rawData = Months.map(month => month.incomeM);
      this.results.graphData = this.formatGraphData(rawData, this.results.sliderRange)
      this.results.graphData.tooltip = this.results.graphData.tooltip.map( item => FORMATTER.format(Math.round(item)));
    },
  },
  watch: {
    assumptions() {
      let params = this.getAssumptionSliderValues();
      this.calculate('results', params);
      this.calculateAssumptionGraphData();
    },
    'assumptions.classHours.value'(hours) {
      if (hours >= 5)
        this.assumptions.floorHours.sliderRange.min = hours;
    }
  },
  mounted() {
    let vm = this;
    let timer;
    window.onresize = function resize() {
      clearTimeout( timer );
      timer = setTimeout( () => {
        Bus.$emit('resize', document.querySelector('#sizing-template .right').offsetWidth);
      }, 25 );
    };

    Bus.$on('slider-update', ({sliderValue, name, initial}) => {
      if (initial) return;

      if ('results' === name) {
        vm.results.selectedMonth = sliderValue;
        vm.setRawResultsGraphData(vm.Months, sliderValue);
        vm.results = Object.assign({}, vm.results);
      } else {
        vm.assumptions[name].value = sliderValue;
        vm.assumptions = Object.assign({}, vm.assumptions);
      }
    });

    // initial width
    Bus.$emit('resize', document.querySelector('#sizing-template .right').offsetWidth);

    // initial calculation
    let params = this.getAssumptionSliderValues();
    this.calculate('results', params);
    this.calculateAssumptionGraphData();
  },
}
</script>

<style lang="scss">
* {
  box-sizing: border-box;
}
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  padding: 20px;
  max-width: 960px;
  width: 100vw;
  margin: auto;

  footer {
    padding: 5px;
    display: grid;
    text-align: center;
  }
}
</style>
