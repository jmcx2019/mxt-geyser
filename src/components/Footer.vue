<template>
  <div class="footer-background">
    <div class="row">
      <div class="col text-center fa-icons">
        <a :href="$t('links.twitter')" target="_blank"><i class="fa fa-twitter" aria-hidden="true"></i></a>
        <a :href="$t('links.facebook')" target="_blank"><i class="fa fa-facebook" aria-hidden="true"></i></a>
        <a :href="$t('links.telegram')" target="_blank"><i class="fa fa-telegram" aria-hidden="true"></i></a>
        <a :href="$t('links.email')" target="_blank"><i class="fa fa-envelope-o" aria-hidden="true"></i></a>
      </div>
    </div>
    <div class="row">
      <div class="col text-center copyright-col">
        <a :href="$t('links.contract')" target="_blank">{{ $t("footer.copyRight") }}</a>
      </div>
    </div>
  </div>
</template>

<script>
	export default {
		name: "Footer",
	}
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
@import "../styles/colors.scss";
@import "../styles/fonts.scss";

.footer-background {
  color: $txtColor;
  font-family: $fontTxt;
  padding: 60px 0 15px;

  .footer-logo {
    height: 32px;
  }
  .footer-col {
    margin: 45px 0;
  }
  .fa-icons {
    margin-bottom: 15px;
  }
  .fa-icons > a {
    font-size: 27px;
    color: $txtColor;
    margin: 20px 20px;
  }
  .copyright-col > a {
    font-size: 16px;
    color: $txtColor;
  }
}
</style>
