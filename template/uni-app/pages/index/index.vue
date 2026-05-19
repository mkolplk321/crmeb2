<template>
  <!-- 轮回自行车 Club 首页 -->
  <view class="page" :style="{ minHeight: windowHeight + 'px' }">
    <!-- 全屏背景图（延伸到页面底部，底部渐变为白色） -->
    <view class="full-bg-wrap" :style="{ paddingTop: statusBarHeight + 'px' }">
      <image class="full-bg-img" src="/static/images/banner-bg.png" mode="aspectFill"></image>
      <view class="full-fade"></view>
    </view>

      <!-- 内容层 -->
      <view class="content-layer">
        <!-- 活动报名区域 - 大卡片样式 -->
        <view class="activity-section-new">
          <view class="activity-header">
            <text class="activity-title-text">活动报名</text>
            <text class="activity-subtitle">开启活力骑行人生</text>
          </view>
          <view class="activity-cards">
            <view class="activity-card" @click="goActivityList('riding')">
              <text class="activity-card-name">轮回特色骑行活动</text>
              <text class="activity-card-desc">探索人文与自然，享受骑行无界限</text>
              <view class="activity-card-btn">即刻探索</view>
            </view>
            <view class="activity-card" @click="goActivityList('race')">
              <text class="activity-card-name">轮回赛事报名</text>
              <text class="activity-card-desc">山川大地皆赛场，挑战永不停</text>
              <view class="activity-card-btn">即刻探索</view>
            </view>
          </view>
        </view>

        <!-- 6宫格功能入口 -->
        <view class="grid-section">
          <view class="grid-container">
            <view class="grid-card" v-for="(item, index) in gridList" :key="index" @click="goPage(item)">
              <image class="card-bg" :src="item.image" mode="aspectFill"></image>
              <view class="card-overlay">
                <text class="card-name">{{ item.name }}</text>
                <text class="card-sub" v-if="item.sub">{{ item.sub }}</text>
              </view>
            </view>
          </view>
        </view>

        <!-- 底部品牌区域 -->
        <view class="footer-section">
          <view class="brand-logo">
            <image class="logo-image" src="/static/images/brand-logo.png" mode="aspectFit"></image>
          </view>
          <view class="brand-name">闪电自行车 Club</view>
          <view class="copyright">© 2026 闪电自行车 Club All Rights Reserved</view>
        </view>
      </view>

      <!-- 优惠券弹窗 -->
    <couponWindow
      :window="isCouponShow"
      @onColse="couponClose"
      :couponImage="couponObj.image"
      :couponList="couponObj.list"
    ></couponWindow>

    <!-- 版权信息 -->
    <view v-if="configData && configData.record_No" class="site-config" @click="goICP(configData.icp_url)">
      {{ configData.record_No }}
    </view>
    <view class="site-config" v-if="configData && configData.network_security" @click="goICP(configData.network_security_url)">
      <image class="ban" src="/static/images/beian.png" alt="" />
      {{ configData.network_security }}
    </view>

    <!-- #ifdef APP -->
    <app-update ref="appUpdate" :force="true" :tabbar="false"></app-update>
    <!-- #endif -->
  </view>
</template>

<script>
const app = getApp();
import { getCouponV2, getCouponNewUser, getCrmebCopyRight, getIndexData } from "@/api/api.js";
import { getShare } from "@/api/public.js";
import { mapGetters, mapMutations } from "vuex";
import Cache from "@/utils/cache";
import { HTTP_REQUEST_URL } from "@/config/app";
// #ifdef APP
import appUpdate from "@/components/update/app-update.vue";
// #endif

export default {
  computed: {
    ...mapGetters(["isLogin", "uid"]),
  },
  components: {
    couponWindow: () => import("@/components/couponWindow/index"),
    // #ifdef APP
    appUpdate,
    // #endif
  },
  data() {
    return {
      statusBarHeight: 0,
      navBarHeight: 44,
      windowHeight: 0,
      bannerList: [
        // 默认轮播图，实际从API获取
        { pic: "/static/images/banner-brand.jpg", link: "" },
        { pic: "/static/images/banner-cycling.jpg", link: "" },
        { pic: "/static/images/banner-race.jpg", link: "" },
      ],
      gridList: [
        {
          name: "服务预约",
          sub: "预约骑行 We服务",
          image: "/static/images/icon-reservation.png",
          url: "/pages/goods_details/index?id=11",
          type: "navigateTo",
        },
        {
          name: "RETUL FIT",
          sub: "Fitting 服务预约",
          image: "/static/images/icon-fit.png",
          url: "/pages/goods_details/index?id=14",
          type: "navigateTo",
        },
        {
          name: "试骑体验",
          sub: "Demo Bike",
          image: "/static/images/icon-test.png",
          url: "/pages/goods_details/index?id=17",
          type: "navigateTo",
        },
        {
          name: "门店服务",
          sub: "Store Service",
          image: "/static/images/icon-store.png",
          url: "/pages/annex/special/index?theme_id=5",
          type: "navigateTo",
        },
        {
          name: "轮回周边",
          sub: "The Cyclist Club",
          image: "/static/images/icon-goods.png",
          url: "/pages/goods_details/index?id=12",
          type: "navigateTo",
        },
        {
          name: "关于我们",
          sub: "About Us...",
          image: "/static/images/icon-about.png",
          url: "/pages/annex/special/index?theme_id=17",
          type: "navigateTo",
        },
      ],
      isCouponShow: false,
      couponObj: {},
      shareInfo: {},
      configData: Cache.get("BASIC_CONFIG"),
      imgHost: HTTP_REQUEST_URL,
    };
  },
  onLoad(options) {
    let that = this;
    this.getOptions(options);
    this.$nextTick(function () {
      uni.getSystemInfo({
        success: function (res) {
          that.windowHeight = res.windowHeight;
          that.statusBarHeight = res.statusBarHeight || 0;
          // #ifdef MP-WEIXIN
          const menuButton = uni.getMenuButtonBoundingClientRect();
          that.navBarHeight = (menuButton.top - (res.statusBarHeight || 0)) * 2 + menuButton.height;
          // #endif
        },
      });
    });
    // 获取首页数据（包含banner）
    this.getIndexData();
    // 获取分享信息
    getShare().then((res) => {
      this.shareInfo = res.data;
    });
    // 获取版权信息
    this.getCopyRight();
    // 优惠券
    this.getCoupon();
  },
  onShow() {
    // 刷新时重新获取优惠券
    if (this.isLogin) {
      this.getCoupon();
    }
  },
  onPullDownRefresh() {
    this.getIndexData();
    uni.stopPullDownRefresh();
  },
  methods: {
    ...mapMutations(["SET_AUTOPLAY"]),
    getOptions(options) {
      let that = this;
      // #ifdef MP
      if (options.scene) {
        let value = that.$util.getUrlParams(decodeURIComponent(options.scene));
        if (value.spid) app.globalData.spid = value.spid;
      }
      // #endif
      if (options.spid) app.globalData.spid = options.spid;
    },
    // 获取首页数据
    getIndexData() {
      getIndexData()
        .then((res) => {
          // 如果有banner数据则使用
          if (res.data.banner) {
            this.bannerList = res.data.banner;
          }
        })
        .catch((err) => {
          console.log("获取首页数据失败", err);
        });
    },
    // 轮播图点击
    goBanner(item) {
      if (item.link) {
        // 如果有链接则跳转
        if (item.link.indexOf("http") !== -1) {
          // 外部链接
          // #ifdef H5
          window.location.href = item.link;
          // #endif
          // #ifdef MP
          uni.navigateTo({
            url: `/pages/annex/web_view/index?url=${encodeURIComponent(item.link)}`,
          });
          // #endif
        } else {
          // 内部链接处理
          uni.navigateTo({
            url: item.link,
          });
        }
      }
    },
    // 跳转活动列表
    goActivityList(type) {
      // 跳转到专题页面
      if (type === 'riding') {
        // 轮回特色骑行活动 -> id=20的专题页面
        uni.navigateTo({
          url: '/pages/annex/special/index?theme_id=20',
        });
      } else if (type === 'race') {
        // 赛事报名 -> id=22的专题页面
        uni.navigateTo({
          url: '/pages/annex/special/index?theme_id=22',
        });
      }
    },
    // 跳转功能页面
    goPage(item) {
      if (item.url) {
        if (item.type === "switchTab") {
          uni.switchTab({
            url: item.url,
          });
        } else {
          uni.navigateTo({
            url: item.url,
          });
        }
      }
    },
    // 获取版权信息
    getCopyRight() {
      getCrmebCopyRight()
        .then((res) => {
          let data = res.data;
          uni.setStorageSync("wechatStatus", data.wechat_status);
          if (!data.copyrightContext && !data.copyrightImage) {
            data.copyrightImage = "/static/images/support.png";
          }
          uni.setStorageSync("copyNameInfo", data.copyrightContext);
          uni.setStorageSync("copyImageInfo", data.copyrightImage);
        })
        .catch((err) => {
          console.log("获取版权信息失败", err);
        });
    },
    // ICP跳转
    goICP(url) {
      if (!url) return;
      // #ifdef H5
      window.open(url);
      // #endif
      // #ifdef MP
      uni.navigateTo({
        url: `/pages/annex/web_view/index?url=${encodeURIComponent(url)}`,
      });
      // #endif
    },
    // 优惠券弹窗
    getCoupon() {
      const tagDate = uni.getStorageSync("tagDate") || "",
        nowDate = new Date().toLocaleDateString();
      if (tagDate === nowDate) {
        this.getNewCoupon();
      } else {
        getCouponV2()
          .then((res) => {
            const { data } = res;
            if (data.list && data.list.length) {
              this.isCouponShow = true;
              this.couponObj = data;
              uni.setStorageSync("tagDate", new Date().toLocaleDateString());
            } else {
              this.getNewCoupon();
            }
          })
          .catch(() => {
            this.getNewCoupon();
          });
      }
    },
    // 新用户优惠券
    getNewCoupon() {
      const oldUser = uni.getStorageSync("oldUser") || 0;
      if (!oldUser) {
        getCouponNewUser()
          .then((res) => {
            const { data } = res;
            if (data.show) {
              if (data.list && data.list.length) {
                this.isCouponShow = true;
                this.couponObj = data;
                uni.setStorageSync("oldUser", 1);
              }
            } else {
              uni.setStorageSync("oldUser", 1);
            }
          })
          .catch(() => {});
      }
    },
    // 优惠券弹窗关闭
    couponClose() {
      this.isCouponShow = false;
      if (!uni.getStorageSync("oldUser")) {
        this.getNewCoupon();
      }
    },
    // 微信分享
    // #ifdef H5
    setOpenShare: function () {
      let that = this;
      let uid = this.uid ? this.uid : 0;
      if (that.$wechat.isWeixin()) {
        getShare().then((res) => {
          let data = res.data;
          let configAppMessage = {
            desc: data.synopsis,
            title: data.title,
            link: location.href + "?spid=" + uid,
            imgUrl: data.img,
          };
          that.$wechat.wechatEvevt(
            [
              "updateAppMessageShareData",
              "updateTimelineShareData",
              "onMenuShareAppMessage",
              "onMenuShareTimeline",
            ],
            configAppMessage
          );
        });
      }
    },
    // #endif
  },
  // #ifdef H5
  onShow() {
    this.setOpenShare();
  },
  // #endif
  // 分享
  //#ifdef MP
  onShareAppMessage() {
    let uid = this.uid ? this.uid : 0;
    if (this.shareInfo.img) {
      return {
        title: this.shareInfo.title || "轮回自行车 Club",
        path: "/pages/index/index?spid=" + uid,
        imageUrl: this.shareInfo.img,
        desc: this.shareInfo.synopsis,
      };
    } else {
      return {
        title: this.shareInfo.title || "轮回自行车 Club",
        path: "/pages/index/index?spid=" + uid,
      };
    }
  },
  onShareTimeline: function () {
    return {
      title: this.shareInfo.title || "轮回自行车 Club",
      path: "/pages/index/index",
      imageUrl: this.shareInfo.img,
    };
  },
  //#endif
};
</script>

<style lang="scss">
.page {
  background-color: transparent;
}

/* 全屏背景图 */
.full-bg-wrap {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
}

.full-bg-img {
  width: 100%;
  height: 100%;
}

.full-fade {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 60%;
  background: linear-gradient(
    to bottom,
    rgba(255, 255, 255, 0) 0%,
    rgba(255, 255, 255, 0.5) 30%,
    rgba(255, 255, 255, 1) 100%
  );
}

/* 内容层 */
.content-layer {
  position: relative;
  z-index: 1;
  padding-top: 80rpx;
}

/* 活动报名区域 - 设计稿大卡片样式 */
.activity-section-new {
  margin-top: 40rpx;
  padding: 40rpx 32rpx;
}

.activity-header {
  text-align: center;
  margin-bottom: 36rpx;
}

.activity-title-text {
  font-size: 36rpx;
  font-weight: bold;
  color: #fff;
  display: block;
}

.activity-subtitle {
  font-size: 24rpx;
  color: rgba(255,255,255,0.7);
  display: block;
  margin-top: 6rpx;
}

.activity-cards {
  display: flex;
  justify-content: space-between;
}

.activity-card {
  width: 48%;
  border-radius: 16rpx;
  overflow: hidden;
  padding: 36rpx 24rpx;
  position: relative;

  &:nth-child(1) {
    background: linear-gradient(135deg, #2b2d32, #3a3c42);
    color: #fff;
    box-shadow: 0 8rpx 24rpx rgba(0, 0, 0, 0.2);
  }

  &:nth-child(2) {
    background: linear-gradient(135deg, #1a1a1a, #333);
    color: #fff;
    box-shadow: 0 8rpx 24rpx rgba(0, 0, 0, 0.25);
  }

  &:active {
    opacity: 0.85;
  }
}

.activity-card-name {
  font-size: 28rpx;
  font-weight: bold;
  color: #fff;
  display: block;
  line-height: 1.4;
}

.activity-card-desc {
  font-size: 22rpx;
  color: rgba(255,255,255,0.6);
  display: block;
  margin-top: 10rpx;
  line-height: 1.4;
}

.activity-card-btn {
  display: inline-block;
  margin-top: 20rpx;
  padding: 8rpx 28rpx;
  font-size: 22rpx;
  color: #333;
  background-color: #d4c598;
  border-radius: 30rpx;
  font-weight: 500;
}

/* 6宫格功能入口 - 图片卡片 */
.grid-section {
  padding: 40rpx 32rpx;
  margin-top: 30rpx;
}

.grid-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
}

.grid-card {
  width: 32%;
  height: 260rpx;
  position: relative;
  border-radius: 12rpx;
  overflow: hidden;
  margin-bottom: 20rpx;

  &:active {
    opacity: 0.85;
    transform: scale(0.98);
  }
}

.card-bg {
  width: 100%;
  height: 100%;
  display: block;
}

.card-overlay {
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0;
  padding: 16rpx 14rpx;
  background: linear-gradient(transparent, rgba(0, 0, 0, 0.6));
  display: flex;
  flex-direction: column;
}

.card-name {
  font-size: 26rpx;
  font-weight: bold;
  color: #fff;
  line-height: 1.3;
}

.card-sub {
  font-size: 20rpx;
  color: rgba(255, 255, 255, 0.8);
  margin-top: 4rpx;
}

/* 底部品牌区域 */
.footer-section {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 80rpx 32rpx;
  margin-top: 40rpx;
}

.brand-logo {
  margin-bottom: 20rpx;
}

.logo-image {
  width: 160rpx;
  height: 80rpx;
}

.brand-name {
  font-size: 32rpx;
  font-weight: bold;
  color: #333;
  margin-bottom: 10rpx;
}

.copyright {
  font-size: 22rpx;
  color: #999;
}

/* 版权信息 */
.site-config {
  margin: 30rpx 0;
  font-size: 24rpx;
  text-align: center;
  color: #666;
  display: flex;
  align-items: center;
  justify-content: center;
  
  .ban {
    width: 22rpx;
    height: 24rpx;
    margin-right: 10rpx;
  }
}
</style>
