<template>
  <view class="page-design" :class="bgClass">
    <view v-if="!errorNetwork" :style="colorStyle">
      <view class="index">
        <block v-for="(item, index) in styleConfig" :key="index">
          <view :id="item.id">
            <userInfor
              v-if="item.name == 'userInfor'"
              :dataConfig="item"
              @changeLogin="changeLogin"
            ></userInfor>
            <homeUserInfor
              v-else-if="item.name == 'member'"
              :dataConfig="item"
              @changeLogin="changeLogin"
            ></homeUserInfor>
            <newVip
              v-else-if="item.name == 'newVip'"
              :dataConfig="item"
            ></newVip>
            <articleList
              v-else-if="item.name == 'articleList'"
              :dataConfig="item"
            ></articleList>
            <blankPage
              v-else-if="item.name == 'blankPage'"
              :dataConfig="item"
            ></blankPage>
            <coupon
              v-else-if="item.name == 'coupon'"
              :dataConfig="item"
              @changeLogin="changeLogin"
            ></coupon>
            <customerService
              v-else-if="item.name == 'customerService'"
              :dataConfig="item"
            ></customerService>
            <goodList
              ref="goodLists"
              v-else-if="item.name == 'goodList' || item.name == 'goodRecommend'"
              :dataConfig="item"
              :list="goodList"
            ></goodList>
            <menus v-else-if="item.name == 'menus'" :dataConfig="item"></menus>
            <pictureCube
              v-else-if="item.name == 'pictureCube'"
              :dataConfig="item"
            ></pictureCube>
            <richText
              v-else-if="item.name == 'richText'"
              :dataConfig="item"
            ></richText>
            <signIn
              v-else-if="item.name == 'signIn'"
              :dataConfig="item"
            ></signIn>
            <customComponent
              v-else-if="item.name == 'customComponent'"
              :dataConfig="item"
              @changeLogin="changeLogin"
            ></customComponent>
          </view>
        </block>

        <slot name="bottom"></slot>

        <view class="pb-safe" :style="[pdHeights]" v-if="isFooter"></view>
        <pageFooter
          v-if="footerConfigData"
          :configData="footerConfigData"
          @newDataStatus="newDataStatus"
        ></pageFooter>
      </view>
    </view>

    <view v-else>
      <view class="error-network">
        <image :src="imgHost + '/statics/images/error-network.gif'"></image>
        <view class="title">{{ $t('网络连接断开') }}</view>
        <view class="btn" @click="reconnect">{{ $t('重新连接') }}</view>
      </view>
    </view>
  </view>
</template>

<script>
import pageFooter from "@/components/pageFooter/index.vue";
import { HTTP_REQUEST_URL } from "@/config/app";
import colors from "@/mixins/color";
// 个人中心精简版：仅引入13个组件（完整版36个）
import userInfor from "./userInfor.vue";
import homeUserInfor from "./homeUserInfor.vue";
import newVip from "./newVip.vue";
import articleList from "./articleList.vue";
import blankPage from "./blankPage.vue";
import coupon from "./coupon.vue";
import customerService from "./customerService.vue";
import goodList from "./goodList.vue";
import menus from "./menus.vue";
import pictureCube from "./pictureCube.vue";
import richText from "./richText.vue";
import signIn from "./signIn.vue";
import customComponent from "./customComponent.vue";

export default {
  name: "PageDesignUser",
  components: {
    pageFooter,
    userInfor,
    homeUserInfor,
    newVip,
    articleList,
    blankPage,
    coupon,
    customerService,
    goodList,
    menus,
    pictureCube,
    richText,
    signIn,
    customComponent,
  },
  mixins: [colors],
  props: {
    diyData: { type: Object, default: () => ({}) },
    isHome: { type: Boolean, default: false },
    isScrolled: { type: Boolean, default: false },
    isFixed: { type: Boolean, default: false },
    productData: { type: Object, default: () => ({}) },
    priceData: { type: Object, default: () => ({}) },
    skuList: { type: Array, default: () => [] },
    reply: { type: Array, default: () => [] },
    replyCount: { type: Number, default: 0 },
    replyChance: { type: [String, Number], default: 0 },
    productId: { type: [Number, String], default: 0 },
    goodList: { type: Array, default: () => [] },
    productVideoStatus: { type: Boolean, default: false },
    belongIndex: { type: Number, default: 0 },
    errorNetwork: { type: Boolean, default: false },
    couponList: { type: Array, default: () => [] },
    activity: { type: Array, default: () => [] },
    attr: { type: Object, default: () => ({}) },
    attrTxt: { type: String, default: "" },
    attrValue: { type: String, default: "" },
    microPage: { type: Boolean, default: false },
    isShowPaidVip: { type: Boolean, default: false },
  },
  data() {
    return {
      styleConfig: [],
      footerConfigData: null,
      bgColor: "",
      bgPic: "",
      bgTabVal: "",
      isFooter: false,
      pdHeight: 0,
      imgHost: HTTP_REQUEST_URL,
    };
  },
  computed: {
    pageStyle() {
      return {
        backgroundColor: this.bgColor,
        backgroundImage: this.bgPic ? `url(${this.bgPic})` : "",
        minHeight: "100vh",
      };
    },
    bgClass() {
      if (this.bgTabVal == 2) return "fullsize noRepeat";
      if (this.bgTabVal == 1) return "repeat ysize";
      return "noRepeat ysize";
    },
    pdHeights() {
      let H = `${this.pdHeight * 2 + 100}rpx`;
      return { height: this.isFooter ? H : "100rpx" };
    },
  },
  watch: {
    diyData: {
      handler(val) {
        if (val && Object.keys(val).length > 0) {
          this.setDiyData(val);
        } else {
          this.styleConfig = [];
          this.footerConfigData = null;
          this.bgColor = "";
          this.bgPic = "";
          this.bgTabVal = "";
        }
      },
      deep: true,
      immediate: true,
    },
  },
  methods: {
    reconnect() {
      this.$emit("reconnect");
    },
    onChangeSpec(item) {
      this.$emit("changeSpec", item);
    },
    onShowSpecModal() {
      this.$emit("showSpecModal");
    },
    onShowCoupon() {
      this.$emit("showCoupon");
    },
    onOpenModal(type) {
      this.$emit("openModal", type);
    },
    onGoActivity(item) {
      this.$emit("goActivity", item);
    },
    onShare() {
      this.$emit("share");
    },
    objToArr(data) {
      if (!data) return [];
      let obj = Object.keys(data).sort();
      let m = obj.map((key) => data[key]);
      return m;
    },
    setDiyData(data) {
      if (!data) return;
      if (data.is_bg_color) {
        this.bgColor = data.color_picker;
      }
      if (data.is_bg_pic) {
        let bgPic = data.bg_pic || "";
        if (bgPic && !bgPic.startsWith("http")) {
          bgPic = HTTP_REQUEST_URL + bgPic;
        }
        this.bgPic = bgPic;
        this.bgTabVal = data.bg_tab_val;
      }

      let temp = [];
      this.footerConfigData = null;
      if (data.value) {
        let lastArr = this.objToArr(data.value);
        lastArr.forEach((item) => {
          if (item.name == "pageFoot" && !this.microPage) {
            this.footerConfigData = item;
          }
          if (!item.isHide) {
            temp.push(item);
          }
        });
        temp.sort((a, b) => a.timestamp - b.timestamp);
        this.styleConfig = temp;
      }
    },
    bindSortId(data) {
      this.$emit("bindSortId", data);
    },
    bindHeight(data) {
      this.$emit("bindHeight", data);
    },
    storeTap(id) {
      this.$emit("storeTap", id);
    },
    changeLogin() {
      this.$emit("changeLogin");
    },
    changeBarg(item) {
      this.$emit("changeBarg", item);
    },
    newDataStatus(val, num) {
      this.isFooter = val ? true : false;
      this.pdHeight = num;
      this.$emit("newDataStatus", { val, num });
    },
  },
};
</script>

<style lang="scss" scoped>
.page-design {
  overflow-y: scroll;
  overflow-x: hidden;
  min-height: 100vh;
}
.ysize {
  background-size: 100%;
}
.fullsize {
  background-size: 100% 100%;
}
.noRepeat {
  background-repeat: no-repeat;
}
.repeat {
  background-repeat: repeat;
}
.error-network {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-top: 100px;
  .title {
    font-size: 14px;
    color: #333;
  }
  .btn {
    margin-top: 20px;
    padding: 10px 20px;
    background: #f5f5f5;
    border-radius: 4px;
    font-size: 14px;
    color: #333;
  }
}
</style>
