<template>
	<view :style="colorStyle">
		<view class="PromoterRank">
			<view class="redBg bg-color">
				<view class="header">
					<view class="nav acea-row row-center-wrapper">
						<view class="item" :class="active == index ? 'font-color' : ''" v-for="(item,index) in navList" :key="index"
						 @click="switchTap(index)">
							{{ $t(item) }}
						</view>
					</view>
					<view class="rank acea-row row-bottom row-around">
						<view class="item two" v-if="Two.uid">
							<view class="pictrue">
								<image :src="Two.avatar"></image>
							</view>
							<view class="name line1">{{Two.nickname}}</view>
							<view class="num">{{Two.count}}{{$t(`人`)}}</view>
						</view>
						<view class="item one" v-if="One.uid">
							<view class="pictrue">
								<image :src="One.avatar"></image>
							</view>
							<view class="name line1">{{One.nickname}}</view>
							<view class="num">{{One.count}}{{$t(`人`)}}</view>
						</view>
						<view class="item three" v-if="Three.uid">
							<view class="pictrue">
								<image :src="Three.avatar"></image>
							</view>
							<view class="name line1">{{Three.nickname}}</view>
							<view class="num">{{Three.count}}{{$t(`人`)}}</view>
						</view>
					</view>
				</view>
			</view>
			<view class="list" v-if="rankList.length">
				<view class="item acea-row row-between-wrapper" v-for="(item,index) in rankList" :key="index">
					<view class="num">{{index+4}}</view>
					<view class="picTxt acea-row row-between-wrapper">
						<view class="pictrue">
							<image :src="item.avatar"></image>
						</view>
						<view class="text line1">{{item.nickname}}</view>
					</view>
					<view class="people font-color">{{item.count}}{{$t(`人`)}}</view>
				</view>
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
		getRankList
	} from '@/api/user.js';
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
	import colors from '@/mixins/color.js';
	export default {
		components: {
			// #ifdef MP
			authorize,
			// #endif
			home
		},
		mixins:[colors],
		data() {
			return {
				navList: [this.$t(`周榜`), this.$t(`月榜`)],
				active: 0,
				page: 1,
				limit: 10,
				type: 'week',
				loading: false,
				loadend: false,
				rankList: [],
				Two: {},
				One: {},
				Three: {},
				isAuto: false, //没有授权的不会自动授权
				isShowAuth: false //是否隐藏授权
			};
		},
		computed: mapGetters(['isLogin']),
		watch:{
			isLogin:{
				handler:function(newV,oldV){
					if(newV){
						this.getRanklist();
					}
				},
				deep:true
			}
		},
		onLoad() {
			if (this.isLogin) {
				this.getRanklist();
			} else {
				toLogin();
			}
		},
		// onShow: function () {
		//    if(this.isClone && app.globalData.isLog){
		//      this.setData({ loadend: false, page: 1, rankList:[]});
		//      this.getRanklist();
		//    }
		//  },
		methods: {
			onLoadFun() {
				this.getRanklist();
			},
			// 授权关闭
			authColse: function(e) {
				this.isShowAuth = e
			},
			getRanklist: function() {
				let that = this;
				if (that.loadend) return;
				if (that.loading) return;
				that.loading = true;
				getRankList({
					page: that.page,
					limit: that.limit,
					type: that.type
				}).then(res => {
					let list = res.data;
					that.rankList.push.apply(that.rankList, list);
					if (that.page == 1) {
						that.One = that.rankList.shift() || {};
						that.Two = that.rankList.shift() || {};
						that.Three = that.rankList.shift() || {};
					}
					that.loadend = list.length < that.limit;
					that.loading = false;
					that.$set(that, 'rankList', that.rankList);
					that.One = that.One;
					that.Two = that.Two;
					that.Three = that.Three;
					that.page = that.page + 1;
				}).catch(err => {
					that.loading = false;
				})
			},

			switchTap: function(index) {
				if (this.active === index) return;
				this.active = index;
				this.type = index ? 'month' : 'week';
				this.page = 1;
				this.loadend = false;
				this.$set(this, 'rankList', []);
				this.Two = {};
				this.One = {};
				this.Three = {};
				this.getRanklist();
			},
		},
		onReachBottom: function() {
			this.getRanklist();
		}
	}
</script>

<style scoped lang="scss">
	
	.PromoterRank .redBg {
		padding: 45rpx 0 30rpx 0;
	}

	.PromoterRank .header {
		// background: url("https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/pages_users_promoter_rank_index_0.jpg") no-repeat;
		width: 100%;
		height: 460rpx;
		// background-size: 100% 100%;
	}

	.PromoterRank .header .nav {
		width: 450rpx;
		height: 66rpx;
		border: 1px solid #fff;
		border-radius: 33rpx;
		font-size: 30rpx;
		color: #fff;
		margin: 0 auto;
	}

	.PromoterRank .header .nav .item {
		width: 223rpx;
		height: 100%;
		text-align: center;
		line-height: 60rpx;
	}

	.PromoterRank .header .nav .item.font-color:nth-of-type(1) {
		background-color: #fff;
		border-radius: 33rpx 0 0 33rpx;
		color: var(--view-theme) !important;
	}

	.PromoterRank .header .nav .item.font-color:nth-of-type(2) {
		background-color: #fff;
		border-radius: 0 33rpx 33rpx 0;
		color: var(--view-theme) !important;
	}

	.PromoterRank .header .rank {
		padding: 0 20rpx;
		margin-top: 30rpx;
	}

	.PromoterRank .header .rank .item .pictrue {
		background-repeat: no-repeat;
		background-size: 100% 100%;
		position: relative;
		margin: 0 auto;
	}
	
	.PromoterRank .header .rank .item.two .pictrue{
		background-image: url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/pages_users_promoter_rank_index_1.png');
	  width: 136rpx;
	  height: 177rpx;
	}

	.PromoterRank .header .rank .item .pictrue image {
		position: absolute;
		display: block;
		bottom: 2rpx;
		border-radius: 50%;
		left: 50%;
		margin-left: -65rpx;
	}
	
	.PromoterRank .header .rank .item.two .pictrue image{
		width: 130rpx;
		height: 130rpx;
	}

	.PromoterRank .header .rank .item.one .pictrue {
		background-image: url("https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/pages_users_promoter_rank_index_2.png");
		width: 156rpx;
		height: 205rpx;
	}

	.PromoterRank .header .rank .item.one .pictrue image {
		width: 150rpx;
		height: 150rpx;
		margin-left: -75rpx;
	}

	.PromoterRank .header .rank .item.three .pictrue {
		background-image: url("https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/pages_users_promoter_rank_index_3.png");
	  width: 136rpx;
	  height: 177rpx;
	}

	.PromoterRank .header .rank .item.three .pictrue image {
		width: 130rpx;
		height: 130rpx;
		margin-left: -64rpx;
	}

	.PromoterRank .header .rank .item .name {
		font-size: 30rpx;
		color: #fff;
		margin-top: 22rpx;
		text-align: center;
		width: 170rpx;
	}

	.PromoterRank .header .rank .item .num {
		font-size: 30rpx;
		color: #fff;
		text-align: center;
	}

	.PromoterRank .list {
		width: 710rpx;
		background-color: #fff;
		border-radius: 20rpx;
		margin: -60rpx auto 0 auto;
		padding: 0 30rpx;
	}

	.PromoterRank .list .item {
		border-bottom: 1px solid #f3f3f3;
		height: 101rpx;
		font-size: 28rpx;
	}

	.PromoterRank .list .item .num {
		color: #666;
		width: 70rpx;
	}

	.PromoterRank .list .item .picTxt {
		width: 350rpx;
	}

	.PromoterRank .list .item .picTxt .pictrue {
		width: 68rpx;
		height: 68rpx;
	}

	.PromoterRank .list .item .picTxt .pictrue image {
		width: 100%;
		height: 100%;
		display: block;
		border-radius: 50%;
	}

	.PromoterRank .list .item .picTxt .text {
		width: 262rpx;
		color: #333;
	}

	.PromoterRank .list .item .people {
		width: 175rpx;
		text-align: right;
	}
</style>
