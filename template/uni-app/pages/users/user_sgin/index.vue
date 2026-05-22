<template>
	<view :style="colorStyle">
		<view class="sign" :style="colorStyle">
			<view class="header bgcolor" :style="'background-image: url(' + sginBg + ');'">
				<view class="headerCon acea-row row-between-wrapper">
					<view class="left acea-row row-between-wrapper">
						<img :src="sginTip" alt="" srcset="" />
					</view>
					<navigator class="right acea-row row-middle" hover-class="none" url="/pages/users/user_sgin_list/index">
						<view>{{ $t(`明细`) }}</view>
					</navigator>
				</view>
			</view>
			<view class="wrapper">
				<view class="sgin-num">
					<view class="text">
						<text>已连续签到</text>
						<text class="num">{{ continuousSignDays }}</text>
						<text>天</text>
					</view>
					<view class="tip" v-if="signRemindSwitch == 1">
						<text class="mr16">签到提醒</text>
						<switch :checked="remindStatus" color="#FFCC33" @change="changeRemind" />
					</view>
				</view>
				<view class="list acea-row row-between-wrapper">
					<template v-if="signMode == 0 || signMode == -1">
						<view class="mot-item" v-for="(item, index) in signSystemList" :key="index">
							<view class="row" :class="{ 'sgin-day': e.sign_day, 'last-day': e.is_sign }" v-for="(e, i) in item"
								:key="i">
								<view class="type-img">
									<img v-if="!e.is_sign" :src="getTypeImg(e.type, e.is_sign)" alt="" srcset="" />
									<text v-else class="iconfont icon-xuanzhong1"></text>
								</view>
								<view class="venus">{{ e.day }}</view>
							</view>
						</view>
					</template>
					<template v-else>
						<view class="mot-item" v-for="(item, index) in signSystemList" :key="index">
							<view class="mot-item-box" :class="{ 'sgin-day': e.sign_day, 'last-day': e.is_sign }"
								v-for="(e, i) in item" :key="i">
								<view class="row">
									<view class="num">+{{ e.point }}</view>
									<view class="type-img">
										<img v-if="!e.is_sign" :src="getTypeImg(e.type, e.is_sign)" alt="" srcset="" />
										<text v-else class="iconfont icon-xuanzhong1"></text>
									</view>
								</view>
								<view class="text">{{weekArr[i]}}</view>
							</view>
						</view>
					</template>
				</view>
				<button class="but bg-color on" v-if="checkSign">{{ $t(`今日已签到，明日再来吧`) }}</button>
				<form @submit="goSign" v-else>
					<button class="but bg-color" formType="submit">{{ $t(`立即签到`) }}</button>
				</form>
				<view class="tip" v-if="nextContinuousDays > 0">
					<img :src="`${imgHost}/statics/images/sgin_icon_4.png`" alt="" />
					再连续签到{{ nextContinuousDays }}天，可额外获得惊喜礼包
				</view>
				<view class="lock"></view>
			</view>
			<view class="wrapper wrapper2">
				<view class="tip">{{ $t(`已累计签到`) }}</view>
				<view class="list2 acea-row row-center row-bottom">
					<view class="item">{{ signCount[0] || 0 }}</view>
					<view class="item">{{ signCount[1] || 0 }}</view>
					<view class="item">{{ signCount[2] || 0 }}</view>
					<view class="item">{{ signCount[3] || 0 }}</view>
					<view class="data">{{ $t(`天`) }}</view>
				</view>
				<view class="tip2" v-if="nextCumulativeDays > 0">
					<img :src="`${imgHost}/statics/images/sgin_icon_4.png`" alt="" />
					{{ $t(`再累计签到`) }}{{ nextCumulativeDays }}{{ $t(`天，可额外获得惊喜礼包`) }}
				</view>
				<view class="list3" v-if="signList.length">
					<view class="item acea-row" v-for="(item, index) in signList" :key="index">
						<view>
							<view class="name line1">{{ $t(item.title) }}</view>
							<view class="data">{{ item.add_time }}</view>
						</view>
						<view class="num">+{{ item.number }}</view>
					</view>
					<view class="loading" @click="goSignList" v-if="signList.length >= 8">
						{{ $t(`点击加载更多`) }}
						<text class="iconfont icon-xiangyou"></text>
					</view>
				</view>
			</view>
			<view class="signTip acea-row row-center-wrapper" :class="active == true ? 'on' : ''">
				<view class="signTipLight loadingpic"></view>
				<view class="signTipCon">
					<view class="signHeight">
						<image src="../static/signH.png"></image>
					</view>
					<view class="state">{{ $t(`签到成功`) }}</view>
					<view class="integral">{{ $t(`获得`) }}{{ integral }}{{ $t(`积分`) }}</view>
					<view class="signTipBnt" @click="close">{{ $t(`好的`) }}</view>
				</view>
			</view>
			<view class="mask" @touchmove.stop.prevent="false" :hidden="active == false"></view>
		</view>
		<!-- #ifdef MP -->
		<!-- <authorize @onLoadFun="onLoadFun" :isAuto="isAuto" :isShowAuth="isShowAuth" @authColse="authColse"></authorize> -->
		<!-- #endif -->
	</view>
</template>

<script>
	import {
		toLogin
	} from '@/libs/login.js';
	import {
		mapGetters
	} from 'vuex';
	import {
		postSignUser,
		getSignConfig,
		getSignList,
		setSignIntegral,
		changeRemindStatus
	} from '@/api/user.js';
	import {
		setFormId,
		colorChange
	} from '@/api/api.js';
	import colors from '@/mixins/color';
	// #ifdef MP
	import authorize from '@/components/Authorize';
	// #endif
	import {
		HTTP_REQUEST_URL
	} from '@/config/app';
	export default {
		components: {
			// #ifdef MP
			authorize
			// #endif
		},
		mixins: [colors],
		data() {
			return {
				active: false,
				userInfo: {},
				signCount: [],
				signSystemList: [],
				signList: [],
				integral: 0,
				isAuto: false, //没有授权的不会自动授权
				isShowAuth: false, //是否隐藏授权
				sign_index: 0,
				picUrl: [],
				imgHost: HTTP_REQUEST_URL,
				sginBg: '',
				sginTip: '',
				signMode: 0, // 0月签到 1周签到
				nextContinuousDays: 0,
				nextCumulativeDays: 0,
				continuousSignDays: 0,
				signRemindSwitch: 0,
				checkSign: 0,
				remindStatus: false,
				weekArr: ['周一', '周二', '周三', '周四', '周五', '周六', '周日']
			};
		},
		computed: mapGetters(['isLogin']),
		watch: {
			isLogin: {
				handler: function(newV, oldV) {
					if (newV) {
						this.getUserInfo();
						this.getSignSysteam();
						this.getSignList();
					}
				},
				deep: true
			}
		},
		onLoad() {
			if (this.isLogin) {
				this.getColor();
				this.getUserInfo();
				this.getSignSysteam();
				this.getSignList();
			} else {
				toLogin();
			}
		},
		methods: {
			/**
			 * 授权回调
			 */
			onLoadFun: function() {
				this.getUserInfo();
				this.getSignSysteam();
				this.getSignList();
			},
			// 授权关闭
			authColse: function(e) {
				this.isShowAuth = e;
			},
			getColor() {
				colorChange('color_change').then((res) => {
					this.sginBg = `${this.imgHost}/statics/images/sgin_bg_${res.data.status}.png`;
					this.sginTip = `${this.imgHost}/statics/images/sgin_tip_${res.data.status}.png`;
					let theme = ['#1db0fc', '#42CA4D', '#e93323', '#ff448f', '#FE5C2D'];
					uni.setNavigationBarColor({
						frontColor: '#ffffff', // 必写项
						backgroundColor: theme[res.data.status - 1] // 必写项
					});
				});
			},
			/**
			 * 获取签到配置
			 */
			getSignSysteam: function() {
				let that = this;
				getSignConfig().then((res) => {
					if (!res.data.signStatus) {
						return that.$util.Tips({
							title: that.$t(`签到功能已关闭`)
						}, {
							tab: 3
						});
					}
					that.$set(that, 'signSystemList', res.data.signList);
					that.signMode = res.data.signMode;
					that.nextContinuousDays = res.data.nextContinuousDays;
					that.nextCumulativeDays = res.data.nextCumulativeDays;
					that.continuousSignDays = res.data.continuousSignDays;
					that.signRemindSwitch = res.data.signRemindSwitch;
					that.checkSign = res.data.checkSign;
					that.remindStatus = !!res.data.signRemindStatus;
					that.signCount = that.PrefixInteger(res.data.cumulativeSignDays, 4);
				});
			},
			changeRemind(e) {
				let status = e.detail.value ? 1 : 0;
				changeRemindStatus(status).then((res) => {
					console.log(res);
				});
			},
			getTypeImg(type, isSgin) {
				let src;
				if (isSgin) {
					src = `${this.imgHost}/statics/images/sgin_suc_1.png`;
					return src;
				}
				switch (type) {
					case 1:
						src = `${this.imgHost}/statics/images/sgin_icon_1.png`;
						break;
					case 2:
						src = `${this.imgHost}/statics/images/sgin_icon_2.png`;
						break;
					case 3:
						src = `${this.imgHost}/statics/images/sgin_icon_3.png`;
						break;
					case 4:
						src = `${this.imgHost}/statics/images/sgin_icon_3.png`;
						break;
				}
				return src;
			},
			/**
			 * 去签到记录页面
			 *
			 */
			goSignList: function() {
				return this.$util.Tips('/pages/users/user_sgin_list/index');
			},
			/**
			 * 获取用户信息
			 */
			getUserInfo: function() {
				let that = this;
				postSignUser({
					sign: 1
				}).then((res) => {
					// res.data.integral = parseInt(res.data.integral);
					// let sum_sgin_day = res.data.sum_sgin_day;
					// that.$set(that, 'userInfo', res.data);
					// // that.signCount = that.PrefixInteger(sum_sgin_day, 4);
					// that.sign_index = res.data.sign_num;
				});
			},

			/**
			 * 获取签到列表
			 *
			 */
			getSignList: function() {
				let that = this;
				getSignList({
					page: 1,
					limit: 8
				}).then((res) => {
					that.$set(that, 'signList', res.data);
				});
			},
			/**
			 * 数字转中文
			 *
			 */
			Rp: function(n) {
				let cnum = ['零', '一', '二', '三', '四', '五', '六', '七', '八', '九'];
				let s = '';
				n = '' + n; // 数字转为字符串
				for (let i = 0; i < n.length; i++) {
					s += cnum[parseInt(n.charAt(i))];
				}
				return s;
			},
			/**
			 * 数字分割为数组
			 * @param int num 需要分割的数字
			 * @param int length 需要分割为n位数组
			 */
			PrefixInteger: function(num, length) {
				return (Array(length).join('0') + num).slice(-length).split('');
			},

			/**
			 * 用户签到
			 */
			goSign: function(e) {
				let that = this,
					sum_sgin_day = that.userInfo.sum_sgin_day;
				if (that.userInfo.is_day_sgin)
					return this.$util.Tips({
						title: that.$t(`您今日已签到!`)
					});
				setSignIntegral()
					.then((res) => {
						that.active = true;
						that.integral = res.data.integral;
						// that.sign_index = (that.sign_index + 1) > that.signSystemList.length ? 1 : that
						// 	.sign_index + 1;
						// that.signCount = that.PrefixInteger(sum_sgin_day + 1, 4);
						// that.$set(that.userInfo, 'is_day_sgin', true);
						// that.$set(that.userInfo, 'integral', that.$util.$h.Add(that.userInfo.integral, res.data
						// 	.integral));
						that.getSignSysteam();
						that.getSignList();
					})
					.catch((err) => {
						return this.$util.Tips({
							title: err
						});
					});
			},
			/**
			 * 关闭签到提示
			 */
			close: function() {
				this.active = false;
			}
		}
	};
</script>

<style scoped lang="scss">
	@import "../static/fonts/font.scss";
	.bgcolor {
		background-repeat: no-repeat;
		background-size: 100% 100%;
		width: 750rpx;
		height: 438rpx;
	}

	.sign {
		padding-bottom: 40rpx;
	}

	.sign .header {
		width: 100%;
	}

	.sign .header .headerCon {
		padding: 0 0 0 30rpx;
		height: 234rpx;
	}

	.sign .header .headerCon .left {
		width: 530rpx;
		font-size: 32rpx;
		color: #fff;
		font-weight: bold;

		img {
			width: 388rpx;
			height: 128rpx;
			margin-top: 56rpx;
		}
	}

	.sign .header .headerCon .left .integral text {
		font-size: 24rpx;
		margin-top: 19rpx;
		background-color: #ff9000;
		text-align: center;
		border-radius: 6rpx;
		font-weight: normal;
		padding: 4rpx 15rpx;
	}

	.sign .header .headerCon .text {
		width: 410rpx;
	}

	.sign .header .headerCon .left .pictrue {
		width: 100rpx;
		height: 100rpx;
		border-radius: 50%;
		border: 4rpx solid #ecddbc;
	}

	.sign .header .headerCon .left .pictrue image {
		width: 100%;
		height: 100%;
		border-radius: 50%;
	}

	.sign .header .headerCon .right {
		// width: 142rpx;
		padding: 6rpx 12rpx 6rpx 16rpx;
		border-radius: 50rpx 0 0 50rpx;
		font-size: 24rpx;
		color: #fff;
		background: rgba(255, 255, 255, 0.2);
	}

	.sign .header .headerCon .right .iconfont {
		font-size: 33rpx;
		padding: 0 10rpx 0 30rpx;
		margin-top: 5rpx;
	}

	.sign .wrapper {
		background-color: #fff;
		margin: -188rpx 20rpx 0 20rpx;
		border-radius: 15rpx;
		padding: 32rpx 32rpx 58rpx 32rpx;
		position: relative;

		.tip {
			display: flex;
			justify-content: center;
			align-items: center;
			font-size: 24rpx;
			font-weight: 400;
			color: #999999;
			line-height: 34rpx;
			margin-top: 20rpx;

			img {
				width: 26rpx;
				height: 26rpx;
				margin-right: 10rpx;
				margin-bottom: 1rpx;
			}
		}

		.sgin-num {
			display: flex;
			justify-content: space-between;
			align-items: center;
			font-size: 32rpx;
			font-weight: 500;
			color: #333333;
			line-height: 44rpx;
			margin-bottom: 38rpx;

			.num {
				font-size: 40rpx;
				font-family: D-DIN-PRO-SemiBold, D-DIN-PRO;
				font-weight: 600;
				color: #fe5c2d;
				line-height: 40rpx;
				margin: 0 4rpx;
			}

			.tip {
				display: flex;
				align-items: center;
				font-size: 24rpx;
				font-family: PingFang SC-Regular, PingFang SC;
				font-weight: 400;
				color: #333333;
				line-height: 34rpx;

				.mr16 {
					margin-right: 16rpx;
				}
			}
		}

		::v-deep uni-switch .uni-switch-input:after {
			width: 32rpx;
			height: 32rpx;
			background-color: #fff;
			margin: 4rpx 0 4rpx 4rpx;
			box-shadow: 0 1px 3px rgba(0, 0, 0, 0.4);
		}

		::v-deep uni-switch .uni-switch-input:before {
			width: 72rpx;
			height: 40rpx;
			background-color: #eeeeee;
		}

		::v-deep uni-switch .uni-switch-input {
			width: 76rpx;
			height: 44rpx;
			background-color: #eeeeee;
		}

		::v-deep uni-switch .uni-switch-input.uni-switch-input-checked:after {
			transform: translateX(32rpx);
		}

		::v-deep .uni-switch-input-checked {
			background-color: var(--view-theme) !important;
			border-color: var(--view-theme) !important;
		}

		::v-deep .wx-switch-input {
			width: 80rpx !important;
			height: 44rpx !important;
			background-color: #eeeeee;
		}

		/*白色样式（false的样式）*/
		::v-deep .wx-switch-input::before {
			width: 76rpx !important;
			height: 40rpx !important;
			background-color: #eeeeee;
		}

		/*绿色样式（true的样式）*/
		::v-deep .wx-switch-input::after {
			width: 32rpx !important;
			height: 32rpx !important;
			margin: 4rpx 4rpx 4rpx 4rpx;
		}

		::v-deep .wx-switch-input-checked {
			background-color: var(--view-theme) !important;
			border-color: var(--view-theme) !important;
		}
	}

	.sign .wrapper .list {
		.mot-item {
			display: flex;
			margin-bottom: 20rpx;
			width: 100%;

			// justify-content: space-between;
			.row.sgin-day {
				border: 2rpx solid var(--view-op-point-eight);
			}

			.mot-item-box.sgin-day .row {
				border: 2rpx solid var(--view-op-point-eight);
				color: var(--view-theme);
				background: var(--view-minorColorT);
			}

			.mot-item-box.last-day .row {
				color: var(--view-theme);
				background: var(--view-minorColorT);
			}

			.row.last-day {
				color: var(--view-theme);
				background: var(--view-minorColorT);
			}

			.mot-item-box.last-day .num {
				color: var(--view-theme);
			}

			.row:last-child {
				margin-right: 0;
			}
		}

		.mot-item-box.sgin-day .num {
			color: var(--view-theme);
		}

		.mot-item-box {
			display: flex;
			flex-direction: column;
			justify-content: space-between;
			align-items: center;

			.text {
				font-size: 24rpx;
				font-weight: 400;
				color: #999999;
				line-height: 34rpx;
				padding-right: 14rpx;
				margin-top: 8rpx;
			}

			.num {
				font-size: 26rpx;
				font-weight: 600;
				color: #666666;
				line-height: 24rpx;
				margin-bottom: 16rpx;
			}

			.icon-xuanzhong1 {
				font-size: 40rpx;
				color: var(--view-theme);
			}
		}

		.mot-item-box:last-child {
			.row {
				margin-right: 0;
			}
		}

		.row {
			display: flex;
			flex-direction: column;
			align-items: center;
			justify-content: center;
			height: 110rpx;
			background: #f5f5f5;
			border-radius: 16rpx;
			width: 80rpx;
			margin-right: 14rpx;

			.type-img {
				display: flex;
				align-items: center;
				justify-content: center;
				width: 40rpx;
				height: 40rpx;

				img {
					width: 100%;
					height: 100%;
				}

				.icon-xuanzhong1 {
					font-size: 40rpx;
					color: var(--view-theme);
				}
			}

			.venus {
				font-size: 24rpx;
				font-weight: 400;
				color: #999999;
				line-height: 34rpx;
				margin-top: 14rpx;
			}
		}
	}

	.sign .wrapper .list .item {
		font-size: 22rpx;
		color: #8a8886;
		text-align: center;
	}

	.sign .wrapper .list .item .rewardTxt {
		width: 74rpx;
		height: 32rpx;
		background-color: #f4b409;
		border-radius: 16rpx;
		font-size: 20rpx;
		color: #a57d3f;
		line-height: 32rpx;
	}

	.sign .wrapper .list .item .num {
		font-size: 30rpx;
		color: #999;
	}

	.sign .wrapper .list .item .num.on {
		color: #ff9000;
	}

	.sign .wrapper .list .item .venus {
		margin: 10rpx auto;

		background-repeat: no-repeat;
		background-size: 100% 100%;
		width: 56rpx;
		height: 56rpx;
	}

	.sign .wrapper .list .item .venus.venusSelect {}

	.sign .wrapper .list .item .venus.reward {
		width: 75rpx;
		height: 56rpx;
	}

	.sign .wrapper .but {
		width: 564rpx;
		height: 88rpx;
		font-size: 30rpx;
		line-height: 88rpx;
		color: #fff;
		border-radius: 50rpx;
		text-align: center;
		margin: 30rpx auto 0 auto;
	}

	.sign .wrapper .but.bg-color {
		background-color: var(--view-theme);
	}

	.sign .wrapper .but.on {
		background-color: var(--view-minorColor) !important;
	}

	.sign .wrapper .lock {
		background-image: url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/index_0.png');
		background-repeat: no-repeat;
		background-size: 100% 100%;
		width: 558rpx;
		height: 68rpx;
		position: absolute;
		left: 50%;
		transform: translateX(-50%);
		bottom: -42rpx;
		z-index: 9;
	}

	.sign .wrapper2 {
		margin-top: 15rpx;
		padding: 73rpx 0 56rpx 0;
	}

	.sign .wrapper2 .tip {
		font-size: 30rpx;
		color: #666;
		text-align: center;
	}

	.sign .wrapper2 .list2 {
		margin: 45rpx 0 26rpx 0;
	}

	.sign .wrapper2 .list2 .item {
		border-radius: 10rpx;
		background-color: var(--view-theme);
		width: 80rpx;
		height: 116rpx;
		background-repeat: no-repeat;
		background-size: 100% 100%;
		color: #fff;
		font-size: 72rpx;
		text-align: center;
		line-height: 116rpx;
		margin-right: 19rpx;
		background-image: url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/index_1.png');
	}

	.sign .wrapper2 .list2 .data {
		font-size: 30rpx;
		color: #232323;
	}

	.sign .wrapper2 .tip2 {
		padding: 0 55rpx;
		text-align: center;
		font-size: 24rpx;
		font-weight: 400;
		color: #999999;
		line-height: 34rpx;
		display: flex;
		align-items: center;
		justify-content: center;

		img {
			width: 26rpx;
			height: 26rpx;
			margin-right: 10rpx;
			margin-bottom: 1rpx;
		}
	}

	.sign .list3 {
		margin: 0rpx 37rpx 0 37rpx;
		border-top: 1px dashed #d8d8d8;
	}

	.sign .list3 .item {
		align-items: center;
		justify-content: space-between;
		// border-bottom: 1px solid #eee;
		height: 130rpx;
	}

	.sign .list3 .item .name {
		color: #232323;
		font-size: 30rpx;
		width: 400rpx;
	}

	.sign .list3 .item .data {
		font-size: 24rpx;
		color: #bbbbbb;
		margin-top: 9rpx;
	}

	.sign .list3 .item .num {
		font-size: 36rpx;
		font-family: 'Guildford Pro';
		color: var(--view-theme);
	}

	.sign .list3 .loading {
		font-size: 26rpx;
		color: #282828;
		height: 97rpx;
		line-height: 97rpx;
		text-align: center;
	}

	.sign .list3 .loading .iconfont {
		font-size: 25rpx;
		color: #212121;
		vertical-align: 2rpx;
		margin-left: 10rpx;
	}

	.sign .signTip {
		width: 644rpx;
		height: 645rpx;
		position: fixed;
		top: 50%;
		left: 50%;
		margin-left: -322rpx;
		margin-top: -322.5rpx;
		z-index: 99;
		text-align: center;
		transition: all 0.3s ease-in-out 0s;
		opacity: 0;
		transform: scale(0);
	}

	.sign .signTip .signTipLight {
		background-image: url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/index_2.png');
		background-repeat: no-repeat;
		background-size: 100% 100%;
		width: 100%;
		height: 100%;
	}

	.sign .signTip.on {
		opacity: 1;
		transform: scale(1);
	}

	.sign .signTip .signTipCon {
		background: url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/index_3.png') no-repeat;
		background-size: 100% 100%;
		width: 420rpx;
		height: 410rpx;
		margin-top: -642rpx;
		position: relative;
		background-color: var(--view-theme);
		border-radius: 10rpx;
	}

	.sign .signTip .signTipCon .signHeight {
		width: 134rpx;
		height: 120rpx;
		position: absolute;
		left: 50%;
		margin-left: -67rpx;
		top: -60rpx;
	}

	.sign .signTip .signTipCon .signHeight image {
		width: 100%;
		height: 100%;
	}

	.sign .signTip .signTipCon .state {
		font-size: 34rpx;
		color: #fff;
		margin-top: 150rpx;
	}

	.sign .signTip .signTipCon .integral {
		font-size: 30rpx;
		color: rgba(255, 255, 255, 0.6);
		margin-top: 9rpx;
	}

	.sign .signTip .signTipCon .signTipBnt {
		font-size: 30rpx;
		color: #fff;
		width: 260rpx;
		height: 76rpx;
		background-color: #f8d168;
		border-radius: 38rpx;
		line-height: 76rpx;
		margin: 48rpx auto 0 auto;
	}
</style>