<template>
	<view :style="colorStyle">
		<view class="OrderCancellation">
			<view class="header"></view>
			<view class="whiteBg">
				<view class="input">
					<input type="number" placeholder="请输入核销码" v-model="verify_code" />
				</view>
				<view class="bnt" @click="codeChange">{{ $t(`立即核销`) }}</view>
			</view>
			<!-- #ifdef MP || MP-WEIXIN || APP-PLUS -->
			<view class="scan" @click="scanCode">
				<image src="https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/scan.jpg"></image>
			</view>
			<!-- #endif -->
			<!-- #ifdef H5  -->
			<view v-if="isWeixin" class="scan" @click="scanCode">
				<image src="https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/scan.jpg"></image>
			</view>
			<!-- #endif -->
		</view>
		<view v-if="iShidden">
			<view class="WriteOff">
				<view class="pictrue">
					<image :src="orderInfo.image" />
				</view>
				<view class="num acea-row row-center-wrapper">
					<text>{{ orderInfo.order_id }}</text>
					<view class="views" @click="goOrderDetails(orderInfo.order_id, orderInfo.order_type)">
						{{ $t(`查看`) }}
						<text class="iconfont icon-jiantou views-jian"></text>
					</view>
				</view>
				<view class="tip">{{ $t(`确定要核销此订单吗`) }}</view>
				<view class="btn sure" @click="confirm">{{ $t(`确定核销`) }}</view>
				<view class="btn cancel" @click="cancel">{{ $t(`取消`) }}</view>
			</view>
			<view class="mask"></view>
		</view>
		<!-- #ifndef MP -->
		<home></home>
		<!-- #endif -->
	</view>
</template>

<script>
import { orderVerific } from '@/api/admin';
import home from '@/components/home';
import colors from '@/mixins/color.js';
import { mapGetters } from 'vuex';
import { toLogin } from '@/libs/login.js';
export default {
	components: {
		home
	},
	mixins: [colors],
	computed: mapGetters(['isLogin']),
	data() {
		return {
			iShidden: false,
			verify_code: '',
			isWeixin: '',
			orderInfo: {}
		};
	},
	onLoad(options) {
		if (!this.isLogin) return toLogin();
		// #ifdef H5 || APP-PLUS
		this.isWeixin = this.$wechat.isWeixin();
		this.verify_code = options.verify_code || '';
		// #endif
		// #ifdef MP
		if (options.scene) {
			let value = this.$util.getUrlParams(decodeURIComponent(options.scene));
			this.verify_code = value.verify_code || '';
		}
		// #endif
	},
	methods: {
		/**
		 * 去订单详情
		 */
		goOrderDetails: function (id, type) {
			if (type == 'integral') {
				uni.navigateTo({
					url: '/pages/points_mall/integral_order_details?order_id=' + id
				});
			} else {
				uni.navigateTo({
					url: '/pages/goods/admin_order_detail/index?id=' + id + '&goname=look'
				});
			}
		},
		// 立即核销
		codeChange: function () {
			let self = this;
			let ref = /[0-9]{12}/;
			if (!this.verify_code)
				return self.$util.Tips({
					title: this.$t(`请输入核销码`)
				});
			if (!ref.test(this.verify_code))
				return self.$util.Tips({
					title: this.$t(`请输入正确的核销码`)
				});
			self.$util.Tips({
				title: this.$t(`查询中`)
			});
			setTimeout(() => {
				orderVerific(this.verify_code, 0)
					.then((res) => {
						self.orderInfo = res.data;
						self.iShidden = true;
					})
					.catch((res) => {
						self.verify_code = '';
						return self.$util.Tips({
							title: res
						});
					});
			}, 800);
		},
		// 扫码核
		scanCode() {
			var self = this;
			// #ifdef MP || APP-PLUS
			uni.scanCode({
				success(res) {
					let path = decodeURIComponent(res.path || res.result);
					self.verify_code = path.split('code=')[1];
					self.codeChange();
				},
				fail(res) {}
			});
			// #endif
			//#ifdef H5
			this.$wechat
				.wechatEvevt('scanQRCode', {
					needResult: 1,
					scanType: ['qrCode', 'barCode']
				})
				.then((res) => {
					if (res.scanType == 'WX_CODE') {
						self.verify_code = res.path.split('%3D')[1];
					} else {
						self.verify_code = res.resultStr.split('=')[1];
					}
					this.codeChange();
				});
			//#endif
		},

		/**
		 * 确定销码
		 */
		confirm: function () {
			let self = this;
			orderVerific(this.verify_code, 1)
				.then((res) => {
					self.verify_code = '';
					self.iShidden = false;
					self.$util.Tips({
						title: res.msg
					});
				})
				.catch((res) => {
					self.$util.Tips({
						title: res
					});
				});
		},
		/**
		 * 取消
		 */
		cancel: function () {
			this.iShidden = false;
		}
	}
};
</script>

<style lang="scss">
page {
	background-color: #fff;
}

.OrderCancellation .header {
	background-image: url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/pages_admin_order_cancellation_index_0.jpg');
	width: 100%;
	height: 300rpx;
	background-repeat: no-repeat;
	background-size: 100% 100%;
	background-color: var(--view-theme);
}

.OrderCancellation {
	width: 100%;
	height: 100%;
	background: #fff;
}

.OrderCancellation .whiteBg {
	width: 90%;
	background-color: #fff;
	margin: -93rpx auto;
	padding-top: 80rpx;
	border-radius: 6rpx 0;
}

.OrderCancellation .whiteBg .input {
	width: 90%;
	margin: 0 auto;
	border-bottom: 1rpx solid #eee;
}

.OrderCancellation .whiteBg .input input {
	font-size: 60rpx;
	color: #282828;
	width: 100%;
	text-align: center;
	line-height: 80rpx;
	height: 80rpx;
}

.OrderCancellation .whiteBg .bnt {
	font-size: 32rpx;
	color: #fff;
	width: 80%;
	height: 86rpx;
	border-radius: 43rpx;
	background-image: linear-gradient(to right, #f67a38 0%, #f11b09 100%);
	text-align: center;
	line-height: 86rpx;
	margin: 55rpx auto 0 auto;
	// background: var(--view-theme);
}

.OrderCancellation .scan {
	width: 300rpx;
	height: 300rpx;
	margin: 160rpx auto 0 auto;
}

.OrderCancellation .scan image {
	width: 100%;
	height: 100%;
	display: block;
}

.WriteOff {
	width: 560rpx;
	height: 860rpx;
	background-color: #fff;
	border-radius: 20rpx;
	position: fixed;
	top: 50%;
	left: 50%;
	margin-top: -400rpx;
	margin-left: -280rpx;
	z-index: 99;
	padding-top: 55rpx;
}

.WriteOff .pictrue {
	width: 340rpx;
	height: 340rpx;
	margin: 0 auto;
}

.WriteOff .pictrue image {
	width: 100%;
	height: 100%;
	display: block;
	border-radius: 10rpx;
}

.WriteOff .num {
	font-size: 30rpx;
	color: #666;
	margin: 28rpx 0 30rpx 0;
}

.WriteOff .num .see {
	font-size: 16rpx;
	color: #fff;
	border-radius: 4rpx;
	background-color: #c68937;
	padding-left: 5rpx;
	margin-left: 12rpx;
}

.WriteOff .num .see .iconfont {
	font-size: 15rpx;
}

.WriteOff .tip {
	font-size: 36rpx;
	color: #282828;
	text-align: center;
	border-top: 1px dashed #ccc;
	padding-top: 40rpx;
	position: relative;
}

.WriteOff .tip:after {
	content: '';
	position: absolute;
	width: 25rpx;
	height: 25rpx;
	border-radius: 50%;
	background-color: #7f7f7f;
	right: -12.5rpx;
	top: -12.5rpx;
}

.WriteOff .tip:before {
	content: '';
	position: absolute;
	width: 25rpx;
	height: 25rpx;
	border-radius: 50%;
	background-color: #7f7f7f;
	left: -12.5rpx;
	top: -12.5rpx;
}

.WriteOff .btn {
	font-size: 32rpx;
	text-align: center;
	line-height: 82rpx;
	height: 82rpx;
	width: 460rpx;
	border-radius: 41rpx;
	margin: 40rpx auto 0rpx auto;
	color: #fff;
}
.sure {
	background-image: linear-gradient(to right, #f11b09 0%, #f67a38 100%);
	background-image: -webkit-linear-gradient(to right, #f11b09 0%, #f67a38 100%);
	background-image: -moz-linear-gradient(to right, #f11b09 0%, #f67a38 100%);
	background-color: var(--view-theme);
}
.WriteOff .cancel {
	// background-image: none;
	// color: #999;
	margin-top: 10rpx;
	border: 1px solid #f11b09;
	color: #f11b09;
	margin-top: 20rpx;
}

.views {
	font-size: 18rpx;
	background: #c68937;
	border-radius: 4px;
	color: #fff;
	padding: 5rpx 2rpx 5rpx 8rpx;
	margin-left: 10rpx;
}

.views-jian {
	font-size: 10px;
}
</style>
