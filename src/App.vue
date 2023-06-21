<template>
  <header class="header">
    <h1 class="header__title">XAE6KA</h1>

    <nav class="header__links">
      <a href="/" @click.prevent="funcPage(1)" class="header__link">home</a>

      <a href="/two-page" @click.prevent="funcPage(2)" class="header__link">
        converter
      </a>
    </nav>
  </header>

  <main class="main">
    <div class="block">
      <h4 v-if="watchError1()" class="main__error">Значение некорректно</h4>

      <p class="main__error"></p>

      <h4 v-if="watchError2()" class="main__error">Значение некорректно</h4>
    </div>

    <nav class="main__nav">
      <div class="main__currency">
        <!-- <label for="mainCurrency" class="main__label">
            Введите основную валюту:
          </label> -->

        <input
          name="currency"
          list="currency"
          autocomplete="off"
          class="main__input"
          id="mainCurrency"
          v-model="mainCurrency"
          placeholder="Main currency"
          type="text"
        />
        <datalist id="currency">
          <option>USD</option>
          <option>RUB</option>
          <option>EUR</option>
          <option>BTN</option>
        </datalist>
      </div>

      <button class="main__button" @click="add()">Добавить</button>

      <div class="main__currency">
        <!-- <label for="currency2" class="main__label">
          Введите вторую валюту:
        </label> -->

        <input
          name="currencyC"
          list="currencyC"
          autocomplete="off"
          class="main__input"
          id="currency2"
          placeholder="Two currency"
          v-model="currency2"
          @keydown.enter="add()"
          type="text"
        />
        <datalist id="currencyC">
          <option>USD</option>
          <option>RUB</option>
          <option>EUR</option>
          <option>ETB</option>
          <option>CUP</option>
          <option>TZS</option>
          <option>BTN</option>
        </datalist>
      </div>
    </nav>

    <section class="main__container">
      <div v-for="block in blocks" :key="block" class="main__block">
        <p class="block__text">
          1 {{ block.c1 }} = {{ block.c2_currency }} {{ block.c2 }}
        </p>

        <button @click="funcRemoveEl(block)" class="block__remove">
          Удалить
        </button>
      </div>
    </section>
  </main>

  <section class="converter _isNotActive">
    <div class="converter__block">
      <select class="conventer__select" v-model="mainCurrency">
        <option value="USD">USD</option>
        <option value="RUB">RUB</option>
        <option value="EUR">EUR</option>
        <option value="BTN">BTN</option>
      </select>

      <input
        type="text"
        class="converter__sum"
        v-model="currency2_quantity"
        v-on:input="convert()"
      />
    </div>

    <img
      src="./assets/i/Frame1.png"
      alt=""
      class="converter__image"
      @click="convert()"
    />

    <div class="converter__block">
      <select class="conventer__select" v-model="currency2">
        <option value="USD">USD</option>
        <option value="RUB">RUB</option>
        <option value="EUR">EUR</option>
        <option value="ETB">ETB</option>
        <option value="CUP">CUP</option>
        <option value="TZS">TZS</option>
        <option value="BTN">BTN</option>
      </select>

      <p class="converter__sum">
        {{ sum }}
      </p>
    </div>
  </section>
</template>

<style>
@import "./assets/css/style.css";
</style>

<script>
export default {
  name: "App",

  data() {
    return {
      mainCurrency: "USD",
      currency2: "",
      currency2_quantity: 0,
      sum: 0,
      blocks: [{ c1: "USD", c2: "RUB", c2_currency: "79.57" }],
      dataDict: null,
      converter_currency1: "",
      converter_currency2: "",
      converter_quantity1: 0,
      converter_quantity2: 0,
    };
  },

  created() {
    this.fetchResponse(this.mainCurrency);

    let data = localStorage.getItem("blocks");

    if (data) {
      this.blocks = JSON.parse(data);
    }
  },

  methods: {
    async fetchResponse(mainC) {
      const URL = `https://v6.exchangerate-api.com/v6/1017d93aa91c75ccca14f2c1/latest/${mainC}`;

      const response = await fetch(URL);

      const data = await response.json();

      console.log(data);

      this.dataDict = data.conversion_rates;
    },

    add() {
      if (!this.mainCurrency || !(this.mainCurrency in this.dataDict)) {
        return;
      }

      this.fetchResponse(this.mainCurrency);

      const newBlock = {
        c1: this.mainCurrency,
        c2: this.currency2.toUpperCase(),
        c2_currency: null,
      };

      if (newBlock.c2 in this.dataDict) {
        newBlock.c2_currency =
          this.dataDict[this.currency2.toUpperCase()].toFixed(2);

        this.blocks.push(newBlock);

        this.currency2 = "";

        localStorage.setItem("blocks", JSON.stringify(this.blocks));
      }
    },

    convert() {
      this.fetchResponse(this.mainCurrency);

      if (
        this.mainCurrency in this.dataDict &&
        this.currency2 in this.dataDict
      ) {
        let sum = this.dataDict[this.currency2];

        this.sum = this.currency2_quantity * sum;
      }
    },

    funcPage(page) {
      if (page === 1) {
        document.querySelector(".main").classList.remove("_isNotActive");
        document.querySelector(".converter").classList.add("_isNotActive");
      } else if (page === 2) {
        console.log(document.querySelector(".converter"));
        document.querySelector(".main").classList.add("_isNotActive");
        document.querySelector(".converter").classList.remove("_isNotActive");
      }
    },

    funcRemoveEl(el) {
      this.blocks = this.blocks.filter((t) => t != el);

      localStorage.setItem("blocks", JSON.stringify(this.blocks));
    },

    watchError1() {
      try {
        if (this.mainCurrency) {
          if (!(this.mainCurrency.toUpperCase() in this.dataDict)) {
            return true;
          } else {
            return false;
          }
        }
      } catch (e) {
        console.error(e);
      }
    },

    watchError2() {
      try {
        if (this.currency2) {
          if (!(this.currency2.toUpperCase() in this.dataDict)) {
            return true;
          } else {
            return false;
          }
        }
      } catch (e) {
        console.error(e);
      }
    },
  },
};
</script>
