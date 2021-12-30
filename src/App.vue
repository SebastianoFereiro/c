<template>
  <div class="container mx-auto flex flex-col items-center bg-gray-100 p-4">
    <div
      v-show="spin"
      class="fixed w-100 h-100 opacity-80 bg-purple-800 inset-0 z-50 flex items-center justify-center"
    >
      <svg
        class="animate-spin -ml-1 mr-3 h-12 w-12 text-white"
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
      >
        <circle
          class="opacity-25"
          cx="12"
          cy="12"
          r="10"
          stroke="currentColor"
          stroke-width="4"
        ></circle>
        <path
          class="opacity-75"
          fill="currentColor"
          d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
        ></path>
      </svg>
    </div>
    <div class="container">
      <section>
        <div class="flex">
          <div class="max-w-xs">
            <label for="wallet" class="block text-sm font-medium text-gray-700"
              >Тикер</label
            >
            <div class="mt-1 relative rounded-md shadow-md">
              <input
                v-model="ticker"
                @keyup.enter="addTicker"
                type="text"
                name="wallet"
                id="wallet"
                class="block w-full pr-10 border-gray-300 text-gray-900 focus:outline-none focus:ring-gray-500 focus:border-gray-500 sm:text-sm rounded-md"
                placeholder="Введите название..."
              />
            </div>
            <div
              class="flex bg-white shadow-md p-1 rounded-md shadow-md flex-wrap"
            >
              <span
                v-for="(s, index) in filterSuggest"
                :key="index"
                @click="ticker = s.name"
                class="inline-flex items-center px-2 m-1 rounded-md text-xs font-medium bg-gray-300 text-gray-800 cursor-pointer"
              >
                {{ s.name }}
              </span>
            </div>
            <div class="text-sm text-red-600">{{ hasTicker }} {{ Error }}</div>
          </div>
        </div>
        <button
          @click="addTicker"
          type="button"
          class="my-4 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
        >
          <!-- Heroicon name: solid/mail -->
          <svg
            class="-ml-0.5 mr-2 h-6 w-6"
            xmlns="http://www.w3.org/2000/svg"
            width="30"
            height="30"
            viewBox="0 0 24 24"
            fill="#ffffff"
          >
            <path
              d="M13 7h-2v4H7v2h4v4h2v-4h4v-2h-4V7zm-1-5C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 18c-4.41 0-8-3.59-8-8s3.59-8 8-8 8 3.59 8 8-3.59 8-8 8z"
            ></path>
          </svg>
          Добавить
        </button>
      </section>
      <section v-if="page">
        <hr class="w-full border-t border-gray-600 my-4" />
        <div class="mt-1 relative rounded-md shadow-md">
          <input
            v-model="filter"
            type="text"
            name="wallet"
            id="wallet"
            class="block w-full pr-10 border-gray-300 text-gray-900 focus:outline-none focus:ring-gray-500 focus:border-gray-500 sm:text-sm rounded-md"
            placeholder="Поиск..."
          />
        </div>

        <button
          v-if="page > 1"
          @click="page--"
          type="button"
          class="mr-4 my-4 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
        >
          <!-- Heroicon name: solid/mail -->

          Назад
        </button>
        <button
          v-if="tickers.length > page * 6"
          @click="page++"
          type="button"
          class="my-4 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
        >
          <!-- Heroicon name: solid/mail -->
          Вперед
        </button>
      </section>
      <section v-if="this.tickers.length">
        <hr class="w-full border-t border-gray-600 my-4" />
        <dl class="mt-5 grid grid-cols-1 gap-5 sm:grid-cols-3">
          <div
            v-for="(item, index) in pageTicker"
            :class="[item === Selection ? 'border-4' : '']"
            @click="subscribeToTicker(item)"
            :key="index"
            class="bg-white overflow-hidden shadow rounded-lg border-purple-800 border-solid cursor-pointer"
          >
            <div class="px-4 py-5 sm:p-6 text-center">
              <dt class="text-sm font-medium text-gray-500 truncate">
                {{ item.FullName }}
              </dt>
              <dd class="mt-1 text-3xl font-semibold text-gray-900">
                <img
                  class="h-20"
                  :src="`https://www.cryptocompare.com${item.ImageUrl}`"
                />
                {{ item.price }}
              </dd>
            </div>
            <div class="w-full border-t border-gray-200"></div>
            <button
              @click.stop="removeSelectTicker(item)"
              class="flex items-center justify-center font-medium w-full bg-gray-100 px-4 py-4 sm:px-6 text-md text-gray-500 hover:text-gray-600 hover:bg-gray-200 hover:opacity-20 transition-all focus:outline-none"
            >
              <svg
                class="h-5 w-5"
                xmlns="http://www.w3.org/2000/svg"
                viewBox="0 0 20 20"
                fill="#718096"
                aria-hidden="true"
              >
                <path
                  fill-rule="evenodd"
                  d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z"
                  clip-rule="evenodd"
                ></path></svg
              >Удалить
            </button>
          </div>
        </dl>
        <hr class="w-full border-t border-gray-600 my-4" />
        <section v-if="tickers.length && graph.length" class="relative">
          <h3 class="text-lg leading-6 font-medium text-gray-900 my-8">
            {{ `${Selection} - USD` }}
          </h3>
          <div class="flex items-end border-gray-600 border-b border-l h-64">
            <div
              v-for="(g, index) in normalizedGraph"
              :key="index"
              :style="{ height: `${g}%` }"
              class="bg-purple-800 border w-10"
            ></div>
            <!-- <div class="bg-purple-800 border w-10 h-32"></div>
            <div class="bg-purple-800 border w-10 h-48"></div>
            <div class="bg-purple-800 border w-10 h-16"></div> -->
          </div>
          <button
            @click.stop="this.graph = []"
            type="button"
            class="absolute top-0 right-0"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              xmlns:xlink="http://www.w3.org/1999/xlink"
              xmlns:svgjs="http://svgjs.com/svgjs"
              version="1.1"
              width="30"
              height="30"
              x="0"
              y="0"
              viewBox="0 0 511.76 511.76"
              style="enable-background: new 0 0 512 512"
              xml:space="preserve"
            >
              <g>
                <path
                  d="M436.896,74.869c-99.84-99.819-262.208-99.819-362.048,0c-99.797,99.819-99.797,262.229,0,362.048    c49.92,49.899,115.477,74.837,181.035,74.837s131.093-24.939,181.013-74.837C536.715,337.099,536.715,174.688,436.896,74.869z     M361.461,331.317c8.341,8.341,8.341,21.824,0,30.165c-4.16,4.16-9.621,6.251-15.083,6.251c-5.461,0-10.923-2.091-15.083-6.251    l-75.413-75.435l-75.392,75.413c-4.181,4.16-9.643,6.251-15.083,6.251c-5.461,0-10.923-2.091-15.083-6.251    c-8.341-8.341-8.341-21.845,0-30.165l75.392-75.413l-75.413-75.413c-8.341-8.341-8.341-21.845,0-30.165    c8.32-8.341,21.824-8.341,30.165,0l75.413,75.413l75.413-75.413c8.341-8.341,21.824-8.341,30.165,0    c8.341,8.32,8.341,21.824,0,30.165l-75.413,75.413L361.461,331.317z"
                  fill="#718096"
                  data-original="#000000"
                ></path>
              </g>
            </svg>
          </button>
        </section>
      </section>
    </div>
  </div>
  <!-- <img alt="Vue logo" src="./assets/logo.png">
  <HelloWorld msg="Welcome to Your Vue.js App"/> -->
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'

// 1. если значение вычисляется на основе данных - метод created (наличие в состоянии зависимых данных)
// 2. при вопросе "Если, что-то меняется на, что-то выполни что-то" в блок Watch

export default {
  name: "App",
  data() {
    return {
      coin: [],
      page: 1,
      filter: "",
      show: false,
      Selection: "-",
      spin: true,
      ticker: "",
      tickers: [],
      graph: [],
      Error: "",
      hasTicker: "",
    }
  },
  created() {
    // if (this.localStorage(crypto)) {
    //   this.coin = this.localStorage(crypto)
    // }
    // localStorage.clear()
    let storage = localStorage.getItem("crypto")
    if (storage) {
      this.coin = JSON.parse(localStorage.getItem("crypto"))
    } else {
      this.wget()
    }

    if (localStorage.getItem("cr")) {
      this.tickers = JSON.parse(localStorage.getItem("cr"))
    }
    let url = Object.fromEntries(
      new URL(window.location).searchParams.entries()
    )
    if (url.page) {
      this.page = url.page
    }
    if (url.search) {
      this.filter = url.search
      // console.log(this.page, this.search)
    }
    this.spin = false
    //console.log(localStorage.getItem("cr"), this.tickers)
    // this.addSuggest()
  },
  computed: {
    normalizedGraph() {
      let min = Math.min(...this.graph)
      let max = Math.max(...this.graph)
      console.log(`min ${min}, max ${max}, ${this.graph}`)
      if (min === max) {
        return this.graph.map(() => 50)
      }
      //  (price) => 3 + ((price - minHeight) * 97) / (maxHeight - minHeight)
      // return this.graph.map((x) => (3 + (x - min) * 97) / 1 + (max - min))
      return this.graph.map((x) => 5 + ((x - min) * 95) / (max - min))
    },
    filterSuggest() {
      return this.coin
        .filter(
          (item) =>
            item.name.toUpperCase().indexOf(this.ticker.toUpperCase()) > -1
        )
        .slice(0, 4)
    },
    pageTicker() {
      let minElement = this.page * 6 - 6
      let maxElement = this.page * 6
      // console.log(this.tickers, this.API)
      let filterTicker = this.tickers.filter(
        (f) => f.name.toUpperCase().indexOf(this.filter.toUpperCase()) > -1
      )
      return filterTicker.slice(minElement, maxElement)
    },
    urlAddOptions() {
      return {
        page: this.page,
        filter: this.filter,
      }
    },
  },
  watch: {
    pageTicker() {
      if (this.pageTicker.length === 0 && this.page > 1) {
        this.page -= 1
      }
      localStorage.setItem("cr", JSON.stringify(this.tickers))
    },
    subscribeToTicker() {
      this.graph = []
    },
    urlAddOptions(v) {
      window.history.pushState(
        null,
        null,
        `http://localhost:8080?page=${v.page}&search=${v.filter}`
      )
    },
    ticker() {
      if (this.tickers.length) {
        this.tickers.some((c) => c.name === this.ticker.toUpperCase())
          ? (this.hasTicker = "Такой тикер уже существует")
          : (this.hasTicker = "")
        this.Error = ""
      }
    },
  },
  methods: {
    removeSelectTicker(item) {
      return this.tickers.splice(item, 1)
    },
    // localStorage(storage) {
    //   return JSON.parse(localStorage.getItem(storage))
    // },
    wget() {
      let c
      const URL =
        "https://min-api.cryptocompare.com/data/all/coinlist?summary=true"
      // let responce = await fetch(URL)
      // let data = (await responce.json(data)).Data
      // console.log(data)
      // console.log(data[42].FullName)
      fetch(URL)
        .then((responce) => responce.json())
        .then((data) => {
          return (c = data.Data)
        })
        .then(() => {
          console.log(c)
          for (let key in c) {
            this.coin.push(Object.assign(c[key], { name: key }))
          }
          localStorage.setItem("crypto", JSON.stringify(this.coin))
          return this.coin
        })
    },
    subscribeToTicker(tickerObject) {
      // тут надо поправить удаление только одной сущности
      this.Selection = tickerObject.name
      setInterval(async () => {
        await fetch(
          `https://min-api.cryptocompare.com/data/price?fsym=${tickerObject.name}&tsyms=USD`
        )
          .then((responce) => responce.json())
          // .then((data) => {
          //   return this.tickers.map((x) => {
          //     console.log(x, x.name, item.name, data)
          //     return x.name === item.name
          //       ? (item.price = "-")
          //       : (item.price = data.USD)
          //   })
          .then(
            (data) =>
              (this.tickers.find((f) => f.name === tickerObject.name).price =
                data.USD > 1 ? data.USD.toFixed(2) : data.USD.toPrecision(2))
          )
        if (tickerObject.name === this.Selection) {
          this.graph.push(tickerObject.price)
        }
        console.log(this.graph, tickerObject)
      }, 5000)
    },

    addTicker() {
      if (this.coin.some((t) => t.name === this.ticker.toUpperCase())) {
        this.tickers.push(
          this.coin.find((t) => t.name === this.ticker.toUpperCase())
        )
        // console.log(localStorage.getItem("cr"))
        this.ticker = ""
        this.show = true
        // console.log(this.tickers.some((c) => c.name === this.ticker))
      } else {
        this.Error = "Taкого тикера не найдено"
      }
    },
  },
}
</script>

<style></style>
