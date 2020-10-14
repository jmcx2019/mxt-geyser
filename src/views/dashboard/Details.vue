<template>
  <div class="detail-div">
    <welcome :logo="'/static/icon/' + this.$route.params.tx_id + '.png'"
             :tit="this.$route.params.tx_id"
             :txt="$t('common.welcomeTxt')"></welcome>

    <div class="row">
      <div class="col-xl-2 col-lg-2 col-md-1 col-sm-12 col-xs-12"></div>
      <div class="col-xl-4 col-lg-4 col-md-5 col-sm-12 col-xs-12 text-center">
        <div class="product-div">
          <div class="row">
            <div class="col">
              <div class="product-logo-div">
                <img class="product-logo" :src="'/static/actions/withdraw.png'" alt=""/>
              </div>
              <p class="pdt-name">{{ mxtRewards }}</p>
              <p class="pdt-txt">{{ $t("details.harvest") }}</p>
            </div>
          </div>
          <div class="row">
            <div class="col">
              <input type="submit" class="btn DeFi-btn claim-btn" :value="$t('action.harvest')"
                     data-toggle="modal" data-target="#harvestModal"
                     id="withdrawBtn"
                     @click="harvest">
            </div>
          </div>
        </div>
      </div>

      <div class="col-xl-4 col-lg-4 col-md-5 col-sm-12 col-xs-12 text-center">
        <div class="product-div">
          <div class="row">
            <div class="col">
              <div class="product-logo-div">
                <img class="product-logo" :src="'/static/actions/deposit.png'" alt=""/>
              </div>
              <p class="pdt-name">{{ stakingAmount }}</p>
              <p class="pdt-txt">{{ $t("details.staked") }}</p>
            </div>
          </div>
          <div class="row">
            <div v-show="enabled" class="col-6 btn-lr-l">
              <input type="submit" class="btn DeFi-btn deposit-btn" :value="$t('action.deposit')"
                                    data-toggle="modal" data-target="#stakeModal"
                                    @click="showStakingModal(false)">
            </div>

            <div v-show="enabled" class="col-6 btn-lr-r">
              <input type="submit" class="btn DeFi-btn withdraw-btn" :value="$t('action.withdraw')"
                     data-toggle="modal" data-target="#stakeModal"
                     @click="showStakingModal(true)">
            </div>

            <div v-show="!enabled" class="col">
              <input type="submit" class="btn DeFi-btn approve-btn" id="approveBtn"
                     :value="$t('action.approve')"
                     @click="enable">
            </div>
          </div>
        </div>
      </div>
      <div class="col-xl-2 col-lg-2 col-md-1 col-sm-12 col-xs-12"></div>
    </div>

    <div class="row">
      <div class="col d-flex justify-content-around text-center">
        <button type="submit" class="DeFi-btn see-menu-btn" @click="seeTheMenu">{{ $t("wallet.seeTheMenu") }}</button>
      </div>
    </div>

    <div class="modal fade" id="stakeModal">
      <stake-modal :isWithdraw="isWithdraw"
                   :cBalance="cBalance"
                   :stakingAmount="stakingAmount"
                   :pid="pid"
                   :poolInfo="this.poolInfo"
                   :do-hide="hideStakeModal"
      ></stake-modal>
    </div>

    <loading :active.sync="loading"
             :can-cancel="false"
             :is-full-page="true"></loading>
  </div>
</template>

<script>
import chainOpt from "../../utils/web3/chainOperation";
import $ from "jquery";
import Loading from 'vue-loading-overlay';
import 'vue-loading-overlay/dist/vue-loading.css';
import Decimal from "decimal.js"
import Welcome from "@/components/Welcome";
import StakeModal from "@/views/dashboard/StakeModal";

let opt = chainOpt.opt

export default {
  name: "Details",

  components: {Loading, StakeModal, Welcome},

  data() {
    return {
      enabled: false,
      loading: false,
      pid: null,
      poolInfo: null,
      isWithdraw: false,
      querying: false,
      cBalance: 0,
    }
  },

  watch: {
    enabled() {
      if (this.enabled) {
        $(this.$el).find(".staking-input").removeAttr("disabled")
      } else {
        $(this.$el).find(".staking-input").attr("disabled", "disabled")
      }
    },
  },

  mounted() {
    this.pid = this.$route.params.pool_idx
    if (!this.$route.params.pool_info) {
      opt.poolInfo(this.pid)
          .then(r => {
            this.poolInfo = r;
            this.cBalance = this.poolInfo.uniBalance
            this.loopQueryGlobalPoolInfo()
          })
    } else {
      this.loopQueryGlobalPoolInfo()
    }

    if (this.mxtRewards === "0") {
      $(this.$el).find("#withdrawBtn").attr("disabled", "disabled")
    } else {
      $(this.$el).find("#withdrawBtn").removeAttr("disabled")
    }

    setTimeout(async _ => {
      this.enabled = await opt.isPoolEnable(this.pid).catch(e => {
        return false
      })

      if (!this.enabled) {
        this.$nextTick(_ => {
          $(this.$el).find(".staking-input").attr("disabled", "disabled")
        })
      }
    }, 500)
  },

  computed: {
    stakingAmount() {
      return this.poolInfo ? this.poolInfo.user.stakeIn : 0
    },

    mxtRewards() {
      let ret = this.poolInfo ? this.poolInfo.userToCollect : "0"
      if (ret === "0") {
        $(this.$el).find("#withdrawBtn").attr("disabled", "disabled")
      } else {
        $(this.$el).find("#withdrawBtn").removeAttr("disabled")
      }

      return ret
    }
  },

  methods: {
    seeTheMenu() {
      this.$router.push({name: "Dashboard"})
    },

    async loopQueryGlobalPoolInfo() {
      await this.queryGlobalPoolInfo()
      setTimeout(_ => {
        this.loopQueryGlobalPoolInfo()
      }, 5000)
    },

    async queryGlobalPoolInfo() {
      if (this.querying) {
        return
      }
      this.querying = true

      if (!window.allPoolInfo[this.pid]) {
        this.poolInfo = await opt.poolInfo(this.pid)
        this.cBalance = this.poolInfo.uniBalance
        this.querying = false
        return
      }

      this.poolInfo = window.allPoolInfo[this.pid]
      this.cBalance = this.poolInfo.uniBalance
      this.querying = false
    },

    async loopCheckEnabled() {
      this.enabled = await opt.isPoolEnable(this.pid).catch(e => {
        return false
      })

      if (this.enabled) {
        return
      }

      setTimeout(this.loopCheckEnabled, 5000)
    },

    async enable() {
      this.loading = true

      let enabled = await opt.isPoolEnable(this.pid)
      if (enabled) {
        this.enabled = true
        this.loading = false
        return
      }

      enabled = await opt.enableDeposit(this.pid)

      this.loading = false
      if (enabled === null) {
        return
      }
      $(this.$el).find("#approveBtn").attr("disabled", "disabled")
      await this.loopCheckEnabled()
    },

    async harvest() {
      this.loading = true
      let zeroWithdraw = new Decimal(0)
      await opt.withdraw(this.pid, zeroWithdraw)
      this.loading = false
      $('#harvestModal').modal('hide')
    },

    hideStakeModal() {
      $('#stakeModal').modal('hide')
    },

    showStakingModal($isWithdraw = false) {
      this.isWithdraw = $isWithdraw
    },
  },
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
@import "../../styles/colors.scss";
@import "../../styles/fonts.scss";

.detail-div {
  .product-div {
    border-width: 1px;
    border-style: solid;
    border-color: $divBorderColor;
    border-image: initial;
    border-radius: 10px;
    padding: 25px 0 15px;
    margin-top: 15px !important;
    background-color: $divBg;

    .product-logo {
      width: 100px;
      height: 100px;
    }

    .product-logo-div {
      background-color: $bg2;
      font-size: 36px;
      height: 100px;
      width: 100px;
      border-radius: 50px;
      -webkit-box-align: center;
      align-items: center;
      display: flex;
      -webkit-box-pack: center;
      justify-content: center;
      box-shadow: $btnBoxShadow1 4px 4px 8px inset, $btnBoxShadow2 -6px -6px 12px inset;
      margin: 0 auto 16px;
    }

    .pdt-name {
      font-family: $fontTit;
      font-size: 20px;
      font-weight: 700;
      color: $titColor;
    }

    .pdt-txt {
      font-family: $txtColor;
      font-size: 16px;
      font-weight: 400;
      color: $titL2Color;
    }

    .claim-btn,
    .approve-btn,
    .deposit-btn,
    .withdraw-btn {
      min-width: 0;
      width: 100%;
      height: 56px;
      margin: 20px 0 !important;
      font-size: 14px;
      font-weight: 500;
    }
  }

  .btn-lr-l {
    padding-right: 7px;
  }
  .btn-lr-r {
    padding-left: 7px;
  }

  .see-menu-btn {
    margin: 45px 0 !important;
  }
}
</style>
