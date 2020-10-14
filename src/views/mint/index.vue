<template>
  <div class="row mint-div">
    <div class="col d-flex justify-content-around">
      <button v-show="walletConnectView" type="submit" class="DeFi-btn" @click="openMetamask">
        <img class="wallet-btn-icon text-right" :src="'/static/logo/logo.png'" :alt="$t('wallet.metamask')"/>
        <h2 class="wallet-btn-name text-left">{{ $t("wallet.name") }}</h2>
      </button>
      <button v-show="!walletConnectView && isAdmin" type="submit" class="DeFi-btn" @click="mint">
        <img class="wallet-btn-icon text-right" :src="'/static/logo/logo.png'" alt=""/>
        <h2 class="wallet-btn-name text-left">{{ $t("wallet.mint") }}</h2>
      </button>
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

let adminAddr = "place-holder"
let opt = chainOpt.opt

export default {
  name: "mint",
  components: { Loading },

  data() {
    return {
      walletConnectView: true,
      connectedWallet: "",
      isAdmin: false,
      loading: false,
    }
  },

  mounted() {
    this.adminCheck()
    if(chainOpt.wallet.isConnected()) {
      this.walletConnectView = false
      this.adminCheck()
    }
  },

  methods: {
    adminCheck() {
      let contractAdmin = chainOpt.wallet.getAccount()
      if(contractAdmin === null) {
        this.isAdmin = false
        return
      }

      contractAdmin = contractAdmin.toLowerCase()
      if((adminAddr === contractAdmin)) {
        this.isAdmin = true
      } else {
        this.$nextTick(()=>{
              this.$router.push({name:"main"})
                  .catch(err => {err})
            })
      }
    },

    async openMetamask() {
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

              this.adminCheck()
              this.loading = false
              this.walletConnectView = false
            })
            .catch(()=>{
              //unknown error, do nothing
              this.loading = false
              this.walletConnectError = true
            })
      } else {
        //already connected
        this.adminCheck()
        this.loading = false
        this.walletConnectView = false
      }
    },
    mint() {
      opt.mintReward()
    }
  }
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
@import "../../styles/colors.scss";
@import "../../styles/fonts.scss";

.mint-div {
  padding-top: 15%;
  padding-bottom: 20%;

  .wallet-btn-name {
    margin: 0 0 0 16px;
    color: $btnFontColor;
    letter-spacing: 2px;
    font-family: $fontTit;
    text-transform: capitalize;
    font-size: 18px;
  }
  .wallet-btn-icon {
    height: 40px;
    width: 40px;
  }
}
</style>
