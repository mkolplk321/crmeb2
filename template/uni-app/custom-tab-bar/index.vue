<template>
  <view class="custom-tab-bar" :style="{ backgroundColor: bgColor }">
    <view
      class="tab-item"
      v-for="(item, index) in tabList"
      :key="index"
      @click="switchTab(item, index)"
    >
      <image
        class="tab-icon"
        :src="selected === index ? item.selectedIconPath : item.iconPath"
        mode="aspectFit"
      />
      <text class="tab-text" :class="{ active: selected === index }">
        {{ item.text }}
      </text>
      <view
        v-if="item.badge && item.badge > 0"
        class="tab-badge"
      >{{ item.badge > 99 ? '99+' : item.badge }}</view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      selected: 0,
      bgColor: '#f8f8f8',
      tabList: [
        {
          pagePath: '/pages/index/index',
          iconPath: '/static/images/tabbar/home.png',
          selectedIconPath: '/static/images/tabbar/home-active.png',
          text: '首页',
          badge: 0,
        },
        {
          pagePath: '/pages/special/index',
          iconPath: '/static/images/tabbar/activity.png',
          selectedIconPath: '/static/images/tabbar/activity-active.png',
          text: '活动',
          badge: 0,
        },
        {
          pagePath: '/pages/reservation/list',
          iconPath: '/static/images/tabbar/reserve.png',
          selectedIconPath: '/static/images/tabbar/reserve-active.png',
          text: '预约',
          badge: 0,
        },
        {
          pagePath: '/pages/user/index',
          iconPath: '/static/images/tabbar/user.png',
          selectedIconPath: '/static/images/tabbar/user-active.png',
          text: '我的',
          badge: 0,
        },
      ],
    };
  },
  created() {
    this.updateSelected();
    // 监听购物车数量变化
    uni.$on('cartNumChange', this.onCartNumChange);
  },
  beforeDestroy() {
    uni.$off('cartNumChange', this.onCartNumChange);
  },
  methods: {
    switchTab(item, index) {
      if (this.selected === index) return;
      uni.switchTab({
        url: item.pagePath,
      });
    },
    updateSelected() {
      const pages = getCurrentPages();
      if (pages.length > 0) {
        const currentPage = pages[pages.length - 1];
        const route = '/' + currentPage.route;
        const idx = this.tabList.findIndex((item) => item.pagePath === route);
        if (idx !== -1) {
          this.selected = idx;
        }
      }
    },
    onCartNumChange(num) {
      // 购物车角标显示在第4个tab（我的）上，或根据实际需求调整
      if (num !== undefined) {
        const cartIndex = this.tabList.findIndex((item) => item.text === '我的');
        if (cartIndex !== -1) {
          this.tabList[cartIndex].badge = num;
        }
      }
    },
  },
};
</script>

<style scoped>
.custom-tab-bar {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  height: 100rpx;
  display: flex;
  justify-content: space-around;
  align-items: center;
  border-top: 1rpx solid #e5e5e5;
  z-index: 9999;
  padding-bottom: env(safe-area-inset-bottom);
  box-sizing: content-box;
}

.tab-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  flex: 1;
  height: 100%;
  position: relative;
  padding-top: 10rpx;
}

.tab-icon {
  width: 44rpx;
  height: 44rpx;
  margin-bottom: 4rpx;
}

.tab-text {
  font-size: 20rpx;
  color: #666666;
  line-height: 1;
}

.tab-text.active {
  color: #333333;
}

.tab-badge {
  position: absolute;
  top: 6rpx;
  right: 30rpx;
  min-width: 32rpx;
  height: 32rpx;
  line-height: 32rpx;
  text-align: center;
  font-size: 18rpx;
  color: #fff;
  background-color: #e93323;
  border-radius: 16rpx;
  padding: 0 8rpx;
}
</style>
