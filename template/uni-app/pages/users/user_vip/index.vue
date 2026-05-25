<template>
	<view>
		<view class="member-center">
			<view class="header">
				<swiper class="swiper" :current="swiperIndex" previous-margin="55rpx" next-margin="55rpx" @change="swiperChange">
					<block v-for="(item, index) in VipList" :key="index">
						<swiper-item>
							<view class="swiper-item" :class="{ on: swiperIndex == index }" :style="{ 'background-image': 'url(' + item.image + ')' }">
								<view class="acea-row row-middle user-wrap">
									<image class="image" :src="userInfo.avatar"></image>
									<view class="text">
										<view class="name">{{ $t(userInfo.nickname || '') }}</view>
										<view>
											{{ $t(`商城购物可享`) }}
											<text class="num">{{ item.discount }}</text>
											{{ $t(`折`) }}
										</view>
									</view>
									<view v-if="item.grade === levelInfo.grade" class="state">{{ $t(`当前等级`) }}</view>
									<view v-if="!levelInfo.grade || item.grade > levelInfo.grade" class="state">
										{{ $t(`未达成`) }}
									</view>
								</view>
								<template v-if="item.grade === levelInfo.grade">
									<view class="grow-wrap">
										<view>
											{{ $t(`今日成长值`) }}
											<text class="num">{{ levelInfo.today_exp || '' }}</text>
											{{ $t(`点`) }}
										</view>
										<view class="process">
											<view :style="{ width: `${Math.floor(levelInfo.exp / item.next_exp_num > 1 ? 100 : (levelInfo.exp / item.next_exp_num) * 100)}%` }" class="fill"></view>
										</view>
										<view class="ratio">
											<text class="num">{{ levelInfo.exp || '' }}</text>
											/
											<text class="num">{{ item.next_exp_num || '' }}</text>
										</view>
									</view>
									<navigator class="acea-row row-between-wrapper record-wrap" url="/pages/users/user_vip_areer/index" hover-class="none">
										<view>{{ $t(`我的成长值记录`) }}</view>
										<view class="iconfont icon-jiantou"></view>
									</navigator>
								</template>
								<template v-if="!levelInfo.grade || item.grade > levelInfo.grade">
									<view class="grow-wrap">
										<view class="lock">
											<text class="iconfont icon-quanxianguanlisuozi"></text>
											{{ $t(`暂未解锁该等级`) }}
										</view>
										<view class="process">
											<view :style="{ width: `${Math.floor((levelInfo.exp / item.exp_num) * 100)}%` }" class="fill"></view>
										</view>
										<view class="ratio">
											<text class="num">
												{{ $t(`当前`) }}
												<text>{{ levelInfo.exp || 0 }}</text>
												{{ $t(`点，需达到`) }}
												<text>{{ item.exp_num || '' }}</text>
												{{ $t(`点解锁`) }}
											</text>
										</view>
										<navigator class="acea-row row-between-wrapper record-wrap" style="padding-left: 0" url="/pages/users/user_vip_areer/index" hover-class="none">
											<view>{{ $t(`我的成长值记录`) }}</view>
											<view class="iconfont icon-jiantou"></view>
										</navigator>
									</view>
								</template>
								<view class="level">{{ $t(item.name) }}</view>
								<view v-if="item.grade < levelInfo.grade" class="pass">{{ $t(`已解锁更高等级`) }}</view>
								<!-- <view v-if="!levelInfo.grade || item.grade > levelInfo.grade" class='lock'><text class='iconfont icon-quanxianguanlisuozi'></text>暂未解锁该等级</view> -->
							</view>
						</swiper-item>
					</block>
				</swiper>
				<view class="right-section">
					<view class="section-hd acea-row row-between-wrapper">
						<view>{{ $t(`我的成长特权`) }}</view>
						<navigator v-if="is_open_member" class="svip" url="/pages/annex/vip_paid/index">{{ $t(`立即升级`) }}</navigator>
					</view>
					<view class="section-bd acea-row">
						<view class="item">
							<image class="image" src="../static/1.png"></image>
							<view class="">{{ $t(`购物折扣`) }}</view>
						</view>
						<view class="item">
							<image class="image" src="../static/2.png"></image>
							<view class="">{{ $t(`专属徽章`) }}</view>
						</view>
						<view class="item">
							<image class="image" src="../static/3.png"></image>
							<view class="">{{ $t(`经验累积`) }}</view>
						</view>
						<view class="item">
							<image class="image" src="../static/4.png"></image>
							<view class="">{{ $t(`尊享客服`) }}</view>
						</view>
					</view>
				</view>
			</view>
			<view class="skill-section">
				<view class="section-hd">{{ $t(`快速升级技巧`) }}</view>
				<view class="section-bd">
					<view class="item acea-row row-middle">
						<view class="text">
							<view class="title">
								{{ $t(`签到`) }}
								<text class="mark">{{ $t(`可获得`) }}{{ taskInfo.sign || '' }}{{ $t(`点经验`) }}</text>
							</view>
							<view class="info">{{ $t(`每日签到可获得经验值，已签到`) }}{{ taskInfo.sign_count || '' }}{{ $t(`天`) }}</view>
						</view>
						<navigator class="link" url="/pages/users/user_sgin/index" hover-class="none">{{ $t(`去签到`) }}</navigator>
					</view>
					<view class="item acea-row row-middle">
						<view class="text">
							<view class="title">
								{{ $t(`购买商品`) }}
								<text class="mark">+{{ taskInfo.order || '' }}{{ $t(`点经验/元`) }}</text>
							</view>
							<view class="info">{{ $t(`购买商品可获得对应的经验值`) }}</view>
						</view>
						<navigator class="link" url="/pages/goods/goods_cate/goods_cate" hover-class="none">{{ $t(`去购买`) }}</navigator>
					</view>
					<view class="item acea-row row-middle">
						<view class="text">
							<view class="title">
								{{ $t(`邀请好友`) }}
								<text class="mark">+{{ taskInfo.invite || '' }}{{ $t(`点经验/人`) }}</text>
							</view>
							<view class="info">{{ $t(`邀请好友注册商城可获得经验值`) }}</view>
						</view>
						<navigator class="link" url="/pages/users/user_spread_code/index" hover-class="none">
							{{ $t(`去邀请`) }}
						</navigator>
					</view>
				</view>
			</view>
			<recommend v-if="hostProduct.length" :hostProduct="hostProduct"></recommend>
			<view class="growthValue" :class="growthValue == false ? 'on' : ''">
				<view class="pictrue">
					<image src="../static/value.jpg"></image>
					<text class="iconfont icon-guanbi3" @click="growthValue"></text>
				</view>
				<view class="conter">{{ illustrate }}</view>
			</view>
			<view class="mask" :hidden="growthValue" @click="growthValueClose"></view>
		</view>
	</view>
</template>

<script>
import { getUserInfo, getlevelInfo, userLevelGrade, userLevelTask, userLevelDetection } from '@/api/user.js';
import { getProductHot } from '@/api/store.js';
import { toLogin } from '@/libs/login.js';
import { mapGetters } from 'vuex';
// #ifdef MP
import authorize from '@/components/Authorize';
// #endif
import recommend from '../components/recommend';
export default {
	components: {
		recommend,
		// #ifdef MP
		authorize
		// #endif
	},
	data() {
		return {
			reach_count: 0,
			VipList: [],
			// indicatorDots: false,
			// circular: true,
			// autoplay: false,
			// interval: 3000,
			// duration: 500,
			swiperIndex: 0,
			growthValue: true,
			task: [], //任务列表
			illustrate: '', //任务说明
			level_id: 0, //任务id,
			hostProduct: [],
			grade: 0,
			isAuto: false, //没有授权的不会自动授权
			isShowAuth: false, //是否隐藏授权
			hotScroll: false,
			hotPage: 1,
			hotLimit: 10,
			level_title: '',
			level_discount: '',
			levelInfo: {},
			task_list: [
				{
					real_name: this.$t(`积分数`),
					number: 0
				},
				{
					real_name: this.$t(`消费金额`),
					number: 0
				},
				{
					real_name: this.$t(`优惠券`),
					number: 0
				}
			],
			userInfo: {},
			taskInfo: {},
			is_open_member: 0
		};
	},
	computed: mapGetters(['isLogin']),
	watch: {
		VipList: function () {
			let that = this;
			if (that.VipList.length > 0) {
				that.VipList.forEach(function (item, index) {
					if (item.is_clear === false) {
						// that.swiper.slideTo(index);
						that.activeIndex = index;
						that.grade = item.grade;
					}
				});
			}
		},
		isLogin: {
			handler: function (newV, oldV) {
				if (newV) {
					this.setLeveLComplete();
					this.get_host_product();
				}
			},
			deep: true
		}
	},
	onLoad() {
		if (this.isLogin) {
			this.setLeveLComplete();
			this.get_host_product();
			this.getlevelInfo();
			this.getUserInfo();
		} else {
			toLogin();
		}
		let that = this;
		setTimeout(function () {
			that.loading = true;
		}, 500);
	},
	methods: {
		getUserInfo: function () {
			getUserInfo().then((res) => {
				this.is_open_member = res.data.is_open_member;
				if (res.data.member_func_status === 0) {
					this.$util.Tips(
						{
							title: this.$t(`暂未开启等级`)
						},
						{
							tab: 3
						}
					);
				}
				this.task_list = [
					{
						real_name: this.$t(`积分数`),
						number: res.data.integral
					},
					{
						real_name: this.$t(`消费金额`),
						number: res.data.orderStatusSum
					},
					{
						real_name: this.$t(`优惠券`),
						number: res.data.couponCount
					}
				];
			});
		},
		getlevelInfo: function () {
			getlevelInfo().then((res) => {
				const { level_info, level_list, task, user } = res.data;
				this.levelInfo = level_info;
				this.VipList = level_list;
				this.userInfo = user;
				this.taskInfo = task;
				this.levelInfo.exp = parseFloat(this.levelInfo.exp);
				this.levelInfo.rate = Math.floor((this.levelInfo.exp / this.levelInfo.exp_num) * 100);
				if (this.levelInfo.rate > 100) {
					this.levelInfo.rate = 100;
				}
				const index = level_list.findIndex(({ grade }) => grade === level_info.grade);
				if (index !== -1) {
					this.swiperIndex = index;
				}
			});
		},
		onLoadFun: function () {
			this.setLeveLComplete();
			this.get_host_product();
		},
		// 授权关闭
		authColse: function (e) {
			this.isShowAuth = e;
		},
		/**
		 * 获取我的推荐
		 */
		get_host_product: function () {
			let that = this;
			getProductHot().then((res) => {
				let that = this;
				if (that.hotScroll) return;
				getProductHot(that.hotPage, that.hotLimit).then((res) => {
					that.hotPage++;
					that.hotScroll = res.data.length < that.hotLimit;
					that.hostProduct = that.hostProduct.concat(res.data);
				});
			});
		},
		/**
		 * 会员切换
		 *
		 */
		swiperChange(e) {
			let index = e.detail.current;
			this.swiperIndex = index;
			this.level_id = this.VipList[index].id || 0;
			this.level_title = this.VipList[index].name || '';
			this.level_discount = this.VipList[index].discount || '';
			// this.grade = this.VipList[index].grade
			// this.getTask();
		},
		/**
		 * 关闭说明
		 */
		growthValueClose: function () {
			this.growthValue = true;
		},
		/**
		 * 打开说明
		 */
		opHelp: function (index) {
			this.growthValue = false;
			this.illustrate = this.task[index].illustrate;
		},
		/**
		 * 设置会员
		 */
		setLeveLComplete: function () {
			let that = this;
			userLevelDetection().then((res) => {
				// that.getVipList();
			});
		},
		/**
		 * 获取会员等级
		 *
		 */
		getVipList: function () {
			let that = this;
			userLevelGrade().then((res) => {
				that.$set(that, 'VipList', res.data.list);
				that.task = res.data.task.task;
				that.reach_count = res.data.task.reach_count;
				that.level_id = res.data.list[0] ? res.data.list[0].id : 0;
				that.level_title = res.data.list[0] ? res.data.list[0].name : '';
				that.level_discount = res.data.list[0] ? res.data.list[0].discount : '';
				// let arr = [];
				// res.data.list.forEach(function(item, index) {
				// 	if (item.is_clear == false) {
				// 		arr.push(item.grade);
				// 	}
				// })
				// that.grade = arr[0] || 0;
				// that.grade = res.data.list[0].grade
			});
		},
		/**
		 * 获取任务要求
		 */
		getTask: function () {
			let that = this;
			userLevelTask(that.level_id).then((res) => {
				that.task = res.data.task;
				that.reach_count = res.data.reach_count;
			});
		}
	},
	onReachBottom() {
		this.get_host_product();
	},
	// 滚动监听
	onPageScroll(e) {
		// 传入scrollTop值并触发所有easy-loadimage组件下的滚动监听事件
		uni.$emit('scroll');
	}
};
</script>

<style lang="scss" scoped>
.swiper {
	.swiper-item {
		height: 330rpx;
		border-radius: 10rpx;
		background: center/100% 100% no-repeat;
		transform: scale(0.9);
		transition: all 0.2s ease-in 0s;
		line-height: 1.1;

		&.on {
			transform: none;
		}
	}

	.user-wrap {
		padding-top: 20rpx;
		padding-left: 22rpx;
		line-height: 1.1;

		.image {
			width: 90rpx;
			height: 90rpx;
			border: 6rpx solid rgba(245, 245, 245, 0.3);
			border-radius: 50%;
		}

		.text {
			flex: 1;
			min-width: 0;
			margin-right: 14rpx;
			margin-left: 14rpx;
			font-size: 22rpx;
			color: #666666;

			.num {
				margin-right: 10rpx;
				margin-left: 10rpx;
				font-size: 30rpx;
				font-style: italic;
			}
		}

		.name {
			margin-bottom: 9rpx;
			overflow: hidden;
			white-space: nowrap;
			text-overflow: ellipsis;
			font-weight: bold;
			font-size: 30rpx;
			color: #282828;
		}

		.state {
			align-self: flex-start;
			height: 36rpx;
			padding-right: 12rpx;
			padding-left: 18rpx;
			border-top-left-radius: 18rpx;
			border-bottom-left-radius: 18rpx;
			background-color: #484848;
			overflow: hidden;
			font-size: 20rpx;
			line-height: 36rpx;
			color: #ffffff;
		}
	}

	.grow-wrap {
		padding-left: 24rpx;
		margin-top: 37rpx;
		font-size: 22rpx;
		color: #282828;

		.num {
			margin-right: 8rpx;
			margin-left: 8rpx;
			font-size: 32rpx;
		}

		.process {
			width: 300rpx;
			height: 6rpx;
			border-radius: 3rpx;
			margin-top: 22rpx;
			background-color: rgba(0, 0, 0, 0.2);
		}

		.fill {
			width: 50%;
			height: 100%;
			border-radius: 3rpx;
			background-color: #333333;
		}

		.ratio {
			margin-top: 15rpx;
			font-size: 24rpx;

			.num {
				margin-right: 0;
				margin-left: 0;
				font-size: 24rpx;

				text {
					margin-right: 8rpx;
					margin-left: 8rpx;

					&:first-child {
						font-size: 32rpx;
					}
				}
			}
		}
	}

	.record-wrap {
		margin-top: 30rpx;
		padding-right: 22rpx;
		padding-left: 24rpx;
		font-size: 22rpx;
		color: #282828;

		.iconfont {
			font-size: 22rpx;
		}
	}

	.pass {
		display: inline-block;
		width: 292rpx;
		height: 56rpx;
		margin-top: 80rpx;
		margin-left: 42rpx;
		background: url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/index_0.png')
			center/100% 100% no-repeat;
		font-weight: bold;
		font-size: 30rpx;
		line-height: 56rpx;
		text-align: center;
		color: #282828;
	}

	.lock {
		font-weight: bold;
		font-size: 30rpx;
		color: #282828;

		.iconfont {
			margin-right: 17rpx;
			font-size: 30rpx;
			vertical-align: middle;
		}
	}

	.level {
		position: absolute;
		top: 195rpx;
		left: 440rpx;
		width: 132rpx;
		height: 44rpx;
		font-weight: bold;
		font-size: 30rpx;
		line-height: 44rpx;
		text-align: center;
		color: #e1e6ed;
	}
}

.right-section {
	border-radius: 10rpx;
	margin: 28rpx 30rpx 0;
	background-color: #ffffff;
	box-shadow: 0 5rpx 16rpx 0 rgba(235, 235, 235, 0.5);

	.section-hd {
		padding: 25rpx 25rpx 0;
		font-weight: bold;
		font-size: 32rpx;
		color: #282828;
	}

	.svip {
		height: 40rpx;
		padding-right: 37rpx;
		padding-left: 19rpx;
		border-radius: 20rpx;
		background: #333333
			url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/index_1.png')
			calc(100% - 13rpx) center/19rpx 22rpx no-repeat;
		font-size: 24rpx;
		line-height: 40rpx;
		text-align: center;
		color: #ffdeb2;
	}

	.section-bd {
		.item {
			flex: 0 0 25%;
			padding-top: 38rpx;
			padding-bottom: 38rpx;
			font-size: 26rpx;
			line-height: 1;
			text-align: center;
			color: #282828;

			.image {
				width: 90rpx;
				height: 90rpx;
				border-radius: 50%;
				margin: 0 auto 19rpx;
				background: linear-gradient(0deg, #e9bb6d 0%, #f2c880 100%);
				font-size: 48rpx;
				line-height: 90rpx;
			}
		}
	}
}

.skill-section {
	margin-top: 65rpx;
	margin-bottom: 20rpx;

	.section-hd {
		padding-top: 38rpx;
		padding-bottom: 32rpx;
		padding-left: 54rpx;
		font-weight: bold;
		font-size: 32rpx;
		color: #282828;
	}

	.section-bd {
		padding-right: 30rpx;
		padding-left: 30rpx;

		.item {
			height: 140rpx;
			padding-right: 25rpx;
			padding-left: 25rpx;
			border-radius: 4rpx;
			background-color: #ffffff;
			box-shadow: 0 5rpx 10rpx 0 rgba(235, 235, 235, 0.5);

			~ .item {
				margin-top: 20rpx;
			}
		}

		.text {
			flex: 1;
		}

		.title {
			padding-left: 19rpx;
			border-left: 5rpx solid #e8b869;
			font-weight: bold;
			font-size: 30rpx;
			line-height: 1.1;
			color: #282828;

			.mark {
				margin-left: 20rpx;
				font-weight: normal;
				font-size: 24rpx;
				color: #c6985c;
			}
		}

		.info {
			padding-left: 19rpx;
			margin-top: 18rpx;
			font-size: 22rpx;
			color: #999999;
		}

		.link {
			width: 146rpx;
			height: 50rpx;
			border-radius: 25rpx;
			background: linear-gradient(-90deg, #e7b667 0%, #ffeab5 100%);
			font-size: 26rpx;
			line-height: 50rpx;
			text-align: center;
			color: #8a5f1e;
		}
	}
}

.member-center .header {
	background: url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/index_2.jpg')
		center top/100% 100% no-repeat;
	height: 600rpx;
	padding-top: 37rpx;
}

.member-center .header swiper {
	position: relative;
	height: 330rpx;
}

.member-center .header swiper-item .memberBg.active {
	transform: none;
	transition: all 0.2s ease-in 0s;
}

.member-center .header swiper-item .memberBg.quiet {
	transform: scale(0.9);
	transition: all 0.2s ease-in 0s;
}

.member-center .header swiper-item .memberBg {
	width: 100%;
	height: 328rpx;
	border-radius: 16rpx;
	color: #fff;
	position: relative;
	background-size: 100% 100%;
}

.member-center .header swiper-item .memberBg image {
	width: 89rpx;
	height: 108rpx;
	display: block;
	position: absolute;
	right: 60rpx;
}

.member-center .header swiper-item .memberBg .name {
	font-size: 46rpx;
	font-weight: bold;
	padding: 40rpx 0 0 35rpx;
}

.member-center .header swiper-item .memberBg .gather {
	padding-left: 86rpx;
	font-size: 22rpx;
	line-height: 40rpx;
}

.member-center .header swiper-item .memberBg .gather .strong {
	font-size: 30rpx;
}

.member-center .header swiper-item .memberBg .ungather {
	padding-left: 40rpx;
	font-size: 22rpx;
	line-height: 55rpx;
}

.member-center .header swiper-item .memberBg .discount {
	font-size: 28rpx;
	font-weight: bold;
	margin: 15rpx 0 0 35rpx;
}

.member-center .header swiper-item .memberBg .discount .iconfont {
	margin-left: 10rpx;
	font-size: 30rpx;
}

.member-center .header swiper-item .memberBg .nav {
	margin-top: 36rpx;
}

.member-center .header swiper-item .memberBg .nav .item {
	flex: 1;
	text-align: center;
	color: rgba(255, 255, 255, 0.6);
	position: relative;
}

.member-center .header swiper-item .memberBg .nav .item .num {
	font-size: 40rpx;
	color: #fff;
	font-family: 'Guildford Pro';
}

.member-center .header swiper-item .memberBg .nav .item ~ .item::before {
	position: absolute;
	width: 2rpx;
	height: 32rpx;
	background-color: rgba(255, 255, 255, 0.6);
	content: '';
	left: 0;
	top: 50%;
	transform: translateY(-50%);
}

.member-center .header swiper-item .memberBg .lock {
	font-size: 26rpx;
	margin: 73rpx 0 0 35rpx;
}

.member-center .header swiper-item .memberBg .lock .iconfont {
	font-size: 37rpx;
	margin-right: 15rpx;
	vertical-align: -4rpx;
}

.member-center .wrapper {
	background-color: #fff;
	padding-bottom: 16rpx;
	margin-bottom: 20rpx;
}

.member-center .wrapper .title {
	height: 98rpx;
	padding: 0 30rpx;
	font-size: 30rpx;
	font-weight: bold;
	color: #282828;
}

.member-center .wrapper .title .iconfont {
	color: #ffae06;
	font-weight: normal;
	font-size: 40rpx;
	margin-right: 12rpx;
	vertical-align: -2rpx;
}

.member-center .wrapper .title .num {
	font-size: 28rpx;
	color: #999;
}

.member-center .wrapper .title .num .current {
	color: #ffae06;
}

.member-center .wrapper .list .item {
	width: 690rpx;
	height: 184rpx;
	background-color: #f9f9f9;
	margin: 0 auto 20rpx auto;
	padding: 27rpx 0 22rpx 0;
	border-radius: 12rpx;
	box-sizing: border-box;
}

.member-center .wrapper .list .item .top {
	padding-right: 27rpx;
	font-size: 26rpx;
	color: #999;
}

.member-center .wrapper .list .item .top .name {
	border-left: 6rpx solid #ffae06;
	padding-left: 20rpx;
	font-size: 28rpx;
	color: #282828;
	font-weight: bold;
}

.member-center .wrapper .list .item .top .name .iconfont {
	color: #999;
	font-size: 30rpx;
	vertical-align: -2rpx;
	margin-left: 10rpx;
}

.member-center .wrapper .list .item .cu-progress {
	overflow: hidden;
	height: 12rpx;
	background-color: #eee;
	width: 636rpx;
	border-radius: 20rpx;
	margin: 35rpx auto 0 auto;
}

.member-center .wrapper .list .item .cu-progress .bg-red {
	width: 0;
	height: 100%;
	transition: width 0.6s ease;
	background-color: #ffaa29;
	border-radius: 20rpx;
}

.member-center .wrapper .list .item .experience {
	margin-top: 17rpx;
	padding: 0 27rpx;
	font-size: 24rpx;
	color: #999;
}

.member-center .wrapper .list .item .experience .num {
	color: #ffad07;
}

.member-center .growthValue {
	background-color: #fff;
	border-radius: 16rpx;
	position: fixed;
	top: 266rpx;
	left: 50%;
	width: 560rpx;
	height: 740rpx;
	margin-left: -280rpx;
	z-index: 99;
	transform: translate3d(0, -200%, 0);
	transition: all 0.3s cubic-bezier(0.25, 0.5, 0.5, 0.9);
}

.member-center .growthValue.on {
	transform: translate3d(0, 0, 0);
}

.member-center .growthValue .pictrue {
	width: 100%;
	height: 257rpx;
	position: relative;
}

.member-center .growthValue .pictrue image {
	width: 100%;
	height: 100%;
	border-radius: 16rpx 16rpx 0 0;
}

.member-center .growthValue .conter {
	padding: 0 35rpx;
	font-size: 30rpx;
	color: #333;
	margin-top: 58rpx;
	line-height: 1.5;
	height: 350rpx;
	overflow: auto;
}

.member-center .growthValue .pictrue .iconfont {
	position: absolute;
	font-size: 65rpx;
	color: #fff;
	top: 775rpx;
	left: 50%;
	transform: translateX(-50%);
}

.trait {
	margin-top: 36rpx;

	.trait-hd {
		display: flex;
		justify-content: center;
		align-items: center;

		.title {
			padding-right: 50rpx;
			padding-left: 50rpx;
			background: url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/index_3.png')
					left center/36rpx 21rpx no-repeat,
				url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/index_4.png')
					right center/36rpx 21rpx no-repeat;
			font-size: 28rpx;
			color: #ffffff;
		}
	}

	.trait-bd {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin: 20rpx 30rpx 0;

		.item {
			display: flex;
			flex-direction: column;
			justify-content: center;
			align-items: center;
			width: 156rpx;
			height: 173rpx;
			border-radius: 8rpx;
			background-color: rgba(255, 255, 255, 0.06);
			font-size: 22rpx;
			color: #ffffff;
		}

		.image {
			width: 68rpx;
			height: 68rpx;
			margin-bottom: 13rpx;
			background: center/cover no-repeat;

			&.image1 {
				background-image: url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/index_5.png');
			}

			&.image2 {
				background-image: url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/index_6.png');
			}

			&.image3 {
				background-image: url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/index_7.png');
			}

			&.image4 {
				background-image: url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/index_8.png');
			}
		}

		.text {
			color: #fcdb9c;
		}
	}
}

.module {
	padding: 40rpx 30rpx 0 30rpx;
	margin-bottom: 20rpx;
	background-color: #ffffff;

	.icons {
		width: 5rpx;
		height: 30rpx;
		margin-right: 8rpx;
		background-color: #e6c083;
	}

	.link {
		font-size: 24rpx;
		color: #999999;

		.iconfont {
			margin-left: 6rpx;
			font-size: 24rpx;
			color: #999999;
		}
	}

	.gainList {
		margin-top: 10rpx;

		.item {
			height: 130rpx;
			position: relative;

			.picTxt {
				.pictrue {
					width: 70rpx;
					height: 70rpx;
					border-radius: 50%;
					background: url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/index_9.png')
						center/cover no-repeat;
					text-align: center;
					line-height: 70rpx;
					color: #fff;

					&.on {
						background-image: url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/index_10.png');
					}

					&.on2 {
						background-image: url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/index_11.png');
					}
				}

				.text {
					margin-left: 30rpx;
					width: 400rpx;

					.name {
						font-size: 30rpx;
						color: #282828;
					}

					.info {
						font-size: 24rpx;
						color: #999999;
						margin-top: 6rpx;
					}
				}
			}

			.button {
				width: 140rpx;
				height: 50rpx;
				background: #fcdb9c;
				border-radius: 25rpx;
				text-align: center;
				line-height: 50rpx;
				font-size: 26rpx;
				color: #8d5306;
			}

			& ~ .item {
				&::after {
					position: absolute;
					content: ' ';
					width: 720rpx;
					height: 1rpx;
					background: rgba(245, 245, 245, 1);
					top: 0;
					left: 0;
				}
			}
		}
	}
}
</style>
