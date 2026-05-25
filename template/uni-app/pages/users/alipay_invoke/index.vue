<template>
	<view>
		<!-- #ifdef H5 -->
		<view v-if="!this.$wechat.isWeixin()">
			<view class="text-section">
				<view>{{ $t(content) }}</view>
			</view>
			<view class="alipaysubmit" v-html="formContent"></view>
		</view>
		<!-- #endif -->
		<view v-else>
			<view class="text-section">
				<view>{{$t(`点击复制网址去浏览器中打开`)}}</view>
				<view class="link">{{ link }}</view>
			</view>
			<view class="button-section">
				<!-- #ifdef H5 -->
				<button class="button copy" :data-clipboard-text="link">{{$t(`点击复制`)}}</button>
				<!-- #endif -->
				<!-- #ifdef MP -->
				<button class="button copy" @click="copyLink">{{$t(`点击复制`)}}</button>
				<!-- #endif -->
				<button class="button off" @click="goDetail">{{$t(`完成支付`)}}</button>
			</view>
		</view>
		<!-- #ifdef H5 -->
		<view v-show="hintShow" class="hint" @click="hintShow = false">
			<view>{{$t(`点击右上角`)}}<text class="iconfont icon-gengduo"></text></view>
			<view>{{$t(`选择 在浏览器 打开，去支付宝支付`)}}</view>
		</view>
		<!-- #endif -->
		<home></home>
	</view>
</template>

<script>
	// #ifdef H5
	import ClipboardJS from '@/plugin/clipboard/clipboard.js';
	import {
		mapGetters
	} from 'vuex';
	import home from '@/components/home/index.vue';
	import {
		aliPay
	} from '@/api/order';
	import {
		toLogin
	} from '@/libs/login.js';
	export default {
		components: {
			home
		},
		data() {
			return {
				// #ifdef H5
				isWeixin: this.$wechat.isWeixin(),
				hintShow: true,
				// #endif
				orderId: '',
				link: '',
				pay_key: '',
				content: this.$t(`正在支付中`),
				formContent: ''
			};
		},
		computed: mapGetters(['isLogin']),
		onLoad(option) {
			if (!this.isLogin && this.$wechat.isWeixin()) {
				toLogin();
			}
			this.orderId = option.id;
			this.link = location.protocol + '//' + window.location.host + '/pages/users/alipay_invoke/index?pay_key=' + option.pay_key;
			this.payKey = option.pay_key;
			if (option.from) this.from = option.from || '';
			if (!this.$wechat.isWeixin()) {
				if (!this.payKey) {
					this.content = this.$t(`支付订单不存在,页面将在2秒后自动关闭`);
					uni.showToast({
						title: this.$t(`支付订单不存在,页面将在2秒后自动关闭`),
						icon: 'none'
					});
					setTimeout(() => {
						uni.redirectTo({
							url: '/pages/index/index'
						});
					}, 2000);
				}
				uni.showLoading({
					title: this.$t(`正在支付中`)
				});
				aliPay(this.payKey, location.protocol + '//' + window.location.host + '/pages/index/index')
					.then(res => {
						uni.hideLoading();
						this.formContent = res.data.pay_content;
						this.$nextTick(() => {
							document.getElementById('alipaysubmit').submit();
						});
					})
					.catch(err => {
						uni.hideLoading();
						uni.showToast({
							title: err,
							icon: 'none'
						});
						setTimeout(() => {
							uni.redirectTo({
								url: '/pages/index/index'
							});
						}, 2000);
					});
			}
		},
		onReady() {
			this.$nextTick(() => {
				// #ifdef H5
				const clipboard = new ClipboardJS(".copy");
				clipboard.on("success", () => {
					uni.showToast({
						title: this.$t(`复制成功`)
					});
				});
				// #endif
			});
		},
		methods: {
			// #ifdef MP
			copyLink() {
				uni.setClipboardData({
					data: this.link,
					success() {
						uni.showToast({
							title: this.$t(`复制成功`),
							icon: 'success'
						});
					},
					fail() {
						uni.showToast({
							title: this.$t(`复制失败`),
							icon: 'none'
						});
					}
				});
			},
			// #endif
			goDetail() {
				uni.navigateTo({
					url: this.from === 'member' ? '/pages/annex/vip_paid/index' : `/pages/goods/order_details/index?order_id=${this.orderId}`
				});
			}
		}
	};
	//#endif
</script>

<style>
	page {
		background-color: #ffffff;
	}
</style>

<style lang="scss" scoped>
	.alipaysubmit {
		display: none;
	}

	.text-section {
		padding-top: 148rpx;
		margin-top: 185rpx;
		background: url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/pages_users_alipay_invoke_index_0.png') center top/280rpx 98rpx no-repeat;
		font-weight: bold;
		font-size: 32rpx;
		text-align: center;
		color: #111111;

		.link {
			margin-top: 16rpx;
			margin-right: 98rpx;
			margin-left: 98rpx;
			white-space: pre-wrap;
			overflow-wrap: break-word;
			font-weight: normal;
			color: #666666;
		}
	}

	.button-section {
		margin-top: 95rpx;
		margin-right: 98rpx;
		margin-left: 98rpx;

		.button {
			height: 80rpx;
			border-radius: 40rpx;
			font-size: 30rpx;
			line-height: 80rpx;

			~.button {
				margin-top: 32rpx;
			}

			&.copy {
				background-color: #333333;
				color: #ffffff;
			}

			&.off {
				border: 2rpx solid #999999;
				color: #666666;
			}
		}
	}

	.hint {
		position: fixed;
		top: 0;
		right: 0;
		bottom: 0;
		left: 0;
		padding-top: 156rpx;
		padding-left: 144rpx;
		background: url("https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/pages_users_alipay_invoke_index_1.png") calc(100% - 32rpx) 12rpx/229rpx 178rpx no-repeat;
		font-weight: 500;
		font-size: 30rpx;
		line-height: 68rpx;
		color: #c1c1c1;
		background-color: rgba(0, 0, 0, 0.8);

		.iconfont {
			margin-left: 10rpx;
			font-weight: bold;
			font-size: 30rpx;
		}
	}
</style>
