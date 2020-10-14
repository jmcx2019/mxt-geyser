<template>
  <div class="product-div">
    <div v-show="productData.double" class="double-reward"><b>{{ $t("common.dReward") }}</b></div>
    <div class="row">
      <div class="col">
        <div class="product-logo-div">
          <img class="product-logo" :src="productData.img" alt=""/>
        </div>
      </div>
    </div>

    <p class="pdt-name">{{ productData.name }}</p>
    <div class="pdt-txt">{{ $t("product.deposit") + productData.tx + $t("product.uni")}}</div>
    <div class="pdt-txt">{{ $t("product.earn") + $t("token.name") }}</div>

    <div class="row">
      <div class="col">
        <countdown :dateTime="productData.dateTime"></countdown>
      </div>
    </div>

    <div class="row">
      <input type="submit" v-show="productData.stat" class="btn DeFi-btn select-btn"
             :value="$t('product.select')"
             id="selectBtn"
             @click="select"
      >
      <input type="submit" v-show="!productData.stat" class="btn DeFi-btn select-btn"
             disabled="disabled"
             :value="$t('product.comingSoon')"
      >
    </div>

    <div class="row pdt-apy">
      <div class="col text-left">
        {{ $t("common.apy") }}
        <div></div>
      </div>
      <div class="col text-right">
        <div>{{ productData.apy }}%</div>
      </div>
    </div>
  </div>
</template>

<script>
import chainOpt from "@/utils/web3/chainOperation";
import Countdown from "../../components/Countdown";
import $ from "jquery";

let opt = chainOpt.opt

export default {
  name: "Product",
  components: {Countdown},
  props: ["productData", "productIndex"],
  data() {
    return {
      querying: false,
      poolInfo: null,
    }
  },
  mounted() {
    if (this.productData.stat) {
      this.loopGetInfo()
      setInterval(async _ => {
        await this.loopGetInfo()
      }, 5000)
      $(this.$el).find("#selectBtn").removeAttr("disabled")
    } else {
      $(this.$el).find("#selectBtn").attr("disabled", "disabled")
    }
  },

  methods: {
    select() {
      this.$router.push({
        name: "actions",
        params: {
          tx_id: this.productData.name,
          pool_info: this.poolInfo, pool_idx: this.productIndex
        }
      });
    },

    async loopGetInfo() {
      if (this.querying) {
        return
      }

      this.querying = true
      this.poolInfo = await opt.poolInfo(this.productIndex)
      window.allPoolInfo[this.productIndex] = this.poolInfo
      this.querying = false
    }
  }
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
@import "../../styles/colors.scss";
@import "../../styles/fonts.scss";

.product-div {
  border-width: 1px;
  border-style: solid;
  border-color: $divBorderColor;
  border-image: initial;
  border-radius: 10px;
  padding: 25px 15px 15px ;
  margin-top: 15px !important;
  background-color: $divBg;

  .double-reward {
    position: absolute;
    right: 25px;
    top: 25px;
    font-size: 12px;
    color: $errColor;
    font-family: $fontTxt;
  }

  .product-logo {
    width: 140px;
    height: 140px;
  }
  .product-logo-div {
    //background-color: $bg2;
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

  .pdt-txt {
    font-family: $fontTxt;
    font-size: 16px;
    font-weight: 400;
    color: $titL2Color;
  }

  .select-btn {
    width: 100%;
    height: 56px;
    margin: 20px 0 !important;
  }

  .pdt-apy {
    background-color: $divBg;
    color: $titL2Color;
    border-width: 1px;
    border-style: solid;
    border-color: $titL2Color;
    border-image: initial;
    border-radius: 5px;
    font-size: 12px;
    padding: 5px;
  }
}
</style>
