<template>
	<view>
		<view class="order-index" ref="container">
		<!-- 	<view class="header acea-row">
				<navigator class="item" url="/pages/admin/orderList/index?types=0" hover-class="none">
					<view class="num">{{ census.unpaid_count }}</view>
					<view>{{$t(`待付款`)}}</view>
				</navigator>
				<navigator class="item" url="/pages/admin/orderList/index?types=1" hover-class="none">
					<view class="num">{{ census.unshipped_count }}</view>
					<view>{{$t(`待发货`)}}</view>
				</navigator>
				<navigator class="item" url="/pages/admin/orderList/index?types=2" hover-class="none">
					<view class="num">{{ census.received_count }}</view>
					<view>{{$t(`待收货`)}}</view>
				</navigator>
				<navigator class="item" url="/pages/admin/orderList/index?types=3" hover-class="none">
					<view class="num">{{ census.evaluated_count }}</view>
					<view>{{$t(`待评价`)}}</view>
				</navigator>
				<navigator class="item" url="/pages/admin/orderList/index?types=-3" hover-class="none">
					<view class="num">{{ census.refund_count }}</view>
					<view>{{$t(`退款`)}}</view>
				</navigator>
			</view> -->
			<view class="wrapper">
				<view class="title">
					<span class="iconfont icon-shujutongji"></span>{{$t(`数据统计`)}}
				</view>
				<view class="list acea-row">
					<navigator class="item" url="/pages/admin/statistics/index?type=price&time=today" hover-class="none">
						<view class="num">{{ census.todayPrice }}</view>
						<view>{{$t(`今日成交额`)}}</view>
					</navigator>
					<navigator class="item" url="/pages/admin/statistics/index?type=price&time=yesterday" hover-class="none">
						<view class="num">{{ census.proPrice }}</view>
						<view>{{$t(`昨日成交额`)}}</view>
					</navigator>
					<navigator class="item" url="/pages/admin/statistics/index?type=price&time=month" hover-class="none">
						<view class="num">{{ census.monthPrice }}</view>
						<view>{{$t(`本月成交额`)}}</view>
					</navigator>
					<navigator class="item" url="/pages/admin/statistics/index?type=order&time=today" hover-class="none">
						<view class="num">{{ census.todayCount }}</view>
						<view>{{$t(`今日订单数`)}}</view>
					</navigator>
					<navigator class="item" url="/pages/admin/statistics/index?type=order&time=yesterday" hover-class="none">
						<view class="num">{{ census.proCount }}</view>
						<view>{{$t(`昨日订单数`)}}</view>
					</navigator>
					<navigator class="item" url="/pages/admin/statistics/index?type=order&time=month" hover-class="none">
						<view class="num">{{ census.monthCount }}</view>
						<view>{{$t(`本月订单数`)}}</view>
					</navigator>
				</view>
			</view>
			<view class="public-wrapper">
				<view class="title">
					<span class="iconfont icon-xiangxishuju"></span>{{$t(`详细数据`)}}
				</view>
				<view class="nav acea-row row-between-wrapper">
					<view class="data">{{$t(`日期`)}}</view>
					<view class="browse">{{$t(`订单数`)}}</view>
					<view class="turnover">{{$t(`成交额`)}}</view>
				</view>
				<view class="conter">
					<view class="item acea-row row-between-wrapper" v-for="(item, index) in list" :key="index">
						<view class="data">{{ item.time }}</view>
						<view class="browse">{{ item.count }}</view>
						<view class="turnover">{{ item.price }}</view>
					</view>
				</view>
			</view>
			<Loading :loaded="loaded" :loading="loading"></Loading>
		</view>
	</view>
</template>

<script>
	import {
		getStatisticsInfo,
		getStatisticsMonth
	} from "@/api/admin";
	import Loading from '../components/Loading/index.vue'
	export default {
		name: 'adminOrder',
		components: {
			Loading
		},
		data() {
			return {
				census: {},
				list: [],
				where: {
					page: 1,
					limit: 15
				},
				loaded: false,
				loading: false
			}
		},
		onShow() {
			this.clear();
			this.getIndex();
			this.getList();
			// this.$scroll(this.$refs.container, () => {
			// 	!this.loading && this.getList();
			// });
		},
		methods: {
			clear(){
				this.loaded = false;
				this.list = []
				this.where.page = 1;
			},
			getIndex: function() {
				var that = this;
				getStatisticsInfo().then(
					res => {
						that.census = res.data;
					},
					err => {
						that.$util.Tips({
							title: err
						})
					}
				);
			},
			getList: function() {
				var that = this;
				if (that.loading || that.loaded) return;
				that.loading = true;
				getStatisticsMonth(that.where).then(
					res => {
						that.loading = false;
						that.loaded = res.data.length < that.where.limit;
						that.list.push.apply(that.list, res.data);
						that.where.page = that.where.page + 1;
					},
					error => {
						that.$util.Tips({
							title: error
						})
					},
					300
				);
			}
		},
		onReachBottom(){
			this.getList()
		}
	}
</script>

<style>
	/*订单首页*/
	.order-index .header {
		background-image: url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/pages_admin_order_index_0.png');
		background-size: 100% 100%;
		width: 100%;
		height: 302upx;
		padding: 45upx 3upx 0 3upx;
		box-sizing: border-box;
	}

	.order-index .header .item {
		flex: 1;
		-webkit-flex: 1;
		-o-flex: 1;
		-ms-flex: 1;
		text-align: center;
		font-size: 24upx;
		color: #fff;
	}

	.order-index .header .item .num {
		font-size: 40upx;
		margin-bottom: 7upx;
	}

	.order-index .wrapper {
		width: 690upx;
		background-color: #fff;
		border-radius: 10upx;
		margin: 30upx auto 0 auto;
		padding-top: 25upx;
	}

	.order-index .wrapper .title {
		font-size: 30upx;
		color: #282828;
		padding: 0 30upx;
		margin-bottom: 40upx;
	}

	.order-index .wrapper .title .iconfont {
		color: #2291f8;
		font-size: 40upx;
		margin-right: 13upx;
		vertical-align: middle;
	}

	.order-index .wrapper .list .item {
		width: 33.33%;
		text-align: center;
		font-size: 24upx;
		color: #999;
		margin-bottom: 45upx;
	}

	.order-index .wrapper .list .item .num {
		font-size: 40upx;
		color: #333;
	}

	.public-wrapper .title {
		font-size: 30upx;
		color: #282828;
		padding: 0 30upx;
		margin-bottom: 20upx;
	}

	.public-wrapper .title .iconfont {
		color: #2291f8;
		font-size: 40upx;
		margin-right: 13upx;
		vertical-align: middle;
	}

	.public-wrapper {
		margin: 18upx auto 0 auto;
		width: 690upx;
		background-color: #fff;
		border-radius: 10upx;
		padding-top: 25upx;
	}

	.public-wrapper .nav {
		padding: 0 30upx;
		height: 70upx;
		line-height: 70upx;
		font-size: 24upx;
		color: #999;
	}

	.public-wrapper .data {
		width: 210upx;
		text-align: left;
	}

	.public-wrapper .browse {
		width: 200upx;
		text-align: left;
	}

	.public-wrapper .turnover {
		width: 220upx;
		text-align: right;
	}

	.public-wrapper .conter {
		padding: 0 30upx;
	}

	.public-wrapper .conter .item {
		border-bottom: 1px solid #f7f7f7;
		height: 70upx;
		font-size: 24upx;
	}

	.public-wrapper .conter .item .turnover {
		color: #d84242;
	}
</style>
