<template>
  <view class="page">
    <!-- 订单列表页面内容 -->
    <view class="header bg-color">
      <view class="picTxt acea-row row-between-wrapper">
        <view class="text">
          <view class="name">订单信息</view>
          <view><text class="mr-40">消费订单：{{ orderData.order_count || 0 }} 单 </text>总消费：￥{{ orderData.sum_price || 0 }}</view>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      orderData: {
        order_count: 0,
        sum_price: 0
      }
    }
  },
  onLoad() {
    this.getOrderData();
  },
  methods: {
    getOrderData() {
      // 跳转到订单列表
      uni.navigateTo({
        url: '/pages/goods/order_list/index'
      });
    }
  }
}
</script>

<style lang="scss" scoped>
.page {
  min-height: 100vh;
  background-color: #f5f5f5;
}
.header {
  padding: 30rpx;
  color: #fff;
}
.name {
  font-size: 32rpx;
  font-weight: bold;
  margin-bottom: 10rpx;
}
.mr-40 {
  margin-right: 20rpx;
}
.bg-color {
  background-color: #333;
}
</style>