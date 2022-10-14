<template>
  <h2 class="head">Currency Converter</h2>
  <div class="calculator-wrapper">
    <div class="calculator-row">
      <select
          v-model="form.currency_from"
          class="calculator-field"
          @change="changeHandler('from')"
      >
        <option
            v-for="(item, i) in currencies"
            :key="i"
            :value="item"
        >
          {{ item }}
        </option>
      </select>
      <input
          v-model="form.value_from"
          class="calculator-field"
          @input="calcToValue"
      />
    </div>
    <div class="calculator-row">
      <select
          v-model="form.currency_to"
          class="calculator-field"
          @change="changeHandler('to')"
      >
        <option
            v-for="(item, i) in currencies"
            :key="i"
            :value="item"
        >
          {{ item }}
        </option>
      </select>
      <input
          v-model="form.value_to"
          class="calculator-field"
          @input="calcFromValue"
      />
    </div>
    <p v-if="conversionRate">
      Rate: {{conversionRate.toFixed(4)}}
    </p>
    <p v-if="rate">
      Rate without 1%: {{rate.toFixed(4)}}
    </p>
  </div>
</template>

<script>

import axios from "axios";
import {computed, onMounted, reactive, ref} from "vue";

export default {
  name: 'App',
  setup() {
    const currencies = ref(["BTC", "ETH", "USD", "EUR"])
    const form = reactive({
      currency_from: "BTC",
      currency_to: "ETH",
      value_from: 1,
      value_to: null,
    })

    let conversionRate = ref(null)
    let rate = ref(null)

    const currencyFromIndex = computed(() => {
      return currencies.value.indexOf(form.currency_from)
    })

    const currencyToIndex = computed(() => {
      return currencies.value.indexOf(form.currency_to)
    })

    const calcToValue = () => {
      form.value_to = (conversionRate.value * form.value_from).toFixed(4)
    }

    const calcFromValue = () => {
      const rate = 1 / conversionRate.value
      form.value_from = (rate * form.value_to).toFixed(4)
    }

    const checkCurrencies = (value) => {
      if (currencyFromIndex.value === currencyToIndex.value) {
        if (value === 'from') {
          if (currencyToIndex.value === currencies.value.length - 1) {
            form.currency_to = currencies.value[0]
          } else {
            form.currency_to = currencies.value[currencyToIndex.value + 1]
          }
        }
        if (value === 'to') {
          if (currencyFromIndex.value === currencies.value.length - 1) {
            form.currency_from = currencies.value[0]
          } else {
            form.currency_from = currencies.value[currencyFromIndex.value + 1]
          }
        }
      }
    }

    const changeHandler = (value) => {
      checkCurrencies(value)
      getConversionRate()
    }

    const getConversionRate = () => {
      axios.post('https://dev-api.finteria.com/api/calculator/exchange/calculate', {
        currency_from: form.currency_from,
        currency_to: form.currency_to,
      })
          .then((response) => {
            rate.value = +response.data.data.conversion_rate

            const onePercent = response.data.data.conversion_rate / 100
            conversionRate.value = response.data.data.conversion_rate - onePercent
            calcToValue()
          })
          .catch((error) => {
            console.log('error', error);
          });
    }

    onMounted(() => {
      getConversionRate()
    })

    return {
      currencies,
      form,
      conversionRate,
      rate,
      calcFromValue,
      calcToValue,
      changeHandler
    }
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}

.head {
  text-align: center;
}
.calculator-wrapper {
  border: 1px solid #d0c9c9;
  border-radius: 10px;
  box-shadow: rgba(100, 100, 111, 0.2) 0px 7px 29px 0px;
  padding: 30px;
  max-width: 350px;
  margin: auto;
}
.calculator-row {
  display: flex;
  justify-content: space-between;
  margin-bottom: 30px;
}
.calculator-field {
  border: 1px solid #d9d9d9;
  border-radius: 4px;
  padding: 10px;
}
</style>
