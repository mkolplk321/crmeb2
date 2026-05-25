<template>
  <view class="member-wrap">
    <common-wrapper :config="configData">
      <view class="member-card style2">
        <!-- Style 1: 头像右+信息左 -->
        <view class="card-header acea-row row-between-wrapper" @click="goUserInfo">
          <view class="user-info style-2 acea-row row-between-wrapper">
            <view class="text">
              <view class="name line1" :style="[nameTextStyle]">{{ userName }}</view>
              <view class="level line1" :style="[numTextStyle]">{{ userSubTextPlain }}</view>
            </view>
            <view class="avatar">
              <image v-if="avatarUrl" :src="avatarUrl" mode="aspectFill" />
              <view class="empty-box" v-else><image src="https://goyoto.oss-cn-beijing.aliyuncs.com/images/shan.png" mode="aspectFill" /></view>
            </view>
          </view>
        </view>
        <!-- memberStyleConfig==2: VIP区域 -->
        <view class="member-style-3" :style="[cardStyle, memberTopStyle]">
          <view class="content-wrapper" :style="[ms3ContainerStyle]">
            <view class="desc line1" :style="{ color: ms3TitleColor }">{{ ms3TitleText }}</view>
            <view class="btn line1" :style="{ color: ms3ButtonColor, borderColor: ms3ButtonColor }" @click="goLink('/pages/annex/vip_paid/index')">{{ ms3ButtonText }}</view>
          </view>
        </view>
      </view>
    </common-wrapper>
  </view>
</template>

<script>
import commonWrapper from "./commonWrapper.vue";
import { mapGetters } from "vuex";
export default {
  components: { commonWrapper },
  name: "homeMember",
  props: { dataConfig: { type: Object, default: () => ({}) } },
  computed: {
    ...mapGetters(["userInfo", "isLogin"]),
    configData() { let dc = this.dataConfig; let pad = dc.paddingConfig || { isAll: false, valList: [{ val: dc.topConfig ? dc.topConfig.val : 0 }, { val: dc.prConfig ? dc.prConfig.val : 0 }, { val: dc.bottomConfig ? dc.bottomConfig.val : 0 }, { val: dc.prConfig ? dc.prConfig.val : 0 }] }; let mar = dc.marginConfig || { isAll: false, valList: [{ val: dc.mbConfig ? dc.mbConfig.val : 0 }, { val: 0 }, { val: 0 }, { val: 0 }] }; return { ...dc, paddingConfig: { ...(dc.paddingConfig || {}), isAll: pad.isAll, valList: pad.valList }, marginConfig: { ...(dc.marginConfig || {}), isAll: mar.isAll, valList: mar.valList } }; },
    userInfoConfig() { return Number(this.dataConfig.userInfoConfig && this.dataConfig.userInfoConfig.tabVal) || 0; },
    avatarUrl() { return this.userInfo.avatar || ""; },
    nameTextStyle() { let c = this.dataConfig.nameColor && this.dataConfig.nameColor.color && this.dataConfig.nameColor.color[0] ? this.dataConfig.nameColor.color[0].item : "#333"; let s = this.dataConfig.nameSize && this.dataConfig.nameSize.val ? this.dataConfig.nameSize.val : 16; return { color: c, fontSize: s * 2 + "rpx" }; },
    numTextStyle() { let c = this.dataConfig.numColor && this.dataConfig.numColor.color && this.dataConfig.numColor.color[0] ? this.dataConfig.numColor.color[0].item : "#333"; let s = this.dataConfig.numSize && this.dataConfig.numSize.val ? this.dataConfig.numSize.val : 14; return { color: c, fontSize: s * 2 + "rpx" }; },
    cardStyle() { let s = { borderRadius: this.radiusFromConfig(this.dataConfig.cardBgRadius) }; let m = this.dataConfig.ms3BgMode ? this.dataConfig.ms3BgMode.tabVal : 0; if (m === 1 && this.dataConfig.ms3BackgroundImage && this.dataConfig.ms3BackgroundImage.url) { s.backgroundImage = `url(${this.dataConfig.ms3BackgroundImage.url})`; s.backgroundRepeat = "no-repeat"; s.backgroundSize = "100% 100%"; } else { let c1 = this.dataConfig.cardBgColor && this.dataConfig.cardBgColor.color && this.dataConfig.cardBgColor.color[0] ? this.dataConfig.cardBgColor.color[0].item : "#fff"; let c2 = this.dataConfig.cardBgColor && this.dataConfig.cardBgColor.color && this.dataConfig.cardBgColor.color[1] ? this.dataConfig.cardBgColor.color[1].item : c1; s.background = `linear-gradient(90deg, ${c1} 0%, ${c2} 100%)`; } return s; },
    memberTopStyle() { return {}; },
    ms3TitleText() { return (this.dataConfig.ms3TitleText && this.dataConfig.ms3TitleText.value) || ""; },
    ms3TitleColor() { return this.dataConfig.ms3TitleColor && this.dataConfig.ms3TitleColor.color && this.dataConfig.ms3TitleColor.color[0] ? this.dataConfig.ms3TitleColor.color[0].item : ""; },
    ms3ButtonText() { return (this.dataConfig.ms3ButtonText && this.dataConfig.ms3ButtonText.value) || ""; },
    ms3ButtonColor() { return this.dataConfig.ms3ButtonColor && this.dataConfig.ms3ButtonColor.color && this.dataConfig.ms3ButtonColor.color[0] ? this.dataConfig.ms3ButtonColor.color[0].item : ""; },
    ms3ContainerStyle() { let pad = this.dataConfig.ms3PaddingConfig && this.dataConfig.ms3PaddingConfig.valList ? this.dataConfig.ms3PaddingConfig.valList : [{ val: 0 }, { val: 0 }, { val: 0 }, { val: 0 }]; return { padding: `${pad[0].val * 2}rpx ${pad[1].val * 2}rpx ${pad[2].val * 2}rpx ${pad[3].val * 2}rpx` }; },
    userName() { if (!this.isLogin) return "请点击登录"; return (this.userInfo && this.userInfo.nickname) || ""; },
    userSubTextPlain() { if (!this.isLogin) return ""; if (this.userInfoConfig == 0) return (this.userInfo && this.userInfo.phone) || ""; return (this.userInfo && (this.userInfo.uid || this.userInfo.id)) || ""; },
  },
  methods: {
    radiusFromConfig(cfg) { if (!cfg) return "0rpx"; let t = Number(cfg.type) || 0; let v = Number(cfg.val) || 0; let l = cfg.valList || []; if (t && l.length >= 4) return `${l[0].val * 2}rpx ${l[1].val * 2}rpx ${l[3].val * 2}rpx ${l[2].val * 2}rpx`; return v * 2 + "rpx"; },
    goUserInfo() { if (!this.isLogin) { this.$emit("changeLogin"); return; } uni.navigateTo({ url: "/pages/users/user_info/index" }); },
    goLink(url) { if (!url) return; if (!this.isLogin) { this.$emit("changeLogin"); return; } if (this.$util && this.$util.JumpPath) { this.$util.JumpPath(url); return; } uni.navigateTo({ url }); },
  },
};
</script>

<style lang="scss" scoped>
.member-card { position: relative; overflow: hidden; .left-card .label { font-size: 22rpx; color: rgba(255, 255, 255, 0.8); margin-top: 0; } }
.card-header { position: relative; padding: 28rpx 20rpx; }
.user-info {
  .avatar { width: 90rpx; height: 90rpx; border-radius: 50%; margin-right: 20rpx; overflow: hidden; image { width: 100%; height: 100%; border-radius: 50%; } .empty-box { width: 100%; height: 100%; background: #f3f9ff; border: 2rpx solid #eee; display: flex; align-items: center; justify-content: center; image { width: 52rpx; height: 40rpx; border-radius: 0; } } }
  .text { flex: 1; .name { font-weight: 600; } .level { margin-top: 8rpx; opacity: 0.9; } }
  &.style-2 { width: 100%; padding: 0 20rpx; .avatar { margin-right: 0; margin-left: 20rpx; } }
}
.member-style-3 { margin-top: 36rpx; .content-wrapper { display: flex; align-items: center; justify-content: space-between; } .desc { font-size: 24rpx; } .btn { border: 2rpx solid; padding: 8rpx 24rpx; border-radius: 28rpx; font-size: 24rpx; } }
</style>
