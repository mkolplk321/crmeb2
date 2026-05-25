<template>
  <view class="user-center" :style="{ minHeight: pageHeight }">
    <!-- 头部 -->
    <view class="header-section" :style="colorStyle">
      <view class="sys-head">
        <view class="sys-bar" :style="{ height: sysHeight }"></view>
        <view class="sys-title">个人中心</view>
        <view class="bg" :style="member_style == 3 ? 'background:#f5f5f5' : ''"></view>
      </view>
    </view>

    <!-- 会员卡片 (Style0-4 对应原 homeUserInfor 的5种样式) -->
    <view class="member-card-wrap" :style="memberCardBgStyle">
      <!-- Style 0/2: 头像+信息 横排 -->
      <view class="card-header" v-if="styleConfig === 0 || styleConfig === 2" @click="goUserInfo">
        <view class="user-row" :class="styleConfig === 2 ? 'center' : ''">
          <view class="avatar-box">
            <image v-if="avatarUrl" :src="avatarUrl" mode="aspectFill" class="avatar" />
            <view v-else class="avatar-empty"><image src="https://goyoto.oss-cn-beijing.aliyuncs.com/images/shan.png" mode="aspectFill" /></view>
          </view>
          <view class="user-text" :class="styleConfig === 2 ? 'center-text' : ''">
            <text class="nickname">{{ userName }}</text>
            <text class="sub-text">{{ userSubText }}</text>
          </view>
        </view>
        <view class="menu-icons" v-if="menuList.length">
          <view class="menu-icon-item" v-for="(item, i) in menuList" :key="i" @click.stop="goLink(item)">
            <image v-if="item.img && menuStyle == 0" :src="item.img" mode="aspectFill" class="m-img" />
            <text v-else-if="item.icon" class="iconfont" :class="item.icon" :style="menuIconStyle"></text>
          </view>
        </view>
      </view>

      <!-- Style 1: 信息左+头像右 -->
      <view class="card-header" v-if="styleConfig === 1" @click="goUserInfo">
        <view class="user-row between">
          <view class="user-text">
            <text class="nickname">{{ userName }}</text>
            <text class="sub-text">{{ userSubTextPlain }}</text>
          </view>
          <view class="avatar-box">
            <image v-if="avatarUrl" :src="avatarUrl" mode="aspectFill" class="avatar" />
            <view v-else class="avatar-empty"><image src="https://goyoto.oss-cn-beijing.aliyuncs.com/images/shan.png" mode="aspectFill" /></view>
          </view>
        </view>
      </view>

      <!-- Style 3: 双卡片布局 -->
      <view class="style4-row" v-if="styleConfig === 3">
        <view class="left-card" :style="moduleCardStyle" @click="goUserInfo">
          <view class="user-row">
            <view class="avatar-box">
              <image v-if="avatarUrl" :src="avatarUrl" mode="aspectFill" class="avatar" />
              <view v-else class="avatar-empty"><image src="https://goyoto.oss-cn-beijing.aliyuncs.com/images/shan.png" mode="aspectFill" /></view>
            </view>
            <view class="user-text">
              <text class="nickname">{{ userName }}</text>
              <text class="sub-text">{{ userSubTextPlain }}</text>
            </view>
          </view>
          <view class="data-row" v-if="dataList.length">
            <view class="data-item" v-for="(item, i) in dataList" :key="i" @click.stop="handleDataNav(item)">
              <text class="d-num">{{ item.val }}</text>
              <text class="d-label">{{ item.name }}</text>
            </view>
          </view>
        </view>
        <view class="right-card" :style="moduleCardStyle" v-if="rightEntryList.length" @click="goLink(rightEntryList[0])">
          <text class="rc-title">{{ rightEntryLabel(rightEntryList[0]) }}</text>
          <text class="rc-sub">{{ rightEntrySub(rightEntryList[0]) }} ›</text>
          <image v-if="rightEntryList[0].img" :src="rightEntryList[0].img" mode="aspectFill" class="rc-img" />
        </view>
      </view>

      <!-- Style 4: 大卡片 -->
      <view class="style5-card" :style="cardStyle" v-if="styleConfig === 4">
        <view class="s5-header" @click="goUserInfo">
          <view class="s5-avatar-box">
            <image v-if="avatarUrl" :src="avatarUrl" mode="aspectFill" class="avatar big" />
            <view v-else class="avatar-empty big"><image src="https://goyoto.oss-cn-beijing.aliyuncs.com/images/shan.png" mode="aspectFill" /></view>
          </view>
          <view class="s5-user-text">
            <text class="nickname">{{ userName }}</text>
            <text class="sub-text">{{ userSubTextPlain }}</text>
          </view>
          <view class="menu-icons" v-if="menuList.length">
            <view class="menu-icon-item" v-for="(item, i) in menuList" :key="i" @click.stop="goLink(item)">
              <image v-if="item.img && menuStyle == 0" :src="item.img" mode="aspectFill" class="m-img" />
              <text v-else-if="item.icon" class="iconfont" :class="item.icon" :style="menuIconStyle"></text>
            </view>
          </view>
        </view>
      </view>

      <!-- 数据统计行 (Style 0/1/2/4 共用) -->
      <view class="card-data" v-if="dataList.length && styleConfig !== 3">
        <view class="data-item" v-for="(item, i) in dataList" :key="i" @click.stop="handleDataNav(item)">
          <text class="d-num">{{ item.val }}</text>
          <text class="d-label">{{ item.name }}</text>
        </view>
      </view>
    </view>

    <!-- 会员样式区：可提现/会员卡 -->
    <view class="member-extra" v-if="memberStyleConfig == 3">
      <view class="me4-row" :style="cardStyle">
        <view class="me4-left">
          <text class="me4-label">可提现(元)</text>
          <text class="me4-val">{{ withdrawAmount }}</text>
        </view>
        <text class="me4-btn" @click="goPage('/pages/users/user_cash/index')">立即提现</text>
      </view>
    </view>

    <!-- 订单状态 -->
    <view class="order-section">
      <view class="order-hd"><text class="o-title">我的订单</text><text class="o-all" @click="goPage('/pages/goods/order_list/index?status=9')">全部 ›</text></view>
      <view class="order-grid">
        <view class="o-item" v-for="(item, i) in orderMenu" :key="i" @click="goPage(item.url)">
          <view class="o-icon-wrap"><text class="iconfont" :class="item.img"></text><text class="o-badge" v-if="item.num > 0">{{ item.num > 99 ? '99+' : item.num }}</text></view>
          <text class="o-label">{{ item.title }}</text>
        </view>
      </view>
    </view>

    <!-- 功能菜单 -->
    <view class="menu-section" v-for="(menu, gIdx) in menuGroups" :key="gIdx">
      <view class="m-item" v-for="(item, i) in menu" :key="i" @click="goPage(item.url)">
        <text class="iconfont" :class="item.pic || 'icon-gerenzhongxin1'"></text>
        <text class="m-name">{{ item.name }}</text>
        <text class="m-arrow">›</text>
      </view>
    </view>

    <!-- 版权 -->
    <image :src="copyRightPic" class="support" />

    <editUserModal :isShow="editModal" @closeEdit="closeEdit" @editSuccess="editSuccess" />
    <pageFooter :style="colorStyle" />
  </view>
</template>

<script>
let sysHeight = uni.getWindowInfo().statusBarHeight + "px";
import { getMenuList, getUserInfo, setVisit } from "@/api/user.js";
import { getThemeInfo } from "@/api/api.js";
import { toLogin } from "@/libs/login.js";
import { mapGetters } from "vuex";
// #ifdef H5
import Auth from "@/libs/wechat";
// #endif
const app = getApp();
import colors from "@/mixins/color";
import pageFooter from "@/components/pageFooter/index.vue";
import editUserModal from "@/components/eidtUserModal/index.vue";
import { getCrmebCopyRight } from "@/api/api.js";

export default {
  components: { pageFooter, editUserModal },
  mixins: [colors],
  computed: {
    ...mapGetters({ cartNum: "cartNum", isLogin: "isLogin" }),
    avatarUrl() { return this.userInfo.avatar || ""; },
    userName() { return this.isLogin ? (this.userInfo.nickname || "") : "请点击登录"; },
    userSubText() {
      if (!this.isLogin) return "";
      if (this.userInfoConfig == 0) return this.userInfo.phone || "";
      return this.userInfo.uid ? "ID: " + this.userInfo.uid : "";
    },
    userSubTextPlain() {
      if (!this.isLogin) return "";
      if (this.userInfoConfig == 0) return this.userInfo.phone || "";
      return this.userInfo.uid || this.userInfo.id || "";
    },
    withdrawAmount() {
      if (!this.isLogin) return "0.00";
      return this.userInfo.brokerage_price || this.userInfo.brokeragePrice || this.userInfo.brokerage || 0;
    },
    menuGroups() {
      let groups = [];
      if (this.MyMenus.length) groups.push(this.MyMenus.map(m => ({...m, pic: m.pic || ''})));
      if (this.storeMenu.length) groups.push(this.storeMenu.map(m => ({...m, pic: m.pic || 'icon-shangjia2'})));
      return groups;
    },
  },
  data() {
    return {
      editModal: false, userInfo: {}, MyMenus: [], storeMenu: [],
      sysHeight, member_style: 0, memberStyleConfig: 0,
      styleConfig: 0, userInfoConfig: 0, dataList: [],
      orderMenu: [
        { img: "icon-daifukuan", title: "待付款", url: "/pages/goods/order_list/index?status=0" },
        { img: "icon-daifahuo", title: "待发货", url: "/pages/goods/order_list/index?status=1" },
        { img: "icon-daishouhuo", title: "待收货", url: "/pages/goods/order_list/index?status=2" },
        { img: "icon-daipingjia", title: "待评价", url: "/pages/goods/order_list/index?status=3" },
        { img: "icon-a-shouhoutuikuan", title: "售后/退款", url: "/pages/users/user_return_list/index" },
      ],
      pageHeight: "100%",
      // #ifdef APP-PLUS
      pageHeight: app.globalData.windowHeight,
      // #endif
      // #ifdef H5
      isWeixin: Auth.isWeixin(),
      //#endif
      copyRightPic: "https://goyoto.oss-cn-beijing.aliyuncs.com/images/support.png",
      menuList: [], menuStyle: 0, rightEntryList: [],
      checkType: [], assetMode: 0, moduleCardStyle: {}, cardStyle: {}, menuIconStyle: {},
      memberCardBgStyle: {},
    };
  },
  onLoad(option) {
    // #ifdef APP-PLUS
    this.pageHeight = app.globalData.windowHeight;
    // #endif
    // #ifdef H5
    if (this.isWeixin && option.code && option.scope === "snsapi_userinfo") {
      Auth.auth(option.code).then(() => this.getUserInfo()).catch(() => {});
    }
    // #endif
    this.getCopyRight();
  },
  onShow() {
    // #ifdef APP-PLUS
    let that = this; uni.getSystemInfo({ success: r => { that.pageHeight = r.windowHeight + "px"; } });
    // #endif
    if (this.isLogin) { this.getUserInfo(); }
    this.getMyMenus();
    this.getDiyData();
    this.getCopyRight();
  },
  onPullDownRefresh() { this.getUserInfo(); this.getMyMenus(); this.getDiyData(); uni.stopPullDownRefresh(); },
  methods: {
    goPage(url) {
      if (!url) return;
      if (!this.isLogin) return toLogin();
      if (url.indexOf("/pages/index/index") === 0 || url.indexOf("/pages/user/index") === 0) {
        uni.switchTab({ url });
      } else if (url.indexOf("switchTab:") === 0) {
        uni.switchTab({ url: url.replace("switchTab:", "") });
      } else {
        uni.navigateTo({ url });
      }
    },
    goUserInfo() {
      if (!this.isLogin) return toLogin();
      uni.navigateTo({ url: "/pages/users/user_info/index" });
    },
    goLink(item) {
      let url = (item && item.info && item.info[1] && item.info[1].value) || "";
      if (!url) return;
      if (!this.isLogin) return toLogin();
      if (this.$util && this.$util.JumpPath) { this.$util.JumpPath(url); return; }
      uni.navigateTo({ url });
    },
    rightEntryLabel(item) { return (item && item.info && item.info[0] && item.info[0].value) || ""; },
    rightEntrySub(item) { return (item && item.info && item.info[1] && item.info[1].value) || ""; },
    handleDataNav(item) {
      let map = {1:"/pages/users/user_money/index",3:"/pages/users/user_coupon/index",2:"/pages/users/user_integral/index",5:"/pages/users/user_goods_collection/index",6:"/pages/users/visit_list/index",8:"/pages/users/user_spread_money/index?type=2",9:"/pages/users/promoter-list/index",10:"/pages/users/promoter-order/index"};
      let url = map[item.id];
      if (url) { if (!this.isLogin) return toLogin(); uni.navigateTo({ url }); }
    },
    getDataVal(key) {
      if (!this.isLogin) return 0;
      let u = this.userInfo || {};
      let m = { money: u.now_money || u.nowMoney || 0, coupon: u.couponCount || u.coupon_num || 0, integral: u.integral || 0, collection: u.collectCount || u.collect_count || 0, visit: u.visitCount || u.visit_num || 0, brokerage: u.brokerage_price || u.brokeragePrice || u.brokerage || 0, spreadCount: u.spread_count || u.spreadCount || 0, spreadOrderCount: u.order_count || u.spread_order_count || u.spreadOrderCount || 0 };
      return m[key] || 0;
    },
    // DIY 数据解析
    getDiyData() {
      getThemeInfo("user", {}).then(res => {
        let d = res.data;
        if (!d || !d.value) return;
        let arr = Object.keys(d.value).sort().map(k => d.value[k]).sort((a,b) => (a.timestamp||0) - (b.timestamp||0));
        // 处理 member 组件数据
        let memberItem = arr.find(x => x.name === 'member');
        if (memberItem) {
          this.styleConfig = Number(memberItem.styleConfig && memberItem.styleConfig.tabVal) || 0;
          this.memberStyleConfig = Number(memberItem.memberStyleConfig && memberItem.memberStyleConfig.tabVal) || 0;
          this.userInfoConfig = Number(memberItem.userInfoConfig && memberItem.userInfoConfig.tabVal) || 0;
          this.menuList = (memberItem.menuConfig && memberItem.menuConfig.list) || [];
          this.menuStyle = Number(memberItem.menuConfig && memberItem.menuConfig.listStyle) || 0;
          this.checkType = (memberItem.checkboxInfo && memberItem.checkboxInfo.type) || [];
          this.assetMode = Number(memberItem.assetMode && memberItem.assetMode.tabVal) || 0;
          this.rightEntryList = (memberItem.rightEntryConfig && memberItem.rightEntryConfig.list) || [];
          // 数据行
          let allItems = [
            {id:1,name:"余额",key:"money"},{id:3,name:"优惠券",key:"coupon"},{id:2,name:"积分",key:"integral"},
            {id:5,name:"收藏商品",key:"collection"},{id:6,name:"浏览记录",key:"visit"},
            {id:8,name:"推广佣金",key:"brokerage"},{id:9,name:"推广人",key:"spreadCount"},{id:10,name:"推广订单",key:"spreadOrderCount"},
          ];
          this.dataList = allItems.filter(x => this.checkType.indexOf(x.id) !== -1).map(x => ({...x, val: this.getDataVal(x.key)}));
          // 样式
          this.menuIconStyle = { fontSize: "40rpx", color: "#333" };
          this.moduleCardStyle = { background: "#fff", borderRadius: "12rpx" };
          this.cardStyle = { background: "#fff", borderRadius: "12rpx" };
          this.memberCardBgStyle = {
            background: memberItem.cardBgColor && memberItem.cardBgColor.color ? `linear-gradient(90deg,${memberItem.cardBgColor.color[0].item} 0%,${(memberItem.cardBgColor.color[1]||memberItem.cardBgColor.color[0]).item} 100%)` : "#fff",
            borderRadius: "12rpx", margin: "20rpx", padding: "20rpx 0",
          };
        }
      });
    },
    getMyMenus() {
      getMenuList().then(res => {
        this.member_style = Number(res.data.diy_data.value);
        let orderSets = {
          1:{dfk:"icon-daifukuan",dfh:"icon-daifahuo",dsh:"icon-daishouhuo",dpj:"icon-daipingjia",sh:"icon-a-shouhoutuikuan"},
          2:{dfk:"icon-daifukuan-lan",dfh:"icon-daifahuo-lan",dsh:"icon-daishouhuo-lan",dpj:"icon-daipingjia-lan",sh:"icon-shouhoutuikuan-lan"},
          3:{dfk:"icon-daifukuan-ju",dfh:"icon-daifahuo-ju",dsh:"icon-daishouhuo-ju",dpj:"icon-daipingjia-ju",sh:"icon-shouhoutuikuan-ju"},
          4:{dfk:"icon-daifukuan-fen",dfh:"icon-daifahuo-fen",dsh:"icon-daishouhuo-fen",dpj:"icon-daipingjia-fen",sh:"icon-shouhoutuikuan-fen"},
          5:{dfk:"icon-daifukuan-lv",dfh:"icon-daifahuo-lv",dsh:"icon-daishouhuo-lv",dpj:"icon-daipingjia-lv",sh:"icon-shouhoutuikuan-lv"},
        };
        let order = orderSets[this.member_style] || orderSets[1];
        this.orderMenu.forEach(o => {
          let m = { "待付款":"dfk","待发货":"dfh","待收货":"dsh","待评价":"dpj","售后/退款":"sh" };
          o.img = order[m[o.title]] || o.img;
        });
        let storeM = [], myM = [];
        (res.data.routine_my_menus || []).forEach(el => {
          if (["/pages/admin/order/index","/pages/admin/order_cancellation/index","/pages/admin/manage/index"].includes(el.url) || el.name == "客服接待") {
            storeM.push(el);
          } else { myM.push(el); }
        });
        this.storeMenu = storeM;
        this.MyMenus = myM;
      });
    },
    getUserInfo() {
      getUserInfo().then(res => {
        this.userInfo = res.data;
        this.$store.commit("UPDATE_USERINFO", res.data);
        this.$store.commit("SETUID", res.data.uid);
        let m = { "待付款":"unpaid_count","待发货":"unshipped_count","待收货":"received_count","待评价":"evaluated_count","售后/退款":"refunding_count" };
        this.orderMenu.forEach(o => { o.num = res.data.orderStatusNum[m[o.title]] || 0; });
        uni.stopPullDownRefresh();
      });
    },
    getCopyRight() { getCrmebCopyRight().then(r => { if (r.data.copyrightImage) this.copyRightPic = r.data.copyrightImage; }); },
    editSuccess() { this.editModal = false; this.getUserInfo(); },
    closeEdit() { this.editModal = false; },
  },
};
</script>

<style lang="scss" scoped>
.user-center { background: #f5f5f5; padding-bottom: 120rpx; }
.sys-head { position: relative; width: 100%; background: #fff; z-index: 50;
  .sys-bar { width: 100%; }
  .sys-title { width: 100%; height: 43px; line-height: 43px; text-align: center; font-size: 32rpx; color: #000; font-weight: 500; position: relative; z-index: 10; }
  .bg { position: absolute; left: 0; top: 0; width: 100%; height: 100%; z-index: 5; }
}

.member-card-wrap { overflow: hidden; }
.card-header { position: relative; padding: 28rpx 20rpx; }
.user-row { display: flex; align-items: center;
  &.between { justify-content: space-between; width: 100%; }
  &.center { flex-direction: column; text-align: center; }
}
.avatar-box { flex-shrink: 0;
  .avatar { width: 90rpx; height: 90rpx; border-radius: 50%; display: block; }
  .avatar-empty { width: 90rpx; height: 90rpx; border-radius: 50%; background: #f3f9ff; border: 2rpx solid #eee; display: flex; align-items: center; justify-content: center;
    image { width: 52rpx; height: 40rpx; }
  }
}
.user-text { margin-left: 20rpx;
  .nickname { font-size: 32rpx; font-weight: 600; color: #333; display: block; }
  .sub-text { font-size: 24rpx; color: rgba(51,51,51,0.7); margin-top: 8rpx; display: block; }
  &.center-text { margin-left: 0; margin-top: 16rpx; text-align: center; }
}
.menu-icons { display: flex; align-items: center; position: absolute; right: 20rpx; top: 28rpx; z-index: 10;
  .menu-icon-item { margin-left: 20rpx; .m-img { width: 40rpx; height: 40rpx; display: block; } }
}

.card-data { display: flex; padding: 24rpx 0; border-top: 1rpx solid rgba(0,0,0,0.06); margin: 0 20rpx;
  .data-item { flex: 1; text-align: center; .d-num { font-size: 32rpx; font-weight: 600; color: #333; display: block; } .d-label { font-size: 22rpx; color: #999; margin-top: 6rpx; display: block; } }
}

.style4-row { display: flex; padding: 20rpx; align-items: stretch;
  .left-card { flex: 2; margin-right: 20rpx; padding: 20rpx; }
  .right-card { flex: 1; padding: 20rpx; display: flex; flex-direction: column; align-items: center; text-align: center;
    .rc-title { font-size: 28rpx; font-weight: 600; color: #333; }
    .rc-sub { font-size: 22rpx; color: #ff7d00; margin: 10rpx 0; }
    .rc-img { width: 100rpx; height: 100rpx; }
  }
  .data-row { display: flex; margin-top: 16rpx; .data-item { flex: 1; text-align: center; .d-num{font-size:28rpx;font-weight:600;} .d-label{font-size:20rpx;color:#999;margin-top:4rpx;} } }
}

.style5-card { padding: 40rpx; border-radius: 24rpx; margin: -90rpx 20rpx 0;
  .s5-header { position: relative; margin-bottom: 90rpx;
    .s5-avatar-box { position: absolute; top: -92rpx; left: 0;
      .avatar.big { width: 148rpx; height: 148rpx; border-radius: 50%; border: 4rpx solid #fff; box-shadow: 0 4rpx 20rpx rgba(0,0,0,0.1); }
      .avatar-empty.big { width: 148rpx; height: 148rpx; border-radius: 50%; background: #eee; display: flex; align-items: center; justify-content: center; border: 4rpx solid #fff;
        image { width: 60rpx; height: 60rpx; }
      }
    }
    .s5-user-text { position: absolute; top: -92rpx; left: 180rpx; .nickname{font-size:32rpx;font-weight:bold;} .sub-text{font-size:24rpx;opacity:0.8;margin-top:8rpx;} }
  }
}

.member-extra { margin: 0 20rpx; .me4-row { display: flex; align-items: center; justify-content: space-between; padding: 40rpx; margin-top: 36rpx;
  .me4-label { font-size: 24rpx; color: #999; } .me4-val { font-size: 48rpx; font-weight: 700; color: #f6d99d; display: block; margin-top: 10rpx; }
  .me4-btn { background: #f6d99d; color: #5a350c; padding: 10rpx 30rpx; border-radius: 40rpx; font-size: 24rpx; font-weight: 700; }
}}

.order-section { background: #fff; margin: 24rpx 20rpx; border-radius: 16rpx; padding: 30rpx 20rpx;
  .order-hd { display: flex; justify-content: space-between; margin-bottom: 24rpx; padding: 0 10rpx; .o-title { font-size: 30rpx; font-weight: bold; color: #333; } .o-all { font-size: 24rpx; color: #999; } }
  .order-grid { display: flex; justify-content: space-around; }
  .o-item { display: flex; flex-direction: column; align-items: center; .o-icon-wrap { position: relative; .iconfont { font-size: 44rpx; color: var(--view-theme, #e93323); } .o-badge { position: absolute; top: -10rpx; right: -16rpx; min-width: 32rpx; height: 32rpx; line-height: 32rpx; text-align: center; font-size: 18rpx; color: #fff; background: #e93323; border-radius: 16rpx; padding: 0 8rpx; } } .o-label { font-size: 22rpx; color: #666; margin-top: 10rpx; } }
}

.menu-section { background: #fff; margin: 0 20rpx 24rpx; border-radius: 16rpx; overflow: hidden;
  .m-item { display: flex; align-items: center; padding: 28rpx 30rpx; border-bottom: 1rpx solid #f5f5f5; &:last-child { border-bottom: none; } .iconfont { font-size: 36rpx; color: var(--view-theme, #e93323); width: 50rpx; text-align: center; } .m-name { flex: 1; font-size: 28rpx; color: #333; margin-left: 16rpx; } .m-arrow { font-size: 30rpx; color: #ccc; } }
}

.support { width: 219rpx; height: 74rpx; margin: 54rpx auto; display: block; }
</style>
