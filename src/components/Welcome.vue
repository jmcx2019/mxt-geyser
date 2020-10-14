<template>
  <div class="row">
    <div class="col text-center">
      <img class="home-logo" :src="logo ? logo : '/static/actions/gift.png'" alt=""/>
      <div class="welcome-tit">{{ tit ? tit : $t("common.welcome") }}</div>
      <p class="welcome-txt">{{ txt ? txt : $t("common.welcomeTxt") }}</p>
    </div>
  </div>
</template>

<script>
export default {
  name: "Welcome",
  props: ["logo", "tit", "txt"],
}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
@import "../styles/colors.scss";
@import "../styles/fonts.scss";

.home-logo{
  width: 180px;
  height: 180px;
//margin-top: 30px;
//margin-bottom: 15px;
}

.welcome-tit {
  letter-spacing: 2px;
  color: $txtColor;
  font-size: 27px;
  font-family: $fontTit;
}
.welcome-txt {
  color: $txtColor;
  font-family: $fontTxt;
}
</style>
