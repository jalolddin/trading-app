<template>
  <div class="home">
    <form action="#">

      <div class="block">
        <span>Деньги на входе</span>  
        <v-text-field v-model="money" type="number" ></v-text-field>
      </div>
      <div class="block"> 
        <span>Компании</span>
        <v-select
        clearable
        :items="['SBER', 'YNDX', 'GAZP', 'OZON', 'TATN']"
        multiple
        v-model="companies"
      ></v-select>
      </div>
      <div class="block"> 
        <span>Период времени</span>
        <v-text-field type="text"></v-text-field>
      </div>
      <button class="submit" type="button" @click="calculate">Рассчитать</button>
      <div v-if="costs?.money"  class="block mt-6"> 
        <span>Деньги на выходе</span>
        <v-text-field type="text" readonly v-model="money_cost"></v-text-field>
      </div>
      <div v-if="costs?.date_exit"  class="block mt-6"> 
        <span>Дата выхода</span>
        <v-text-field type="text" readonly v-model="date_exit"></v-text-field>
      </div>
      <div v-if="costs?.percent"  class="block mt-6"> 
        <span>% заработка</span>
        <v-text-field type="text" readonly v-model="percent"></v-text-field>
      </div>
    </form>
      
    <div id="chart">
      <apexchart type="area" height="350" :options="chartOptions" :series="series"></apexchart>
    </div>
  </div>
</template>

<script>
import { defineComponent } from 'vue';
import axios from 'axios'
import moment from 'moment';
export default defineComponent({
  name: 'HomeView',
  data: function() {
    return {
      moment,
      costs: null,
      percent: null,
      money: null,
      date_exit: null,
      money_cost: null,
      companies: null,
      chartOptions: {
        chart: {
          id: "vuechart-example",
        },
        xaxis: {
          categories: [2023, 2024, 2025, 1994, 1995, 1996, 1997, 1998],
        },
      },
      series: [
        {
          name: "series-1",
          data: [0],
        },
      ],
    };
  },
  methods: {
    calculate(){
      axios.post('https://go-algo.onrender.com/select', {

      "money_input": this.money,
      "company_input": this.companies,
      "period_input": "string", 
       "strategy": "rare"
      })
      .then((response) => {
        this.costs = response.data
        this.money_cost = this.costs.money.toFixed(0)
        this.date_exit = this.moment(this.costs.date_exit[0]).format('DD-MM-YYYY')
        this.percent = this.costs.percent.toFixed(2) + '%'
        const predicted_close = this.costs?.date_cost?.map((item) => {
            return item.Predicted_Close;
          });
        console.log(predicted_close);
        this.series = [{
          data: predicted_close
        }]
      });
    }
  }
});
</script>
