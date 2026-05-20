<template>
  <view class="special-page">
    <PageDesign
      :pid="themeId"
      :diyData="currentDiyData"
      :isPreview="isPreview"
      @getVerification="getVerification"
    />
  </view>
</template>

<script>
import PageDesign from "@/components/diyComponents/pageDesign.vue";

export default {
  components: {
    PageDesign,
  },
  data() {
    return {
      themeId: 20, // 固定使用 id=20 的专题
      currentDiyData: {},
      isPreview: false,
    };
  },
  onLoad(options) {
    this.getMicroPage();
  },
  methods: {
    getMicroPage() {
      // 获取专题数据
    },
    getVerification(e) {
      // 验证处理
    },
  },
};
</script>

<style scoped>
.special-page {
  min-height: 100vh;
  background: #f5f5f5;
}
</style>
