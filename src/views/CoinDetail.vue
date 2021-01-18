<template>
  <div class="flex-col">
    <div class="flex justify-center">
      <pulse-loader :loading="isLoading" :color="'#68d391'" :size="100" />
    </div>

    <template v-if="!isLoading">
      <div class="flex flex-col sm:flex-row justify-around items-center">
        <div class="flex flex-col items-center">
          <img
            :src="
              `https://static.coincap.io/assets/icons/${asset.symbol.toLowerCase()}@2x.png`
            "
            class="w-20 h-20 mr-5"
            :alt="asset.name"
          />
          <h1 class="text-5xl">
            {{ asset.name }}
            <small class="sm:mr-2 text-gray-500">{{ asset.symbol }}</small>
          </h1>
        </div>

        <div class="my-10 flex flex-col">
          <ul>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Ranking</b>
              <span>#{{ asset.rank }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio actual</b>
              <span>{{ asset.priceUsd | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio más bajo</b>
              <span>{{ min | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio más alto</b>
              <span>{{ max | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Precio Promedio</b>
              <span>{{ avg | dollar }}</span>
            </li>
            <li class="flex justify-between">
              <b class="text-gray-600 mr-10 uppercase">Variación 24hs</b>
              <span>{{ asset.changePercent24Hr | percentage }}</span>
            </li>
          </ul>
        </div>

        <div class="my-10 sm:mt-0 flex flex-col justify-center text-center">
          <button
            @click="toggleConverter"
            class="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded"
          >
            {{ fromUsd ? `USD to ${asset.symbol}` : `${asset.symbol} to USD` }}
          </button>

          <div class="flex flex-row my-5">
            <label class="w-full" for="convertValue">
              <input
                v-model="convertValue"
                id="convertValue"
                type="number"
                class="text-center bg-white focus:outline-none focus:shadow-outline border border-gray-300 rounded-lg py-2 px-4 block w-full appearance-none leading-normal"
                :placeholder="`Valor en ${fromUsd ? 'USD' : asset.symbol}`"
              />
            </label>
          </div>

          <span class="text-xl">
            {{ convertResult }} {{ fromUsd ? asset.symbol : "USD" }}</span
          >
        </div>
      </div>

      <line-chart
        class="my-10"
        :colors="['orange']"
        :min="min"
        :max="max"
        :data="chartData"
      />

      <h3 class="text-xl my-10">Mejores Ofertas de Cambio</h3>
      <table>
        <tr
          v-for="m in markets"
          :key="`${m.exchangeId}-${m.priceUsd}`"
          class="border-b"
        >
          <td>
            <b>{{ m.exchangeId }}</b>
          </td>
          <td>
            {{ m.priceUsd | dollar }}
          </td>
          <td>{{ m.baseSymbol }} / {{ m.quoteSymbol }}</td>
          <td>
            <px-button
              :isLoading="m.isLoading || false"
              v-if="!m.url"
              @custom-click="getWebsite(m)"
            >
              <slot> Obtener Link </slot>
            </px-button>

            <a v-else class="hover:underline text-green-600" target="_blank">{{
              m.url
            }}</a>
          </td>
        </tr>
      </table>
    </template>
  </div>
</template>

<script>
import { getAsset, getAssetHistory, getMarkets, getExchange } from "@/api"
import PxButton from "@/components/PxButton"

export default {
  name: "CoinDetail",

  data() {
    return {
      isLoading: false,
      asset: {},
      history: [],
      markets: [],
      fromUsd: true,
      convertValue: null
    }
  },

  components: {
    PxButton
  },

  created() {
    this.getCoin()
  },

  methods: {
    getCoin() {
      this.isLoading = true
      const id = this.$route.params.id
      Promise.all([getAsset(id), getAssetHistory(id), getMarkets(id)])
        .then(([asset, history, markets]) => {
          this.asset = asset
          this.history = history
          this.markets = markets
        })
        .finally(() => (this.isLoading = false))
    },

    getWebsite(exchange) {
      this.$set(exchange, "isLoading", true)
      return getExchange(exchange.exchangeId)
        .then(res => {
          this.$set(exchange, "url", res.exchangeUrl)
        })
        .finally(() => this.$set(exchange, "isLoading", true))
    },

    toggleConverter() {
      this.fromUsd = !this.fromUsd
    }
  },

  computed: {
    min() {
      return Math.min(
        ...this.history.map(h => parseFloat(h.priceUsd).toFixed(2))
      )
    },
    max() {
      return Math.max(
        ...this.history.map(h => parseFloat(h.priceUsd).toFixed(2))
      )
    },
    avg() {
      const sum = this.history.reduce((acum, h) => {
        acum += parseFloat(h.priceUsd)
        return acum
      }, 0)

      return (sum / this.history.length).toFixed(2)
    },

    chartData() {
      return this.history.map(h => [h.date, parseFloat(h.priceUsd).toFixed(2)])
    },

    convertResult() {
      if (!this.convertValue) {
        return 0
      }
      const result = this.fromUsd
        ? this.convertValue / this.asset.priceUsd
        : this.convertValue * this.asset.priceUsd

      return result.toFixed(4)
    }
  },

  watch: {
    $route() {
      this.getCoin()
    }
  }
}
</script>

<style scoped>
td {
  padding: 10px;
  text-align: center;
}
</style>
