<template>
	<view :style="colorStyle">
		<view class='my-promotion'>
			<view class="header">
				<view class='name acea-row row-center-wrapper'>
					<!-- 当前佣金 -->
					<view>
						<view class="user-msg">
							<image class="avatar" :src="userInfo.avatar" mode=""></image>
							<view class="nickname line1">{{userInfo.nickname}}</view>
							<view v-if="userInfo.is_agent_level" class="level line1" @click="jumbPath">
								<text>{{userInfo.agent_level_name?$t(userInfo.agent_level_name):$t(`分销等级`)}}</text>
								<text v-if="userInfo.is_agent_level" class='iconfont icon-xiangyou'></text>
							</view>
						</view>
					</view>
				</view>
				<view class='num'>{{userInfo.brokerage_price}}</view>
				<view class='profit acea-row row-between-wrapper'>
					<view class='item'>
						<view>{{$t(`昨日收益`)}}</view>
						<view class='money'>{{userInfo.yesterDay}}</view>
					</view>
					<view class='item' @click="jumbPath(1)">
						<view>{{$t(`累积已提`)}}<text class='iconfont icon-xiangyou'></text></view>
						<view class='money'>{{userInfo.extractTotalPrice}}</view>
					</view>
				</view>
				<view class="apply"
					v-if="userInfo.division_open && userInfo.agent_apply_open && ((userInfo.is_division && userInfo.division_invite && userInfo.division_status) || (!userInfo.is_division && !userInfo.is_agent))">
					<view v-if="userInfo.is_division">{{$t(`邀请码`)}}：{{userInfo.division_invite}}</view>
					<view v-if="!userInfo.is_division && !userInfo.is_agent">
						<navigator url='/pages/annex/settled/index?type=agent' hover-class="none">
							<view>{{$t(`代理商申请`)}}</view>
						</navigator>
					</view>
				</view>
			</view>
			<!-- #ifdef APP-PLUS || H5 -->
			<navigator url="/pages/users/user_cash/index" hover-class="none" class='bnt bg-color'>{{$t(`立即提现`)}}</navigator>
			<!-- #endif -->
			<!-- #ifdef MP -->
			<view @click="openSubscribe('/pages/users/user_cash/index')" class='bnt bg-color'>{{$t(`立即提现`)}}</view>
			<!-- #endif -->
			<view class='list acea-row row-between-wrapper'>
				<navigator url='/pages/users/user_spread_code/index' hover-class="none"
					class='item acea-row row-center-wrapper row-column'>
					<text class='iconfont icon-erweima'></text>
					<view>{{$t(`推广名片`)}}</view>
				</navigator>
				<navigator url='/pages/users/promoter-list/index' hover-class="none"
					class='item acea-row row-center-wrapper row-column'>
					<text class='iconfont icon-tongji'></text>
					<view>{{$t(`推广人统计`)}}</view>
				</navigator>
				<navigator url='/pages/users/user_spread_money/index?type=2' hover-class="none"
					class='item acea-row row-center-wrapper row-column'>
					<text class='iconfont icon-qiandai'></text>
					<view>{{$t(`佣金明细`)}}</view>
				</navigator>

				<navigator  v-if="(userInfo.division_open && !userInfo.is_agent && !userInfo.is_division) || !userInfo.division_open" url='/pages/users/promoter-order/index' hover-class="none"
					class='item acea-row row-center-wrapper row-column'>
					<text class='iconfont icon-dingdan'></text>
					<view>{{$t(`推广人订单`)}}</view>
				</navigator>
				<navigator v-if="userInfo.division_open && (userInfo.is_agent || userInfo.is_division)" url='/pages/users/promoter-order/index?type=1' hover-class="none"
					class='item acea-row row-center-wrapper row-column'>
					<text class='iconfont icon-dingdan'></text>
					<view>{{userInfo.is_division?$t(`事业部`):$t(`代理商`)}}{{$t(`推广订单`)}}</view>
				</navigator>
				<navigator url='/pages/users/promoter_rank/index' hover-class="none"
					class='item acea-row row-center-wrapper row-column'>
					<text class='iconfont icon-paihang1'></text>
					<view>{{$t(`推广人排行`)}}</view>
				</navigator>
				<navigator url='/pages/users/commission_rank/index' hover-class="none"
					class='item acea-row row-center-wrapper row-column'>
					<text class='iconfont icon-paihang'></text>
					<view>{{$t(`佣金排行`)}}</view>
				</navigator>
				<navigator v-if="userInfo.division_open && userInfo.is_agent" url='/pages/users/staff_list/index' hover-class="none"
					class='item acea-row row-center-wrapper row-column'>
					<text class='iconfont icon-tuandui'></text>
					<view>{{$t(`员工列表`)}}</view>
				</navigator>
				
			</view>
		</view>
		<!-- #ifdef MP -->
		<!-- <authorize @onLoadFun="onLoadFun" :isAuto="isAuto" :isShowAuth="isShowAuth" @authColse="authColse"></authorize> -->
		<!-- #endif -->
		<!-- #ifndef MP -->
		<home></home>
		<!-- #endif -->
	</view>
</template>

<script>
	import {
		getUserInfo
	} from '@/api/user.js';
	import {
		openExtrctSubscribe
	} from '@/utils/SubscribeMessage.js';
	import {
		toLogin
	} from '@/libs/login.js';
	import {
		mapGetters
	} from "vuex";
	// #ifdef MP
	import authorize from '@/components/Authorize';
	// #endif
	import home from '../components/home';
	import colors from '@/mixins/color.js'
	export default {
		components: {
			// #ifdef MP
			authorize,
			// #endif
			home
		},
		mixins: [colors],
		data() {
			return {
				userInfo: {},
				yesterdayPrice: 0.00,
				isAuto: false, //没有授权的不会自动授权
				isShowAuth: false //是否隐藏授权
			};
		},
		computed: mapGetters(['isLogin']),
		watch: {
			isLogin: {
				handler: function(newV, oldV) {
					if (newV) {
						this.getUserInfo();
					}
				},
				deep: true
			}
		},
		onLoad() {
			if (this.isLogin) {
				// try {
				// 	const user_spread_user = uni.getStorageSync('user_spread_user');
				// 	if (user_spread_user) {
				// 		this.getUserInfo();
				// 	} else {
				// 		uni.redirectTo({
				// 			url: '/pages/user/index'
				// 		});
				// 	}
				// } catch (e) {
				// 	uni.showToast({
				// 		title: e,
				// 		icon: 'none'
				// 	});
				// }
				this.getUserInfo()
			} else {
				toLogin();
			}
		},
		methods: {
			onLoadFun: function() {
				this.getUserInfo();
			},
			//跳转
			jumbPath(type) {
				if (type == 1) {
					uni.navigateTo({
						// 提现记录
						url: '/pages/users/user_spread_money/index?type=1'
					})
				} else {
					uni.navigateTo({
						// 分佣等级
						url: '/pages/users/user_distribution_level/index'
					})
				}
			},
			// 授权关闭
			authColse: function(e) {
				this.isShowAuth = e
			},
			openSubscribe: function(page) {
				// uni.showLoading({
				// 	title: this.$t(`正在加载`),
				// })
				openExtrctSubscribe().then(res => {
					uni.hideLoading();
					uni.navigateTo({
						url: page,
					});
				}).catch(() => {
					uni.hideLoading();
				});
			},
			/**
			 * 获取个人用户信息
			 */
			getUserInfo: function() {
				let that = this;
				getUserInfo().then(res => {
					that.$set(that, 'userInfo', res.data);
					if (!res.data.spread_status) {
						that.$util.Tips({
							title: that.$t(`您目前暂无推广权限`)
						}, {
							tab: 1,
							url: '/pages/index/index'
						});
					}
				});
			}
		}
	}
</script>

<style scoped lang="scss">
	.my-promotion .header {
		// background-image: url("https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/pages_users_user_spread_user_index_0.png");
		background-image: url("https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/pages_users_user_spread_user_index_1.png");
		background-repeat: no-repeat;
		background-size: 100% 100%;
		width: 100%;
		// height: 480rpx;
		padding-bottom: 20rpx;
		background-color: var(--view-theme);
	}

	.my-promotion .header .name {
		font-size: 30rpx;
		color: #fff;
		padding-top: 37rpx;
		position: relative;

		.distribution {
			height: 52rpx;
			background-color: #ccc;
			font-size: 24rpx;
			color: #fff;
			position: absolute;
			right: 0;
			border-radius: 30rpx 0 0 30rpx;
			padding: 0 5rpx 0 10rpx;

			&.on {
				background-color: #FFF9E3;
				color: #D16739;
			}

			.iconfont {
				font-size: 18rpx;
			}

			.icon-dengjitubiao {
				font-size: 32rpx;
				margin-right: 10rpx;
			}
		}

		.user-msg {
			display: flex;
			align-items: center;
			justify-content: center;
			flex-direction: column;

			.nickname {
				font-size: 32rpx;
				margin: 10rpx 0;
				max-width: 8rem;
			}

			.level {
				font-size: 18rpx;
				padding: 4rpx 10rpx;
				background: linear-gradient(135deg, var(--view-bntColor) 0%, var(--view-main-over) 100%);
				border-radius: 6rpx;
				transform: scale(0.9);
				display: flex;
				align-items: center;
				max-width: 10rem;
				.icon-xiangyou {
					transform: scale(0.7);
					font-size: 28rpx;
				}
			}

			image {
				width: 100rpx;
				height: 100rpx;
				border-radius: 50%;
			}
		}
	}

	.my-promotion .header .name .record {
		font-size: 26rpx;
		color: rgba(255, 255, 255, 0.8);
		position: absolute;
		right: 20rpx;
	}

	.my-promotion .header .name .record .iconfont {
		font-size: 25rpx;
		margin-left: 10rpx;
		vertical-align: 2rpx;
	}

	.my-promotion .header .num {
		text-align: center;
		color: #fff;
		margin-top: 8rpx;
		font-size: 90rpx;
		font-family: 'Guildford Pro';
	}

	.my-promotion .header .profit {
		padding: 0 20rpx;
		margin-top: 35rpx;
		font-size: 24rpx;
		color: rgba(255, 255, 255, 0.8);
	}

	.my-promotion .header .profit .item {
		min-width: 200rpx;
		text-align: center;
	}

	.my-promotion .header .profit .item .iconfont {
		font-size: 18rpx;
		color: #fff;
		margin-top: 5rpx;
	}

	.my-promotion .header .profit .item .money {
		font-size: 34rpx;
		color: #fff;
		margin-top: 5rpx;
	}

	.my-promotion .bnt {
		font-size: 28rpx;
		color: #fff;
		width: 258rpx;
		border-radius: 50rpx;
		text-align: center;
		line-height: 68rpx;
		margin: -32rpx auto 0 auto;

		border: 16rpx solid #f5f5f5;
	}

	.my-promotion .list {
		padding: 0 20rpx 50rpx 20rpx;
		margin-top: 10rpx;
	}

	.my-promotion .list .item {
		width: 345rpx;
		height: 240rpx;
		border-radius: 20rpx;
		background-color: #fff;
		margin-top: 20rpx;
		font-size: 30rpx;
		color: #666;
	}

	.my-promotion .list .item .iconfont {
		font-size: 70rpx;
		// background-image: linear-gradient(to right, #fc4d3d 0%, #e93323 100%);
		background-color: var(--view-theme);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
		margin-bottom: 20rpx;
	}

	.apply {
		top: 52rpx;
		right: 0;
		position: absolute;
		width: max-content;
		height: 56rpx;
		padding: 0 14rpx;
		background-color: #fff1db;
		color: #a56a15;
		font-size: 22rpx;
		border-radius: 30rpx 0 0 30rpx;
		display: flex;
		align-items: center;
		justify-content: center;
	}
</style>
