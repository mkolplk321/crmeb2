<template>
  <view class="page">
    <view class="header">
      <image class="banner" src="/static/images/banner-cycling.jpg" mode="aspectFill"></image>
    </view>
    <view class="section-title">活动报名</view>
    <view class="activity-cards">
      <view class="card" @click="goActivity('riding')">
        <view class="card-bg">
          <image src="/static/images/banner-race.jpg" mode="aspectFill"></image>
        </view>
        <view class="card-content">
          <view class="card-title">闪电特色骑行活动</view>
          <view class="card-desc">九条·黄贡椒山水之旅、骑行·美食的盛宴</view>
          <view class="card-btn">即刻探索</view>
        </view>
      </view>
      <view class="card" @click="goActivity('race')">
        <view class="card-bg">
          <image src="/static/images/banner-race.jpg" mode="aspectFill"></image>
        </view>
        <view class="card-content">
          <view class="card-title">闪电赛事报名</view>
          <view class="card-desc">山马&公路赛、更精彩</view>
          <view class="card-btn">即刻探索</view>
        </view>
      </view>
    </view>
    <view class="grid-section">
      <view class="grid-item" v-for="(item, index) in gridList" :key="index" @click="goPage(item)">
        <image :src="item.image" mode="aspectFill"></image>
        <view class="grid-text">{{ item.title }}</view>
        <view class="grid-subtitle">{{ item.subtitle }}</view>
      </view>
    </view>
    <view class="footer-logo">
      <image src="/static/images/brand-logo.png" mode="aspectFit"></image>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      gridList: [
        { title: '服务预约', subtitle: '贴心好服务 轻松搞定', image: '/static/images/icon-reservation.jpg', page: '' },
        { title: 'RETUL FIT', subtitle: 'Fitting提升体验', image: '/static/images/icon-fit.jpg', page: '' },
        { title: '试骑体验', subtitle: 'Demo Day', image: '/static/images/icon-test.jpg', page: '' },
        { title: '门店服务', subtitle: 'Retail Services', image: '/static/images/icon-store.jpg', page: '' },
        { title: '闪电周边', subtitle: 'The Cycles Club', image: '/static/images/icon-goods.jpg', page: '' },
        { title: '关于我们', subtitle: 'About Us', image: '/static/images/icon-about.jpg', page: '' }
      ]
    }
  },
  methods: {
    goActivity(type) {
      const themeId = type === 'race' ? 22 : 20;
      uni.navigateTo({
        url: `/pages/annex/special/index?theme_id=${themeId}`
      });
    },
    goPage(item) {
      if (item.page) {
        uni.navigateTo({ url: item.page });
      }
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
  width: 100%;
  height: 400rpx;
  .banner {
    width: 100%;
    height: 100%;
  }
}
.section-title {
  padding: 30rpx;
  font-size: 36rpx;
  font-weight: bold;
  color: #333;
}
.activity-cards {
  padding: 0 30rpx;
  display: flex;
  gap: 20rpx;
  .card {
    flex: 1;
    background: #1a1a1a;
    border-radius: 16rpx;
    overflow: hidden;
    .card-bg {
      height: 180rpx;
      image {
        width: 100%;
        height: 100%;
        opacity: 0.6;
      }
    }
    .card-content {
      padding: 20rpx;
      .card-title {
        color: #fff;
        font-size: 28rpx;
        font-weight: bold;
      }
      .card-desc {
        color: #999;
        font-size: 22rpx;
        margin: 10rpx 0;
      }
      .card-btn {
        display: inline-block;
        background: transparent;
        border: 1rpx solid #fff;
        color: #fff;
        font-size: 24rpx;
        padding: 6rpx 24rpx;
        border-radius: 30rpx;
        margin-top: 10rpx;
      }
    }
  }
}
.grid-section {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16rpx;
  padding: 30rpx;
  .grid-item {
    border-radius: 12rpx;
    overflow: hidden;
    position: relative;
    height: 220rpx;
    image {
      width: 100%;
      height: 100%;
    }
    .grid-text {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -60%);
      color: #fff;
      font-size: 28rpx;
      font-weight: bold;
      text-shadow: 0 2rpx 8rpx rgba(0,0,0,0.5);
      white-space: nowrap;
    }
    .grid-subtitle {
      position: absolute;
      top: 62%;
      left: 50%;
      transform: translateX(-50%);
      color: rgba(255,255,255,0.8);
      font-size: 20rpx;
      white-space: nowrap;
    }
  }
}
.footer-logo {
  display: flex;
  justify-content: center;
  padding: 40rpx;
  image {
    width: 300rpx;
    height: 80rpx;
  }
}
</style>