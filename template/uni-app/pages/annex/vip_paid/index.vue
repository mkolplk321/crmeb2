<template>
	<view class="">
		<view class="card-section">
			<view class="acea-row row-middle">
				<image class="image" :src="userInfo.avatar"></image>
				<view class="text">
					<view class="name line1">{{ userInfo.nickname || '' }}</view>
					<view v-if="userInfo.is_ever_level">{{ $t(`永久SVIP会员`) }}</view>
					<view v-else-if="userInfo.is_money_level">
						{{ $t(`SVIP会员`) }} {{ userInfo.overdue_time | dateFormat }}
						{{ $t(`到期`) }}
					</view>
					<view v-else>
						{{ $t(`您与`) }} {{ userInfo.shop_name || '' }} {{ $t(`商城的第`) }} {{ userInfo.register_days || '' }}
						{{ $t(`天.`) }}
					</view>
				</view>
			</view>
			<view class="acea-row row-between-wrapper info">
				<view v-if="userInfo.is_money_level">
					{{ $t(`已累积为您节省`) }} {{ $t(`￥`) }}
					<text class="num">{{ userInfo.economize_money || '' }}</text>
				</view>
				<view v-else>{{ $t(`开通即享会员权益`) }}</view>
				<view class="btn" @click="scrollToCard" v-if="!userInfo.is_ever_level">
					{{ userInfo.is_money_level ? $t(`续费会员`) : $t(`开通会员`) }}
				</view>
			</view>
		</view>
		<view class="right-section" v-if="memberRights.length">
			<view class="section-hd acea-row row-center-wrapper">
				<view class="title acea-row row-center row-bottom">
					<text class="iconfont icon-huiyuan2"></text>
					{{ $t(`SVIP会员尊享权`) }}
				</view>
			</view>
			<view class="section-bd acea-row row-between-wrapper">
				<view v-for="item in memberRights" :key="item.id" class="acea-row row-middle item">
					<image class="image" :src="item.pic"></image>
					<view class="text">
						<view class="name">{{ item.title }}</view>
						<view>{{ item.explain }}</view>
					</view>
				</view>
			</view>
		</view>
		<view class="type-section" id="card" v-if="!userInfo.is_ever_level && memberType.length > 0">
			<view class="title">
				<view class="bold">{{ userInfo.is_money_level ? $t(`续费会员`) : $t(`开通会员`) }}</view>
				<view>
					{{ $t(`有效期至`) }}
					<text class="time">{{ memberEndTime }}</text>
				</view>
			</view>
			<scroll-view class="scroll" scroll-x="true">
				<view v-for="item in memberType" :key="item.type" class="item" :class="{ on: item.mc_id === mc_id }" @click="checkType(item)">
					<view>{{ item.title }}</view>
					<view class="new">
						{{ $t(`￥`) }}
						<text class="num">{{ item.pre_price | moneyFormat }}</text>
					</view>
					<view v-if="item.type === 'free'" class="info">{{ $t(`试用`) }}{{ item.vip_day }}{{ $t(`天`) }}</view>
					<view v-else class="old">{{ $t(`￥`) }}{{ item.price | moneyFormat }}</view>
				</view>
			</scroll-view>
			<view v-if="memberExplain" class="agree">
				<navigator class="link" url="/pages/annex/vip_clause/index" hover-class="none">
					{{ $t(`购买即视为同意`) }}《
					<text class="mark">{{ $t(`会员用户协议`) }}</text>
					》
				</navigator>
			</view>
			<view class="buy" @click="pay">{{ type === 'free' ? $t(`立即试用`) : $t(`立即支付`) }}</view>
			<view class="cash">
				<text @click="drawMemberCard">{{ $t(`点击兑换卡密`) }}</text>
			</view>
		</view>
		<view v-if="memberCoupons.length" class="coupon-section">
			<view class="section-hd acea-row row-between-wrapper">
				<view class="title">{{ $t(`会员优惠券`) }}</view>
				<navigator class="link" url="/pages/annex/vip_coupon/index">
					{{ $t(`查看更多`) }}
					<text class="iconfont icon-xiangyou"></text>
				</navigator>
			</view>
			<scroll-view class="section-bd" scroll-x="true">
				<view v-for="item in memberCoupons" :key="item.id" class="item acea-row row-center-wrapper" :class="{ gray: item.is_fail || item.status === $t(`已使用`) }">
					<view class="money">
						{{ $t(`￥`) }}
						<text class="num">{{ item.coupon_price | moneyFormat }}</text>
					</view>
					<view class="text">
						<view class="name">{{ item.coupon_title }}</view>
						<view v-if="item.use_min_price === '0.00'" class="mark">{{ $t(`无门槛券`) }}</view>
						<view v-else class="mark">{{ $t(`满`) }}{{ item.use_min_price | moneyFormat }}{{ $t(`元可用`) }}</view>
					</view>
					<!-- <view class="btn">{{item.status}}</view> -->
				</view>
			</scroll-view>
		</view>
		<view v-if="goodsList.length" class="goods-section">
			<view class="section-hd">{{ $t(`SVIP商品推荐`) }}</view>
			<view class="section-bd acea-row">
				<view v-for="item in goodsList" :key="item.id" class="item" @click="goDetail(item.id)">
					<image class="image" :src="item.image"></image>
					<view class="name">{{ item.store_name }}</view>
					<view class="svip-price">
						{{ $t(`￥`) }}{{ item.vip_price }}
						<image src="../../..https://goyoto.oss-cn-beijing.aliyuncs.com/images/vip.png"></image>
					</view>
					<view class="shop-price">{{ $t(`商城价`) }}：{{ $t(`￥`) }}{{ item.price }}</view>
				</view>
			</view>
		</view>
		<view :class="{ mask: popupShow }" @touchmove.prevent></view>
		<view class="popup" :class="{ on: popupShow }">
			<view class="cont">
				<view class="title">
					<view class="line"></view>
					<view class="name">{{ $t(`激活会员卡`) }}</view>
					<view class="line"></view>
				</view>
				<form class="form" @submit="checkForm">
					<input v-model="account" class="input" name="account" type="text" :placeholder="$t(`请输入卡号`)" placeholder-style="color:#C49D89" />
					<input v-model="password" class="input" name="password" type="text" :placeholder="$t(`请输入卡密`)" placeholder-style="color:#C49D89" password />
					<button class="button" form-type="submit">{{ $t(`确认激活`) }}</button>
				</form>
			</view>
			<button class="iconfont icon-guanbi2" @click="closePopup"></button>
		</view>
		<payment :payMode="payMode" :pay_close="pay_close" :is-call="true" @onChangeFun="onChangeFun" :order_id="pay_order_id" :totalPrice="totalPrice"></payment>
		<!-- #ifndef MP -->
		<home></home>
		<!-- #endif -->
		<view v-show="false" v-html="formContent"></view>
	</view>
</template>

<script>
import home from '../components/home';
import payment from '../components/payment';
import { mapGetters } from 'vuex';
import { memberCard, memberCardDraw, memberCardCreate, groomList, memberOverdueTime } from '@/api/user.js';
import { toLogin } from '@/libs/login.js';
import { openPaySubscribe } from '@/utils/SubscribeMessage.js';
import dayjs from '@/plugin/dayjs/dayjs.min.js';
import { basicConfig } from '@/api/public.js';
export default {
	components: {
		home,
		payment
	},
	filters: {
		dateFormat: function (value) {
			return dayjs(value * 1000).format('YYYY-MM-DD');
		},
		moneyFormat: function (value) {
			return parseFloat(value);
		}
	},
	data() {
		return {
			memberType: [],
			userInfo: {},
			memberRights: [],
			memberExplain: [],
			memberCoupons: [],
			isGetFree: null,
			popupShow: false,
			account: '',
			password: '',
			goodsList: [],
			pay_order_id: '',
			payMode: [
				{
					name: this.$t(`微信支付`),
					icon: 'icon-weixinzhifu',
					value: 'weixin',
					title: this.$t(`微信支付`),
					payStatus: true
				},
				{
					name: this.$t(`支付宝支付`),
					icon: 'icon-zhifubao',
					value: 'alipay',
					title: this.$t(`支付宝支付`),
					payStatus: true
				},
				{
					name: this.$t(`余额支付`),
					icon: 'icon-qiandai',
					value: 'yue',
					title: this.$t(`余额支付`),
					payStatus: true,
					number: 0
				}
			],
			pay_close: false,
			totalPrice: '0',
			formContent: '',
			page: 1,
			limit: 15,
			finished: false,
			memberEndTime: '',
			// #ifdef H5
			isWeixin: this.$wechat.isWeixin(),
			// #endif
			type: '',
			svip: null,
			mc_id: 0,
			initIn: false
		};
	},
	watch: {
		popupShow: {
			immediate: true,
			handler() {
				this.account = '';
				this.password = '';
			}
		},
		isLogin: {
			deep: true, //深度监听设置为 true
			handler: function (newV, oldV) {
				if (newV) {
					this.memberCard();
					this.groomList();
					this.getOrderPayType();
				}
			}
		}
	},
	computed: mapGetters(['isLogin']),
	onLoad() {
		if (this.isLogin) {
			this.memberCard();
			this.groomList();
			this.getOrderPayType();
		} else {
			toLogin();
		}
	},
	onShow() {
		this.payClose();
		let options = wx.getEnterOptionsSync();
		if (options.scene == '1038' && options.referrerInfo.appId == 'wxef277996acc166c3' && this.initIn) {
			// 代表从收银台小程序返回
			let extraData = options.referrerInfo.extraData;
			this.initIn = false;
			if (!extraData) {
				this.memberCard();
				// "当前通过物理按键返回，未接收到返参，建议自行查询交易结果";
			} else {
				if (extraData.code == 'success') {
					this.memberCard();
				} else if (extraData.code == 'cancel') {
					this.memberCard();
				} else {
					this.memberCard();
					// "支付失败：" + extraData.errmsg;
				}
			}
		}
	},
	onReachBottom() {
		this.groomList();
	},

	methods: {
		pay() {
			if (this.type === 'free') {
				this.createMemberCard('weixin');
			} else {
				this.pay_close = true;
			}
		},
		scrollToCard() {
			const query = uni.createSelectorQuery().in(this);
			query
				.select('#card')
				.boundingClientRect((data) => {
					uni.pageScrollTo({
						scrollTop: data.top
					});
				})
				.exec();
		},
		goDetail(id) {
			uni.navigateTo({
				url: `/pages/goods_details/index?id=${id}`
			});
		},
		// 付费会员数据
		memberCard() {
			uni.showLoading({
				title: this.$t(`正在加载中`)
			});
			memberCard()
				.then((res) => {
					uni.hideLoading();
					const { is_get_free, member_coupons, member_explain, member_rights, member_type } = res.data;
					this.isGetFree = is_get_free;
					this.userInfo = is_get_free.user_info;
					this.memberRights = member_rights;
					this.memberType = member_type;
					this.memberCoupons = member_coupons;
					this.memberExplain = member_explain;
					if (is_get_free.is_record) {
						this.memberType = this.memberType.filter((item) => item.type !== 'free');
					}
					this.totalPrice = this.memberType[0].pre_price;
					this.type = this.memberType[0].type;
					this.svip = this.memberType[0];
					this.mc_id = this.memberType[0].mc_id;
					this.payMode[2].number = is_get_free.user_info.now_money;
					memberOverdueTime({
						member_type: this.svip.type,
						vip_day: this.svip.vip_day
					}).then((res) => {
						this.memberEndTime = res.data.data;
					});
				})
				.catch((err) => {
					uni.showToast({
						title: err,
						icon: 'none'
					});
				});
		},
		groomList() {
			if (this.finished) {
				return;
			}
			uni.showLoading({
				title: this.$t(`正在加载中`)
			});
			groomList(5, {
				page: this.page,
				limit: this.limit
			})
				.then((res) => {
					uni.hideLoading();
					this.goodsList = this.goodsList.concat(res.data.list);
					this.finished = res.data.list.length < this.limit;
					this.page += 1;
				})
				.catch((err) => {
					uni.hideLoading();
				});
		},
		checkType(svip) {
			this.type = svip.type;
			this.svip = svip;
			this.totalPrice = svip.pre_price;
			this.mc_id = svip.mc_id;
			memberOverdueTime({
				member_type: svip.type,
				vip_day: svip.vip_day,
				mc_id: svip.mc_id
			}).then((res) => {
				this.memberEndTime = res.data.data;
			});
		},
		closePopup() {
			this.popupShow = false;
		},
		getMemberCard() {
			memberCard()
				.then((res) => {
					let data = res.data;
					this.memberType = data.member_type;
					this.memberRights = data.member_rights;
					this.isGetFree = data.is_get_free;
				})
				.catch((err) => {});
		},
		getOrderPayType() {
			basicConfig()
				.then((res) => {
					const { ali_pay_status, pay_weixin_open, yue_pay_status } = res.data;
					this.payMode[0].payStatus = pay_weixin_open;
					this.payMode[2].payStatus = yue_pay_status;
					// #ifdef APP-PLUS || H5
					this.payMode[1].payStatus = ali_pay_status;
					// #endif
					//#ifdef MP
					this.payMode[1].payStatus = false;
					//#endif
				})
				.catch((err) => {
					uni.showToast({
						title: err,
						icon: 'none'
					});
				});
		},
		memberSelect(index) {
			this.memberSelected = index;
		},
		// 卡密兑换
		drawMemberCard() {
			let data = {
				member_card_code: '',
				member_card_pwd: '',
				from: 'weixinh5'
			};
			if (!this.popupShow) {
				return (this.popupShow = true);
			}
			if (!this.account) {
				return uni.showToast({
					title: this.$t(`请输入卡号`),
					icon: 'none'
				});
			}
			if (!this.password) {
				return uni.showToast({
					title: this.$t(`请输入卡密`),
					icon: 'none'
				});
			}

			// #ifdef H5
			let ua = navigator.userAgent.toLowerCase();
			if (ua.match(/MicroMessenger/i) == 'micromessenger') {
				data.from = 'weixin';
			}
			// #endif
			// #ifdef MP
			data.from = 'routine';
			// #endif
			uni.showLoading({
				title: this.$t(`正在加载中`)
			});
			// #ifdef MP
			memberCardDraw(data)
				.then((res) => {
					uni.showToast({
						title: res.msg,
						success() {
							this.popupShow = false;
							this.getMemberCard();
						}
					});
				})
				.catch((err) => {
					uni.showToast({
						title: err,
						icon: 'none'
					});
				});
			// #endif
		},
		// 立即购买
		createMemberCard(type) {
			uni.showLoading({
				title: this.$t(`正在加载中`)
			});
			let query = {
				pay_type: type,
				type: '1',
				// #ifdef H5
				from: this.isWeixin ? 'weixin' : 'weixinh5',
				// #endif
				// #ifdef MP
				from: 'routine',
				// #endif
				// #ifdef APP-PLUS
				from: 'weixin',
				// #endif
				member_type: this.svip.type,
				mc_id: this.mc_id,
				price: this.svip.pre_price,
				money: this.svip.price,
				// #ifdef H5
				quitUrl: location.port
					? location.protocol + '//' + location.hostname + ':' + location.port + '/pages/annex/vip_paid/index'
					: location.protocol + '//' + location.hostname + '/pages/annex/vip_paid/index'
				// #endif
			};
			memberCardCreate(query)
				.then((res) => {
					if (res.data.status === 'PAY_DEFICIENCY') {
						return uni.showToast({
							icon:'none',
							title: this.$t(`${res.msg}`)
						});
					}
					if (parseFloat(this.svip.pre_price) > 0) {
						this.callPay(res);
					} else {
						this.memberCard();
						this.groomList();
						uni.hideLoading();
						uni.showToast({
							title: this.$t(`成功开启试用`)
						});
					}
				})
				.catch((err) => {
					uni.showToast({
						title: err,
						icon: 'none'
					});
				});
		},
		formpost(url, postData) {
			let tempform = document.createElement('form');
			tempform.action = url;
			tempform.method = 'post';
			tempform.target = '_self';
			tempform.style.display = 'none';
			for (let x in postData) {
				let opt = document.createElement('input');
				opt.name = x;
				opt.value = postData[x];
				tempform.appendChild(opt);
			}
			document.body.appendChild(tempform);
			this.$nextTick((e) => {
				tempform.submit();
			});
		},
		// 调用支付
		callPay(data) {
			let that = this;
			let { status, result } = data.data,
				{ orderId, jsConfig } = result;
			switch (status) {
				case 'ORDER_EXIST':
				case 'EXTEND_ORDER':
				case 'ALLINPAY_PAY':
					uni.hideLoading();
					// #ifdef MP
					this.initIn = true;
					wx.openEmbeddedMiniProgram({
						appId: 'wxef277996acc166c3',
						extraData: {
							cusid: jsConfig.cusid,
							appid: jsConfig.appid,
							version: jsConfig.version,
							trxamt: jsConfig.trxamt,
							reqsn: jsConfig.reqsn,
							notify_url: jsConfig.notify_url,
							body: jsConfig.body,
							remark: jsConfig.remark,
							validtime: jsConfig.validtime,
							randomstr: jsConfig.randomstr,
							paytype: jsConfig.paytype,
							sign: jsConfig.sign,
							signtype: jsConfig.signtype
						}
					});
					this.jumpData = {
						orderId: data.data.result.orderId,
						msg: data.msg
					};
					// #endif
					// #ifdef APP-PLUS
					plus.runtime.openURL(jsConfig.payinfo);
					setTimeout((e) => {
						uni.reLaunch({
							url: '/pages/annex/vip_paid/index'
						});
					}, 1000);
					// #endif
					// #ifdef H5
					this.formpost(data.data.result.pay_url, jsConfig);
					// #endif
					break;
				case 'PAY_ERROR':
					uni.hideLoading();
					this.$util.Tips(
						{
							title: data.msg
						},
						{
							tab: 5,
							url: '/pages/annex/vip_paid/index'
						}
					);
					break;
				case 'SUCCESS':
					uni.hideLoading();
					this.$util.Tips(
						{
							title: data.msg,
							icon: 'success'
						},
						{
							tab: 5,
							url: '/pages/annex/vip_paid/index'
						}
					);
					break;
				case 'WECHAT_PAY':
					// #ifdef MP
					let mp_pay_name = '';
					if (uni.requestOrderPayment) {
						mp_pay_name = 'requestOrderPayment';
					} else {
						mp_pay_name = 'requestPayment';
					}
					uni[mp_pay_name]({
						timeStamp: jsConfig.timestamp,
						nonceStr: jsConfig.nonceStr,
						package: jsConfig.package,
						signType: jsConfig.signType,
						paySign: jsConfig.paySign,
						success: function (res) {
							that.$util.Tips(
								{
									title: that.$t(`支付成功`),
									icon: 'success'
								},
								{
									tab: 5,
									url: '/pages/annex/vip_paid/index'
								}
							);
						},
						fail: function () {
							uni.showToast({
								title: that.$t(`取消支付`),
								icon: 'none'
							});
						},
						complete: function () {
							uni.hideLoading();
						}
					});
					// #endif
					// #ifdef H5
					this.$wechat
						.pay(result.jsConfig)
						.then((res) => {
							this.$util.Tips(
								{
									title: this.$t(`支付成功`),
									icon: 'success'
								},
								{
									tab: 5,
									url: '/pages/annex/vip_paid/index'
								}
							);
						})
						.catch((err) => {
							if (err.errMsg == 'chooseWXPay:cancel') {
								uni.showToast({
									title: this.$t(`取消支付`),
									icon: 'none'
								});
							}
						});
					// #endif
					// #ifdef APP-PLUS
					uni.requestPayment({
						provider: 'wxpay',
						orderInfo: jsConfig,
						success: (e) => {
							uni.showToast({
								title: this.$t(`支付成功`)
							});
							setTimeout((res) => {
								uni.navigateBack();
							}, 2000);
						},
						fail: (e) => {
							uni.showToast({
								title: this.$t(`支付失败`),
								icon: 'none',
								duration: 2000
							});
						},
						complete: () => {
							uni.hideLoading();
						}
					});
					// #endif
					break;
				case 'WECHAT_H5_PAY':
					uni.showToast({
						title: data.msg,
						success() {
							location.href = jsConfig.h5_url;
						}
					});
					break;
				case 'ALIPAY_PAY':
					uni.hideLoading();
					//#ifdef H5
					this.formContent = jsConfig;
					this.$nextTick(() => {
						document.getElementById('alipaysubmit').submit();
					});
					//#endif
					// #ifdef MP
					uni.navigateTo({
						url: `/pages/users/alipay_invoke/index?id=${data.data.result.order_id}&link=${jsConfig.qrCode}&from=member`
					});
					// #endif
					// #ifdef APP-PLUS
					uni.requestPayment({
						provider: 'alipay',
						orderInfo: jsConfig,
						success: (e) => {
							uni.showToast({
								title: this.$t(`支付成功`)
							});
							setTimeout((res) => {
								uni.navigateBack();
							}, 2000);
						},
						fail: (e) => {
							uni.showModal({
								content: this.$t(`支付失败`),
								showCancel: false,
								success: function (res) {
									if (res.confirm) {
									} else if (res.cancel) {
									}
								}
							});
						},
						complete: () => {
							uni.hideLoading();
						}
					});
					// #endif
					break;
			}
		},
		onChangeFun: function (e) {
			let opt = e;
			let action = opt.action || null;
			let value = opt.value != undefined ? opt.value : null;
			this.pay_close = false;
			action && this[action] && this[action](value);
		},
		payClose: function () {
			this.pay_close = false;
		},
		payCheck: function (type) {
			this.createMemberCard(type);
		},
		// 激活
		checkForm(e) {
			let formData = e.detail.value,
				data = {
					member_card_code: '',
					member_card_pwd: '',
					from: 'H5'
				};
			if (!formData.account) {
				return uni.showToast({
					title: this.$t(`请输入卡号`),
					icon: 'none'
				});
			}
			if (!formData.password) {
				return uni.showToast({
					title: this.$t(`请输入卡密`),
					icon: 'none'
				});
			}
			data.member_card_code = formData.account;
			data.member_card_pwd = formData.password;
			// #ifdef H5
			let ua = navigator.userAgent.toLowerCase();
			if (ua.match(/MicroMessenger/i) == 'micromessenger') {
				data.from = 'weixin';
			}
			// #endif
			// #ifdef MP
			data.from = 'routine';
			// #endif
			uni.showLoading({
				title: this.$t(`激活中`)
			});
			memberCardDraw(data)
				.then((res) => {
					let that = this;
					uni.showToast({
						title: res.msg,
						success() {
							that.memberCard();
							that.groomList();
							that.getOrderPayType();
							that.popupShow = false;
						}
					});
				})
				.catch((err) => {
					uni.showToast({
						title: err,
						icon: 'none'
					});
				});
		}
	}
};
</script>

<style lang="scss" scoped>
.card-section {
	height: 328rpx;
	padding: 88rpx 66rpx 0;
	background: #ffffff
		url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/index_0.png')
		center/100% 100% no-repeat;

	.image {
		width: 80rpx;
		height: 80rpx;
		border-radius: 50%;
	}

	.text {
		flex: 1;
		margin-left: 15rpx;
		font-size: 22rpx;
		line-height: 30rpx;
		color: #89735b;
	}

	.name {
		margin-bottom: 2rpx;
		font-weight: bold;
		font-size: 30rpx;
		line-height: 42rpx;
		color: #333333;
		max-width: 360rpx;
	}

	.info {
		margin-top: 74rpx;
		font-size: 24rpx;
		color: #ae5a2a;
	}

	.num {
		font-size: 36rpx;
	}

	.btn {
		width: 146rpx;
		height: 44rpx;
		border-radius: 22rpx;
		background-color: #ffffff;
		line-height: 44rpx;
		text-align: center;
	}
}

.right-section {
	background-color: #ffffff;

	.section-hd {
		padding-top: 34rpx;
		padding-bottom: 34rpx;

		.title {
			width: 543rpx;
			height: 90rpx;
			background: url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/index_1.png')
				center/cover no-repeat;
			font-weight: bold;
			font-size: 34rpx;
			line-height: 1.1;
			color: #89735b;
		}

		.iconfont {
			font-size: 34rpx;
			color: #89735b;
		}
	}

	.section-bd {
		padding: 0 30rpx 15rpx;

		.item {
			width: 338rpx;
			height: 140rpx;
			padding-left: 30rpx;
			border-radius: 6rpx;
			margin-bottom: 14rpx;
			background-color: #f7f7f7;
		}

		.image {
			width: 80rpx;
			height: 80rpx;
		}

		.text {
			flex: 1;
			min-width: 0;
			margin-left: 22rpx;
			overflow: hidden;
			white-space: nowrap;
			font-size: 24rpx;
			line-height: 33rpx;
			color: #9a856d;
		}

		.name {
			margin-bottom: 2rpx;
			font-size: 26rpx;
			line-height: 37rpx;
			color: #333333;
		}
	}
}

.type-section {
	margin-top: 20rpx;
	background-color: #ffffff;

	.title {
		padding: 26rpx 30rpx 0;
		font-size: 24rpx;
		color: #797979;

		.bold {
			display: inline-block;
			margin-right: 14rpx;
			font-weight: bold;
			font-size: 32rpx;
			line-height: 45rpx;
			color: #333333;

			+ view {
				display: inline-block;
			}
		}

		.time {
			margin-left: 14rpx;
			font-size: 24rpx;
			color: #ae5a2a;
		}
	}

	.scroll {
		white-space: nowrap;
	}

	.item {
		display: inline-flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		width: 210rpx;
		height: 232rpx;
		border-radius: 12rpx;
		margin: 32rpx 30rpx 30rpx 0;
		box-shadow: 0 3rpx 30rpx rgba(0, 0, 0, 0.1);
		font-size: 30rpx;
		line-height: 42rpx;
		color: #754e19;

		&:first-child {
			margin-left: 30rpx;
		}

		&.on {
			border: 3rpx solid #fcc282;
			background-color: #fef7ec;
		}
	}

	.new {
		margin-top: 22rpx;
		font-weight: 600;
		font-size: 34rpx;

		.num {
			font-size: 48rpx;
			line-height: 48rpx;
		}
	}

	.old {
		margin-top: 13rpx;
		text-decoration: line-through;
		font-size: 24rpx;
	}

	.info {
		margin-top: 13rpx;
		font-size: 24rpx;
	}

	.agree {
		font-size: 22rpx;
		text-align: center;
		color: #797979;

		.link {
			display: inline-block;
		}

		.mark {
			color: #ae5a2a;
		}
	}

	.buy {
		height: 80rpx;
		border-radius: 12rpx;
		margin: 30rpx 30rpx 0;
		background: linear-gradient(90deg, #fee2b7 0%, #fdc383 100%);
		font-size: 30rpx;
		line-height: 80rpx;
		text-align: center;
		color: #5d3324;
	}

	.cash {
		padding-top: 26rpx;
		padding-bottom: 29rpx;
		font-size: 28rpx;
		text-align: center;
		color: #754e19;
	}
}

.coupon-section {
	margin-top: 20rpx;
	background-color: #ffffff;

	.section-hd {
		padding: 26rpx 30rpx 0;

		.title {
			font-weight: bold;
			font-size: 32rpx;
			line-height: 45rpx;
			color: #333333;
		}

		.link {
			display: flex;
			align-items: center;
			font-size: 24rpx;
			color: #89735b;

			.iconfont {
				margin-left: 6rpx;
				font-size: 14rpx;
			}
		}
	}

	.section-bd {
		white-space: nowrap;

		.item {
			display: inline-flex;
			width: 360rpx;
			height: 148rpx;
			margin: 26rpx 20rpx 30rpx 0;
			background: url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/index_2.png')
				center/cover no-repeat;

			&:first-child {
				margin-left: 30rpx;
			}

			&:last-child {
				margin-right: 30rpx;
			}

			&.gray {
				background-image: url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/index_3.png');

				.btn {
					color: #9f9f9f;
				}

				.money {
					color: #fff;

					.num {
						color: #fff;
					}
				}

				.name {
					color: #fff;
				}

				.text {
					color: #9f9f9f;
				}

				.mark {
					color: #a8a8a8;
				}
			}

			.btn {
				width: 52rpx;
				padding-right: 14rpx;
				padding-left: 14rpx;
				white-space: pre-wrap;
				font-size: 24rpx;
				color: #5d3324;
			}

			.text {
				max-width: 160rpx;
				margin-left: 14rpx;
			}

			.name {
				overflow: hidden;
				white-space: nowrap;
				text-overflow: ellipsis;
				font-weight: bold;
				font-size: 28rpx;
				line-height: 40rpx;
				color: #5d3324;
			}

			.mark {
				display: inline-block;
				height: 35rpx;
				padding-right: 14rpx;
				padding-left: 14rpx;
				border-radius: 19rpx;
				margin-top: 6rpx;
				background-color: rgba(255, 255, 255, 0.3);
				font-size: 22rpx;
				line-height: 35rpx;
				color: #5f3426;
			}

			.money {
				max-width: 170rpx;
				text-align: center;
				font-size: 30rpx;
				font-weight: 800;
			}

			.num {
				position: relative;
				font-size: 54rpx;
				color: #333333;
			}
		}
	}
}

.goods-section {
	margin-top: 20rpx;
	background-color: #ffffff;

	.section-hd {
		padding: 26rpx 30rpx 0;
		font-weight: bold;
		font-size: 32rpx;
		line-height: 45rpx;
		color: #333333;
	}

	.section-bd {
		padding: 0 30rpx 0 7rpx;
		margin-top: 26rpx;

		.item {
			width: 215rpx;
			padding-bottom: 24rpx;
			margin-left: 23rpx;
		}

		.image {
			width: 215rpx;
			height: 215rpx;
			border-radius: 6rpx;
		}

		.name {
			margin-top: 10rpx;
			overflow: hidden;
			white-space: nowrap;
			text-overflow: ellipsis;
			font-size: 26rpx;
			line-height: 37rpx;
			color: #333333;
		}

		.svip-price {
			margin-top: 6rpx;
			font-size: 26rpx;
			color: #333333;

			image {
				width: 60rpx;
				height: 24rpx;
				margin-left: 6rpx;
			}
		}

		.shop-price {
			margin-top: 4rpx;
			font-size: 20rpx;
			color: #999999;
		}
	}
}

.popup {
	position: fixed;
	top: 50%;
	left: 50%;
	z-index: 10;
	width: 600rpx;
	height: 634rpx;
	padding: 60rpx 40rpx 0;
	margin-top: -79rpx;
	background: url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/index_4.png')
		center/100% 100% no-repeat;
	transform: translate(-50%, -50%) scale(0);
	transition: 0.3s;

	&.on {
		transform: translate(-50%, -50%) scale(1);
	}

	.cont {
		padding: 20rpx;
	}

	.title {
		display: flex;
		justify-content: center;
		align-items: center;
		padding-top: 45rpx;
		padding-bottom: 45rpx;

		.line {
			width: 70rpx;
			height: 2rpx;
			background-color: #e6c3a5;
		}

		.name {
			margin-right: 25rpx;
			margin-left: 25rpx;
			font-size: 40rpx;
			color: #89735b;
		}
	}

	.form {
		.input {
			padding-right: 50rpx;
			padding-left: 50rpx;
			height: 98rpx;
			border-radius: 12rpx;
			margin-bottom: 35rpx;
			background: rgba(255, 187, 129, 0.15);
			font-size: 30rpx;
		}

		.button {
			height: 72rpx;
			border-radius: 36rpx;
			margin: 45rpx 80rpx 20rpx;
			background: #353841;
			font-size: 32rpx;
			line-height: 72rpx;
			color: #ffffff;
		}
	}

	.image {
		position: absolute;
		top: -57rpx;
		left: 50%;
		width: 146rpx;
		height: 114rpx;
		transform: translateX(-50%);
	}

	.iconfont {
		position: absolute;
		bottom: -110rpx;
		left: 50%;
		background: none;
		transform: translateX(-50%);
		font-size: 48rpx;
		line-height: 1;
		color: #ffffff;
	}
}
</style>
