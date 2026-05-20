<template>
  <view class="activity-page">
    <view class="content">
      <image src="/static/images/tabbar/activity-active.png" mode="aspectFit" class="icon"></image>
      <text class="title">活动专区</text>
      <text class="desc">点击查看最新活动</text>
      <button class="btn" @click="goActivity">进入活动</button>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {};
  },
  onLoad() {
    // 自动跳转到专题页面
    this.goActivity();
  },
  methods: {
    goActivity() {
      uni.navigateTo({
        url: '/pages/annex/special/index',
        fail: () => {
          // 如果跳转失败，显示当前页面
        }
      });
    },
  },
};
</script>

<style scoped>
.activity-page {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  background: #f5f5f5;
}
.content {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 40rpx;
}
.icon {
  width: 200rpx;
  height: 200rpx;
  margin-bottom: 30rpx;
}
.title {
  font-size: 36rpx;
  font-weight: bold;
  color: #333;
  margin-bottom: 20rpx;
}
.desc {
  font-size: 28rpx;
  color: #999;
  margin-bottom: 40rpx;
}
.btn {
  width: 400rpx;
  height: 80rpx;
  line-height: 80rpx;
  background: linear-gradient(90deg, #e93323, #ff6b35);
  color: #fff;
  font-size: 30rpx;
  border-radius: 40rpx;
  border: none;
}
</style>
