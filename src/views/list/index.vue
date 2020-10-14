<template>
  <div class="list-container">
    <div class="row">
      <div class="col-xl-4 col-lg-4 col-md-4 col-sm-12 col-xs-12 text-center"
           v-for="(p, index) in products" :key="index">
        <div class="lists-div">
          <div class="row">
            <div class="col">
              <div class="product-logo-div">
                <img class="product-logo" :src="p.img" alt=""/>
              </div>
            </div>
          </div>
          <div class="row">
            <div class="col">
              <p class="pdt-name">{{ p.name }}</p>
            </div>
          </div>
          <div class="row">
            <div class="col text-left">
              <p class="pdt-tit">{{ $t("list.info") }}</p>
              <p class="pdt-txt">{{ $t("token.name") }} {{ $t("list.unlimited") }}</p>
            </div>
          </div>
          <div class="row">
            <div class="col text-left">
              <p class="pdt-tit">{{ $t("list.deposit") }}</p>
              <p class="pdt-txt">{{ $t("list.total") }}{{ $HelperTools.toFinancialVal(p.chainInfo.poolStaked) }}</p>
              <p class="pdt-txt">{{ $t("list.your") }}{{ $HelperTools.toFinancialVal(p.chainInfo.userStaked) }}</p>
            </div>
          </div>
          <div class="row">
            <div class="col text-left">
              <p class="pdt-tit">{{ $t("list.earned") }}</p>
              <p class="pdt-txt">{{ $t("list.available") }}{{ $HelperTools.toFinancialVal(p.chainInfo.userReward) }}</p>
              <p class="pdt-txt">{{ $t("list.hReward") }}{{ $HelperTools.toFinancialVal(p.chainInfo.rewardPerHour) }}</p>
              <p class="pdt-txt">{{ $t("list.dReward") }}{{ $HelperTools.toFinancialVal(p.chainInfo.rewardPerDay) }}</p>
              <p class="pdt-txt">{{ $t("list.wReward") }}{{ $HelperTools.toFinancialVal(p.chainInfo.rewardPerWeek) }}</p>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="row">
      <div class="col d-flex justify-content-around text-center">
        <button type="submit" class="DeFi-btn list-menu-btn" @click="seeTheMenu">{{ $t("wallet.seeTheMenu") }}</button>
      </div>
    </div>

    <loading :active.sync="loading"
             :can-cancel="false"
             :is-full-page="true"></loading>
  </div>
</template>

<script>
import Loading from 'vue-loading-overlay';
import 'vue-loading-overlay/dist/vue-loading.css';
import chainOpt from "@/utils/web3/chainOperation";
import Decimal from "decimal.js";

let rewardPerBlock = 1000;
let opt = chainOpt.opt

let defaultChainInfo = {
  supply: 0,

  userStaked: 0,
  poolStaked: 0,

  rewardPerHour: 0,
  rewardPerDay: 0,
  rewardPerWeek: 0,
}

export default {
  name: "List",
  components: { Loading },

  data() {
    return {
      queryingFlag: [],

      allBalance: null,
      poolInfo: [],
      loading: false,

      allTotal: 0,
      depositTotal: 0,
      yourTotal: 0,
      rewards: 0,
      hourRewards: 0,
      dayRewards: 0,
      weekRewards: 0,
      products: [
        {
          img: "/static/icon/MXT-ETH.png",
          name: "MixTrust",
          txt: "Deposit MXT-ETH UNI-V2 LP Earn MXT",
          tx: "MXT-ETH",
          lp: false,
          apy: "300",
          chainInfo: defaultChainInfo,
          totalSupply: "80000000" ,
          stat: true,
          dateTime: "2020-09-26 22:00:00",
          double: false,
          LPToken: "",
          link: "",
          pid: 3,
        },
        {
          img: "/static/icon/MXT-OKB.png",
          name: "OKEX",
          txt: "Deposit MXT-OKB UNI-V2 LP Earn MXT",
          tx: "MXT-OKB",
          lp: false,
          apy: "300",
          chainInfo: defaultChainInfo,
          totalSupply: "80000000" ,
          stat: true,
          dateTime: "2020-09-26 22:00:00",
          double: false,
          LPToken: "",
          link: "",
          pid: 5,
        },
        {
          img: "/static/icon/MXT-USDT.png",
          name: "Tether",
          txt: "Deposit MXT-USDT UNI-V2 LP Earn MXT",
          tx: "MXT-USDT",
          lp: false,
          apy: "300",
          chainInfo: defaultChainInfo,
          totalSupply: "80000000" ,
          stat: true,
          dateTime: "2020-09-26 22:00:00",
          double: false,
          LPToken: "",
          pid: 4,
        },
      ]
    }
  },

  mounted() {
    this.queryInfo()
  },

  methods: {
    updatePoolInfo(pid, pool, orgPid) {
      if(!pool) {
        return
      }


      let poolHourlyCollect = pool.poolToCollect.toString()
      let chainInfo = {
        userStaked: new Decimal(pool.user.stakeIn).toDP(8, Decimal.ROUND_FLOOR).toString(),
        userReward: new Decimal(pool.userToCollect).toDP(8, Decimal.ROUND_FLOOR).toString(),
        poolStaked: new Decimal(pool.pool.staked).toDP(8, Decimal.ROUND_FLOOR).toString(),

        rewardPerHour: new Decimal(poolHourlyCollect).toDP(8, Decimal.ROUND_FLOOR).toString(),
        rewardPerDay: new Decimal(poolHourlyCollect).mul(24 ).toDP(8, Decimal.ROUND_FLOOR).toString(),
        rewardPerWeek: new Decimal(poolHourlyCollect).mul(24 * 7).toDP(8, Decimal.ROUND_FLOOR).toString(),
      }

      let org = this.products[orgPid]
      org.chainInfo = chainInfo
      this.$set(this.products, orgPid, org);
    },

    seeTheMenu() {
      this.$router.push({name: "Dashboard"})
    },

    async loopQueryGlobalPoolInfo(pid, orgPid) {
      await this.queryGlobalPoolInfo(pid, orgPid)
      setTimeout(_ => {
        this.loopQueryGlobalPoolInfo(pid, orgPid)
      }, 3000)
    },

    async queryGlobalPoolInfo(pid, orgPid) {
      if (this.queryingFlag[pid]) {
        return
      }
      this.queryingFlag[pid] = true

      if (!window.allPoolInfo[pid]) {
        let pool = await opt.poolInfo(pid)
        this.queryingFlag[pid] = false
        this.updatePoolInfo(pid, pool, orgPid)
        return
      }

      let pool = window.allPoolInfo[pid]
      this.updatePoolInfo(pid, pool, orgPid)
      this.queryingFlag[pid] = false
    },

    queryInfo() {
      this.products.forEach((p, i)=>{
        if(p.stat) {
          this.loopQueryGlobalPoolInfo(p.pid, i)
        }
      })
    }
  },
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
@import "../../styles/colors.scss";
@import "../../styles/fonts.scss";

.list-container {
  .lists-div {
    border-width: 1px;
    border-style: solid;
    border-color: $divBorderColor;
    border-image: initial;
    border-radius: 15px;
    padding: 25px 15px 15px;
    margin-top: 15px !important;
    background-color: $divBg;

    .product-logo {
      width: 120px;
      height: 120px;
    }

    .product-logo-div {
      //background-color: $bg2;
      //font-size: 36px;
      //height: 120px;
      //width: 120px;
      //border-radius: 60px;
      //-webkit-box-align: center;
      //align-items: center;
      //display: flex;
      //-webkit-box-pack: center;
      //justify-content: center;
      //box-shadow: $btnBoxShadow1 4px 4px 8px inset, $btnBoxShadow2 -6px -6px 12px inset;
      margin: 0 auto 16px;
    }

    .pdt-name {
      font-family: $fontTit;
      font-size: 24px;
      font-weight: 700;
      color: $titColor;
    }

    .pdt-tit {
      font-family: $fontTit;
      font-size: 16px;
      font-weight: 700;
      color: $titL2Color;
    }

    .pdt-txt {
      font-family: $fontTxt;
      font-size: 14px;
      font-weight: 400;
      color: $notesColor;
    }
  }

  .list-menu-btn {
    margin-top: 30px;
    letter-spacing: 2px;
    font-family: $fontTit;
    text-transform: capitalize;
    font-size: 18px;
  }
}

</style>
