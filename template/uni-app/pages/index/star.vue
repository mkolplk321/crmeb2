<template>
  <view class="star-page">
    <view class="star-btn" @click="goStar">
      <image src="/static/images/tabbar/star.png" mode="aspectFit"></image>
    </view>
  </view>
</template>

<script>
export default {
  methods: {
    goStar() {
      uni.navigateTo({
        url: '/pages/annex/special/index?theme_id=20'
      });
    }
  }
}
</script>

<style lang="scss" scoped>
.star-page {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  .star-btn {
    width: 120rpx;
    height: 120rpx;
    image {
      width: 100%;
      height: 100%;
    }
  }
}
</style>