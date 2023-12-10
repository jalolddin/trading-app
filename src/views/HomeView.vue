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
        v-model="companies"
      ></v-select>
      </div>
      <div class="block"> 
        <span>Стратегия</span>
        <v-select
        clearable
        :items="actions"
        v-model="action"
      ></v-select>
      </div>
      <div class="block"> 
        <span>Период времени</span>
        <v-text-field v-model="period" type="date" clearable></v-text-field>
      </div>
      <button class="submit" type="button" @click="calculate">Рассчитать</button>
      <div v-if="costs?.money"  class="block mt-6"> 
        <span>Деньги на выходе</span>
        <v-text-field type="text" readonly v-model="money_cost"></v-text-field>
      </div>
      <div v-if="costs?.percent"  class="block mt-6"> 
        <span>% заработка</span>
        <v-text-field type="text" readonly v-model="percent"></v-text-field>
      </div>
    </form>
    <div class="right">
      <div id="chart">
        <apexchart type="area" height="350" :options="chartOptions" :series="series"></apexchart>
      </div>
      <div v-if="buy">

        <h2>Покупать</h2>
        <hr class="mt-3">
        <v-table class="mt-3">
          <thead>
            <tr>
              <th class="text-left">
                Компания
              </th>
              <th class="text-left">
                Дата рекомендации покупки
              </th>
              <th class="text-left">
                Прогнозируемое сумма
              </th>
              <th class="text-left">
                Кол-во акции
              </th>
            </tr>
          </thead>
          <tbody>
            <tr
            v-for="item in buy"
            :key="item"
            >
            <td>{{ item.Stock }}</td>
            <td>{{ moment(item.Date).format('DD-MM-YYYY') }}</td>
            <td>{{ item.Predicted_Close }}</td>
            <td>{{ item.amount }}</td>
          </tr>
        </tbody>
      </v-table>
    </div>
    <div class="mt-4" v-if="sell">
    <h2>Продавать</h2>
    <hr class="mt-3">
    <v-table class="mt-3">
      <thead>
        <tr>
          <th class="text-left">
            Компания
          </th>
          <th class="text-left">
            Дата рекомендации покупки
          </th>
          <th class="text-left">
            Прогнозируемое сумма
          </th>
          <th class="text-left">
            Кол-во акции
          </th>
        </tr>
      </thead>
      <tbody>
        <tr
        v-for="item in sell"
        :key="item"
        >
        <td>{{ item.Stock }}</td>
        <td>{{ moment(item.Date).format('DD-MM-YYYY') }}</td>
        <td>{{ item.Predicted_Close }}</td>
        <td>{{ item.amount }}</td>
      </tr>
    </tbody>
  </v-table>
</div>
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
      actions: [
        {
          title: 'Частые действия',
          value: 'rare'
        },
        {
          title: 'Редкия',
          value: 'frequent'
        }
      ],
      action: null,
      moment,
      costs: null,
      buy: null,
      sell: null,
      percent: null,
      money: null,
      date_exit: null,
      money_cost: null,
      period: null,
      companies: null,
      chartOptions: {
        chart: {
          id: "vuechart-example",
        },
        xaxis: {
          categories: [2023],
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
      "company_input": [this.companies],
      "period_input": this.period, 
       "strategy": this.action
      })
      .then((response) => {
        this.costs = response.data
        this.money_cost = this.costs.money.toFixed(0)
        this.date_exit = this.moment(this.costs.date_exit[0]).format('DD-MM-YYYY')
        this.percent = this.costs.percent.toFixed(2) + '%'
        this.buy = this.costs.buy
        this.sell = this.costs.sell
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
