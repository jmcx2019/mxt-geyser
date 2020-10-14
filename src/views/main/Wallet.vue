<template>
  <div class="wallet-container">
    <div class="row">
      <div class="col-xl-6 col-lg-6 col-md-6 col-sm-12 col-xs-12">
        <div class="balance-col-div">
          <div class="row">
            <div class="col-3 d-flex justify-content-around">
              <img class="geyser-logo" :src="'/static/logo/logo.png'" alt=""/>
            </div>
            <div class="col-9">
              <p class="preview-tit">{{ $t("home.balance") }}</p>
              <p v-if="userBalance === false" class="preview-txt">{{ $t("home.status") }}</p>
              <p v-if="userBalance !== false" class="preview-txt">{{ $HelperTools.toFinancialVal(userBalance) }}</p>
            </div>
          </div>
          <div class="row stats-div">
            <div class="col stats-txt">{{ $t("home.pending") }}</div>
            <div class="col text-right stats-txt">
              {{ $HelperTools.toFinancialVal(tokenBalance) }}
              {{ $t("token.name") }}
            </div>
          </div>
        </div>
      </div>

      <div class="col-xl-6 col-lg-6 col-md-6 col-sm-12 col-xs-12">
        <div class="supply-col-div">
          <div class="row">
            <div class="col">
              <p class="preview-tit">{{ $t("home.total") }}</p>
              <p v-if="!totalReward" class="preview-txt">{{ $t("home.status") }}</p>
              <p v-if="totalReward" class="preview-txt">{{ $t("home.unlimited") }}</p>
            </div>
          </div>
          <div class="row stats-div">
            <div class="col-7 stats-txt">{{ $t("home.rewards") }}</div>
            <div class="col-5 text-right stats-txt">{{ rewardsBalance }} {{ $t("token.name") }}</div>
          </div>
        </div>
      </div>
    </div>

    <div class="row" v-show="walletConnectView">
      <div class="col text-center wallet-connect-tit">
        {{ $t("wallet.connect") }}
      </div>
    </div>

    <div class="row">
      <div class="col d-flex justify-content-around text-center">
        <button v-show="walletConnectView" type="submit" class="DeFi-btn" @click="unlockWallet">
          {{ $t("wallet.name") }}
        </button>
        <button v-show="!walletConnectView" type="submit" class="DeFi-btn see-menu-btn" @click="seeTheMenu">
          {{ $t("wallet.seeTheMenu") }}
        </button>
      </div>
    </div>

    <div class="row" v-show="walletConnectError">
      <div class="col text-center wallet-error-txt">
        {{ $t("wallet.error") }}
      </div>
    </div>

    <loading :active.sync="loading"
             :can-cancel="false"
             :is-full-page="true"></loading>
  </div>
</template>

<script>
import chainOpt from "../../utils/web3/chainOperation";
import Loading from 'vue-loading-overlay';
import 'vue-loading-overlay/dist/vue-loading.css';
import Decimals from "decimal.js"
import {allPairs} from "@/utils/web3/constant";
import {getUserHarvest, getBlockReward} from "@/utils/web3/globalQuerying";

let opt = chainOpt.opt
const rewardPerBlock = "..."

export default {
  name: "Wallet",
  components: { Loading },

  data() {
    return {
      ConnectWalletModalText: 'Waiting for connection',
      loading: false,
      walletConnectView: true,
      walletConnectError: false,

      tokenBalance: "0",
      rewardsBalance: "0",

      totalReward: true,
      userBalance: false,
    };
  },

  mounted() {
    if(chainOpt.wallet.isConnected()) {
      this.setWalletConnected()
    }
  },

  methods: {
    async updatePoolInfo() {
      let totalCollected = new Decimals(0);
      for(let i=0; i<allPairs.length; i++) {
        try {
          let pool = await opt.poolInfo(allPairs[i].pid)
          totalCollected = totalCollected.add(pool.userToCollect)
          this.$nextTick(_=>{
            this.tokenBalance = totalCollected.toDP(6, Decimals.ROUND_FLOOR).toString()
          })
        } catch (e) {}
      }

    },

    setWalletConnected() {
      opt.userTokenBalance()
          .then(r=>{
            this.userBalance = r
          })

      setInterval(_=>{
        this.tokenBalance = getUserHarvest()
        this.rewardsBalance = getBlockReward()
      }, 1000)
      this.loading = false
      this.walletConnectView = false
    },

    async unlockWallet() {
      //wallet connect
      this.loading = true

      if(!chainOpt.wallet.isConnected()) {
        chainOpt.wallet.connect()
          .then(r=>{
            if(r === null) {
              //not connect, do nothing
              this.loading = false
              return
            }
            this.setWalletConnected()
          })
          .catch(()=>{
            //unknown error, do nothing
            this.loading = false
            this.walletConnectError = true
          })
      } else {
        //already connected
        this.setWalletConnected()
      }
    },

    seeTheMenu() {
      this.$router.push({name: "Dashboard"})
    }
  },
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
@import "../../styles/colors.scss";
@import "../../styles/fonts.scss";

.wallet-container {
  max-width: 800px;
  margin: 15px auto;
  height: 100%;

  .balance-col-div,
  .supply-col-div {
    border-width: 1px;
    border-style: solid;
    border-color: $divBorderColor;
    border-image: initial;
    border-radius: 10px;
    margin-bottom: 15px !important;
    padding: 15px 0;
    background-color: $divBg;
  }

  .geyser-logo {
    width: 60px;
    height: 60px;
    margin: auto;
  }

  .preview-tit {
    font-size: 14px;
    font-family: $fontTit;
    color: $titL2Color;
  }
  .preview-txt {
    font-size: 27px;
    font-family: $fontTit;
    color: $titColor;
  }

  .stats-div {
    border-top-width: 1px;
    border-style: solid none none none;
    border-color: $divBorderColor;
    padding-top: 15px;
  }
  .stats-txt {
    font-size: 14px;
    font-family: $fontTxt;
    color: $titL2Color;
  }

  .wallet-connect-tit {
    font-size: 16px;
    color: $txtColor;
    line-height: 28px;
    font-family: $fontTxt;
    margin-top: 15px;
    margin-bottom: 15px;
  }
  .wallet-error-txt {
    font-size: 16px;
    color: $errColor;
    line-height: 28px;
    font-family: $fontTxt;
    margin-top: 15px;
  }
  .see-menu-btn {
    margin-top: 45px;
  }
}
</style>
