<template>
  <div class="modal-dialog">
    <div class="modal-content text-center">
      <div class="modal-body">
        <div class="row">
          <div class="col">
            <img class="modal-logo" :src="'/static/actions/burn.png'" alt=""/>
            <p class="modal-tit">{{ isWithdraw ? $t("modal.tit2") : $t("modal.tit1") }} {{ this.$route.params.tx_id }}</p>
            <input v-show="!isWithdraw" type="text" v-model="willStake" placeholder="0" class="staking-input">
            <input v-show="isWithdraw" type="text" v-model="willWithdraw" placeholder="0" class="staking-input unstake">
          </div>
        </div>
        <div class="row">
          <div class="col text-left balance-tit">
            {{ $t("common.balance")}}
            <b class="balance-txt" v-show="!isWithdraw">{{ cBalance }}</b>
            <b class="balance-txt" v-show="isWithdraw">{{ stakingAmount }}</b>
          </div>
          <div class="col text-right">
            <input type="submit" class="btn DeFi-btn max-btn" :value="$t('common.max')"
                   @click="maxBalance">
          </div>
        </div>
        <div class="row">
          <div class="col">
            <button type="button" class="btn DeFi-btn cancel-btn" data-dismiss="modal">{{ $t("common.cancel") }}</button>
          </div>
          <div class="col">
            <input type="submit" class="btn DeFi-btn confirm-btn" :value="$t('common.confirm')"
                   v-show="!isWithdraw && (cBalance > 0)"
                   @click="staking">

            <input type="submit" class="btn DeFi-btn confirm-btn" :value="$t('common.buyLp')"
                   v-show="!isWithdraw && (cBalance == 0) && buyLPEnabled"
                   @click="buyLp">

            <input type="submit" class="btn DeFi-btn approve-btn" :value="$t('common.approveLPBuying')"
                   v-show="!isWithdraw && (cBalance == 0) && !buyLPEnabled"
                   @click="approveLPBuying">

            <input type="submit" class="btn DeFi-btn confirm-btn" :value="$t('common.confirm')"
                   v-show="isWithdraw"
                   @click="withdraw">
          </div>
        </div>
      </div>
    </div>
    <loading :active.sync="loading"
             :can-cancel="false"
             :is-full-page="true"></loading>
  </div>
</template>

<script>
import Decimal from "decimal.js";
import $ from "jquery";
import Loading from 'vue-loading-overlay';
import 'vue-loading-overlay/dist/vue-loading.css';
import chainOpt from "@/utils/web3/chainOperation";
import {GetInputTokenFromPoolID, GetFlashSwapInfoFromPoolID, connectorAddr} from "@/utils/web3/constant";

let opt = chainOpt.opt;

export default {
  name: "StakeModal",
  props: ["isWithdraw", "cBalance", "stakingAmount", "pid", "poolInfo", "doHide"],
  components: {Loading},
  data() {
    return {
      willStake: 0,
      willWithdraw: 0,
      loading: false,
      buyLPEnabled: false,
      queryingLPBuyingApprove: false,
      inputToken: "",
      approving: true,
    }
  },

  watch: {
    cBalance() {
      this.willStake = this.cBalance
    },

    pid() {
      if(this.pid !== null) {
        this.inputToken = GetInputTokenFromPoolID(this.pid)
        if("ETH" === this.inputToken) {
          this.buyLPEnabled = true
        } else {
          this.loopCheckERC20LPBuyingApprove(this.inputToken)
          setInterval(_=>{
            this.loopCheckERC20LPBuyingApprove(this.inputToken)
          }, 5000)
        }
      }
    },

    willStake() {
      try {
        let willStake = new Decimal(this.willStake)
        if(willStake.gt(this.cBalance)) {
          this.willStake = this.cBalance
        }
      } catch (e) {}
    }
  },

  methods: {
    async loopCheckERC20LPBuyingApprove(inputToken) {
      if(this.queryingLPBuyingApprove) {
        return
      }
      this.queryingLPBuyingApprove = true;
      let contracts = opt.getContracts()
      if(contracts === null) {
        this.queryingLPBuyingApprove = false;
        return
      }
      let tokenInfo = GetFlashSwapInfoFromPoolID(this.pid)
      this.buyLPEnabled = await opt.commonApproveCheck(contracts.inputTokens[tokenInfo.inputTokenIdx], connectorAddr).catch(e=> {return false})
      if(this.buyLPEnabled) {

      }
      this.queryingLPBuyingApprove = false;
    },

    async staking() {
      this.loading = true
      let stakeAmount = this.willStake
      try {
        stakeAmount = new Decimal(stakeAmount)
      } catch (e) {
        //todo: show alert to tell user input not a number
        return
      }

      let enabled = await opt.isPoolEnable(this.pid)
      if (!enabled) {
        this.enabled = false
        this.loading = false
        return
      }

      let uniAmount = await opt.userUniBalance(this.pid)
      if (stakeAmount.gt(uniAmount)) {
        //todo: show some alert to tell user they don't have enough uni token to stake
        return
      }

      let tx = await opt.deposit(this.pid, stakeAmount)

      if (tx !== null) {
        console.log("transaction send success, tx hash is: ", tx)
      } else {

      }

      this.loading = false
      this.doHide()
    },

    async withdraw() {
      this.loading = true
      let willWithdraw = 0
      try {
        willWithdraw = new Decimal(this.willWithdraw)
      } catch (e) {
        //todo: show alert to tell user input not a number
        return
      }

      if (willWithdraw.gt(this.poolInfo.orgStake)) {
        //todo: show some alert to tell user they don't have enough staked uni token to withdraw
        return
      }
      await opt.withdraw(this.pid, willWithdraw)
      this.loading = false
      this.doHide()
    },

    async approveLPBuying() {

      console.log(["this,pid:", this.pid])
      this.loading = true
      let tokenInfo = GetFlashSwapInfoFromPoolID(this.pid)
      let contracts = opt.getContracts()
      if(contracts === null) {
        this.loading = false
        return
      }
      let approveRet = await opt.commonApprove(contracts.inputTokens[tokenInfo.inputTokenIdx], connectorAddr).catch(e=>{
        return null
      })

      if(approveRet !== null) {
        $(this.$el).find(".approve-btn").attr("disabled", "disabled")
      }
      this.loading = false
    },

    async buyLp() {
      let inputToken = GetInputTokenFromPoolID(this.pid)
      this.loading = true
      if("ETH" === inputToken) {
        let ret = await opt.flashStakingForETH(this.pid)
        console.log(ret)
      } else {
        let ret = await opt.flashStakingForERC20(this.pid)
        console.log(ret)
      }

      this.doHide()
      this.loading = false
    },

    maxBalance() {
      if (this.isWithdraw) {
        this.willWithdraw = this.stakingAmount;
      } else {
        this.willStake = this.cBalance;
      }
    },
  },
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
@import "../../styles/colors.scss";
@import "../../styles/fonts.scss";

.modal-dialog {
  margin-top: 10% !important;
}
.modal-body {
  background-color: $bg;
  padding: 30px 0;

  .modal-tit {
    font-size: 21px;
    color: $txtColor;
    letter-spacing: 2px;
    font-family: $fontTit;
    margin: 15px 0;
  }

  .modal-logo {
    width: 120px;
    height: 120px;
  }

  .balance-tit,
  .balance-txt {
    font-family: $fontTxt;
    color: $txtColor;
    font-size: 12px;
  }
  .balance-txt {
    color: $txtColor;
  }

  .staking-input {
    color: $notesColor;
    background-color: $divBg;
    font-family: $fontTxt;
    width: 100%;
    height: 54px;
    font-size: 24px;
    padding: 16px;
    border-width: 1px;
    border-style: solid;
    border-color: $inputBorder;
    border-image: initial;
    border-radius: 15px;
    outline: none;
    margin-top: 25px !important;
    margin-bottom: 15px !important;
  }

  .approve-btn,
  .confirm-btn,
  .cancel-btn {
    min-width: 0;
    width: 100%;
    height: 56px;
    margin: 20px 0 !important;
    font-size: 14px;
    font-weight: 500;
  }
  .max-btn {
    font-size: 12px;
    height: 18px;
    padding: 0;
    min-width: 0;
    width: 35%;
    float: right;
  }
}
</style>
