<template>
  <div id="app">
    <MadlabHeader />
    <h1>When Will I Meet My Goal?</h1>
    <p>The bluecolumn shows when your goal is reached.</p>
    <p>The right column show a snapshot of any selected month along your journey.</p>
    <MadlabResults :results="results" :assumptions="assumptions"/>
    <MadlabMain :assumptions="assumptions"/>
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
      results: {
        selectedMonth: 1,
        sliderRange: {
            min: 1,
            max: 36,
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
        },
      },
      assumptions: {
        desiredIncomePerMonth: {
          description: 'Desired Income per Month',
          value: 3000,
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
          value: 1,
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
    Months() {
      let mode = 'results'
      let params = this.getAssumptionSliderValues();
      return this.calculate(mode, params);
    },
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
    getCommission(paygrade) {
      if (paygrade == 0) return 0;
      if (paygrade == 1) return .2;
      if (paygrade == 2) return .3;
      if (paygrade == 3) return .35;
      if (paygrade == 4) return .4;
      return .4
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
    ptPerFun(salesmanship) {
      let start = 10; //pF1
      let end = 16; //pF10
      return this.getSalesmanshipValue(start, end, salesmanship);
    },
    yearlyAttrition(salesmanship) {
      let start = 40; // yA1
      let end = 10; // yA10
      return this.getSalesmanshipValue(start, end, salesmanship);
    },
    getSalesmanshipValue(start, end, salesmanship) {
      // salesmanship is a value from 1 - 10
      // this returns a value inbetween start and end based on salesmanship
      let increment = (salesmanship - 1) / 9;
      return start - ( (start - end) * increment );
    },
    getCurrentSalesmanship(startingSalesmanship, monthsToSuperstar, currentMonth) {
      let max = 10;
      let increment = (max - startingSalesmanship) / (monthsToSuperstar - 1);
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
        vm.assumptions[key].graphData.tooltip = vm.assumptions[key].graphData.tooltip.map(val => 0 === val ? '36+ Months' : `${val} Months`);
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
      salesmanship,
      superstar }) {

      let introToFun = this.introToFun(salesmanship);
      let funToGroup = this.funToGroup(salesmanship);
      let ptPerFun = this.ptPerFun(salesmanship);
      let yearlyAttrition = this.yearlyAttrition(salesmanship);

      let fundamentalsClients = Math.min( ( floorHours - classHours ) / ( ( 1 / introToFun ) + ( ptPerFun / 4.3 )) , newClients1);
      fundamentalsClients = fundamentalsClients < 0 ? 0 : fundamentalsClients;
      let introHours = fundamentalsClients / introToFun * 100;
      let Clients = startClients + newClients1;
      let newClients = Clients;
      let ptHours = (fundamentalsClients * ptPerFun) / 4.3;
      let HoursWeek = ptHours + classHours + introHours;
      let clientsS = sourceClients1;

      let PayGrade = this.getPaygrade(startingPayGrade, Clients);
      let Commission = this.getCommission(PayGrade);
      let actualIncomePerMonth = (monthlyRate * Clients + hourlyRate * ptPerFun * fundamentalsClients) * (Commission + (clientsS / 100) * .1);
      let IncomeH = actualIncomePerMonth / (HoursWeek * 4.3);
      let IncomeT = actualIncomePerMonth;
      
      let Months = [];
      Months.push({
          introHours,
          introToFun,
          fundamentalsClients,
          funToGroup,
          yearlyAttrition,
          Clients,
          ptPerFun,
          ptHours,
          classHours,
          HoursWeek,
          clientsS,
          actualIncomePerMonth,
          IncomeH,
          IncomeT,
          PayGrade,
          month: 1,
          newClients
      });

      let i = 0;
      let goalMonth = 0;
      let noHours = 0;
      let checkGoals = 0;  // 'assumptions' = mode

      if (goalMonth == 0 && actualIncomePerMonth > desiredIncomePerMonth) {
        goalMonth = i+1;
        checkGoals = 'results' === mode ? 0 : goalMonth;
      }
      if (actualIncomePerMonth <= 0 || introHours <= 0 || fundamentalsClients <= 0 || ptHours <= 0) {
        noHours = 1;
      }
        

      //SUBSEQUENT MONTHS
      while (i < 35 && 0 === checkGoals) {
        let currentSalesmanship = this.getCurrentSalesmanship(salesmanship, superstar, i+1);
        ptPerFun = this.ptPerFun(currentSalesmanship);
        introToFun = this.introToFun(currentSalesmanship);
        funToGroup = this.funToGroup(currentSalesmanship);
        yearlyAttrition = this.yearlyAttrition(currentSalesmanship);
        if (i+2 > 24)
          newClients = newClients3;
        else if (i+1 == 12)
          newClients =  newClients2;
        else if (i+2 > 12)
          newClients =  (newClients3 - newClients2) / 12 + Months[i].newClients;
        else
          newClients =  (newClients2 - newClients1) / 12 + Months[i].newClients;

        let currentClassHours = Math.max(Months[i].Clients / 4 , classHours);

        fundamentalsClients = Math.min( ( floorHours - currentClassHours ) / ( ( 1 / introToFun ) + ( ptPerFun / 4.3 )) , newClients);
        fundamentalsClients = fundamentalsClients < 0 ? 0 : fundamentalsClients;

        introHours = fundamentalsClients / introToFun * 100;
        
        Clients = Months[i].fundamentalsClients * (Months[i].funToGroup/100) + Months[i].Clients * (1 - (Months[i].yearlyAttrition/100) / 12);

        ptHours = (fundamentalsClients * ptPerFun) / 4.3;

        HoursWeek = ptHours + currentClassHours + introHours;

        let clientsS = this.getClientsSource(sourceClients1, sourceClients2, sourceClients3, Months, i);

        PayGrade = this.getPaygrade(startingPayGrade, Clients);

        Commission = this.getCommission(PayGrade);
        
        actualIncomePerMonth = (monthlyRate * Clients + hourlyRate * ptPerFun * fundamentalsClients) * (Commission + (clientsS / 100) * .1);

        IncomeH = actualIncomePerMonth / (HoursWeek * 4.3);

        IncomeT = actualIncomePerMonth + Months[i].IncomeT;

        if (goalMonth == 0 && actualIncomePerMonth > desiredIncomePerMonth) {
          goalMonth = i+2;
          checkGoals = 'results' === mode ? 0 : goalMonth; // 'assumptions' = mode
        }
        if (actualIncomePerMonth <= 0 || introHours <= 0 || fundamentalsClients <= 0 || ptHours <= 0) {
          noHours = 1;
        }
        if (goalMonth == 0 && i == 34) {  // 'assumptions' = mode
            checkGoals = 'results' === mode ? 0 : goalMonth;
          }
        
        //add data from current month to array
        Months.push({
            introHours,
            introToFun,
            fundamentalsClients,
            funToGroup,
            yearlyAttrition,
            Clients,
            ptPerFun,
            ptHours,
            currentClassHours,
            HoursWeek,
            clientsS,
            actualIncomePerMonth,
            IncomeH,
            IncomeT,
            PayGrade,
            month: i+2,
            newClients,
        });	
        i++;
      }

      if ('results' === mode) {
        this.results.tableData.goalMonth = goalMonth;
        this.results.noHours = noHours;
        this.setRawResultsGraphData(Months);

        return Months;
      }
      if ('assumptions' === mode) {
        return checkGoals;
      }
      return 'Error';
    },
    setRawResultsGraphData(Months) {
      
      let tD = this.results.tableData;
      let gM = tD.goalMonth >= 36 ? 35 : tD.goalMonth;
      let sM = this.results.selectedMonth - 1;
      // goal month
      tD.clients[0] = 0 === gM ? 'N/A' : Math.round(Months[gM].Clients);
      tD.hrsPerWeek[0] = 0 === gM ? 'N/A' : Math.round(Months[gM].HoursWeek * 10) / 10;
      tD.dollarsPerMonth[0] = 0 === gM ? 'N/A' : Months[gM].actualIncomePerMonth;
      tD.dollarsPerHour[0] = 0 === gM ? 'N/A' : Months[gM].IncomeH;
      tD.totalIncome[0] = 0 === gM ? 'N/A' : Months[gM].IncomeT;
      tD.payGrade[0] = 0 === gM ? 'N/A' : PAY_GRADE_MAP[Months[gM].PayGrade];
      // month selected
      tD.clients[1] = Math.round(Months[sM].Clients);
      tD.hrsPerWeek[1] = Math.round(Months[sM].HoursWeek * 10) / 10;
      tD.dollarsPerMonth[1] = Months[sM].actualIncomePerMonth;
      tD.dollarsPerHour[1] = Months[sM].IncomeH;
      tD.totalIncome[1] = Months[sM].IncomeT;
      tD.payGrade[1] = PAY_GRADE_MAP[Months[sM].PayGrade];
        
      // build graph
      let rawData = Months.map(month => month.actualIncomePerMonth);
      this.results.graphData = this.formatGraphData(rawData, this.results.sliderRange)
      this.results.graphData.tooltip = this.results.graphData.tooltip.map( item => FORMATTER.format(Math.round(item)));
    },
  },
  watch: {
    Months() {
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

    Bus.$on('slider-update', ({sliderValue, name}) => {
      if ('results' === name)
        vm.results.selectedMonth = sliderValue;
      else {
        vm.assumptions[name].value = sliderValue;
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
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  padding: 20px;
  max-width: 960px;
  margin: auto;
}
</style>
