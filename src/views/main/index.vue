<template>
  <div class="container-fluid main-container">
    <welcome></welcome>
    <wallet></wallet>

    <div class="row">
      <div class="col text-center">
        <div class="security-tit">{{ $t("security.audit") }}</div>
        <img class="security-logo" :src="'/static/security/lianan.png'" alt=""/>
      </div>
    </div>
  </div>
</template>

<script>
import Wallet from "./Wallet";
import Welcome from "./../../components/Welcome";

export default {
  components: { Wallet, Welcome },
};
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
@import "../../styles/colors.scss";
@import "../../styles/fonts.scss";

.main-container {
  height: 100%;

  .security-tit {
    margin-top: 45px;
    color: $txtColor;
    font-family: $fontTit;
  }
  .security-logo {
    margin-top: 10px;
    height: 35px;
  }
}
</style>
