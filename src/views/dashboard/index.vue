<template>
  <div class="dashboard-container">
    <welcome></welcome>

    <div class="row">
      <div class="col-xl-4 col-lg-4 col-md-4 col-sm-12 col-xs-12 text-center"
           v-for="(product, index) in products" :key="index">
        <product :product-data="product" :product-index="PairNameToPoolID(product.name)"></product>
      </div>
    </div>

    <loading :active.sync="loading"
             :can-cancel="false"
             :is-full-page="true"></loading>
  </div>
</template>

<script>
import {PairNameToPoolID} from "@/utils/web3/constant";
import Product from "./Product";
import Welcome from "./../../components/Welcome";
import Loading from 'vue-loading-overlay';
import 'vue-loading-overlay/dist/vue-loading.css';

let defaultChainInfo = {
  supply: 0,

  userStaked: 0,
  poolStaked: 0,

  rewardPerHour: 0,
  rewardPerDay: 0,
  rewardPerWeek: 0,
}

export default {
  name: "dashboard",
  components: {Product, Welcome, Loading},

  data() {
    return {
      allBalance: null,
      poolInfo: null,
      loading: false,

      products: [
        {
          img: "/static/icon/MXT-ETH.png",
          name: "MXT-ETH",
          tx: "MXT-ETH",
          apy: "300",
          chainInfo: defaultChainInfo,
          stat: true,
          isETHPair: true,
          dateTime: "2020-10-10 22:00:00",
          double: false,
          link: "",
        },
        {
          img: "/static/icon/MXT-OKB.png",
          name: "MXT-OKB",
          tx: "MXT-OKB",
          apy: "300",
          chainInfo: defaultChainInfo,
          stat: true,
          isETHPair: false,
          dateTime: "2020-10-10 22:00:00",
          double: false,
          link: "",
        },
        {
          img: "/static/icon/MXT-USDT.png",
          name: "MXT-USDT",
          tx: "MXT-USDT",
          apy: "300",
          chainInfo: defaultChainInfo,
          stat: true,
          isETHPair: false,
          dateTime: "2020-10-10 22:00:00",
          double: false,
          link: "",
        },
      ]
    }
  },

  methods: {
    PairNameToPoolID: PairNameToPoolID,
    seeTheMenu() {
      this.$router.push({name: "Dashboard"})
    },
  },
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
.dashboard-container {
  //width: 100%;
  //height: 100%;
}
</style>
