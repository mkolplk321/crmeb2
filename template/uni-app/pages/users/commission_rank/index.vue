<template>
	<view>
		<view class="CommissionRank" :style="colorStyle">
			<view class="header">
				<view class="rank" v-if="position">{{$t(`您目前的排名`)}}<text class="num">{{ position }}</text></view>
				<view class="rank" v-else>{{$t(`您目前暂无排名`)}}</view>
			</view>
			<view class="wrapper">
				<view class="nav acea-row row-around">
					<view class="item" :class="active == index ? 'fontcolor' : ''" v-for="(item,index) in navList"
						:key="index" @click="switchTap(index)">
						{{ $t(item) }}
					</view>
				</view>
				<view class="list">
					<view class="item acea-row row-between-wrapper" v-for="(item,index) in rankList" :key="index">
						<view class="num" v-if="index <= 2">
							<image :src="'../static/medal0'+(index+1)+'.png'"></image>
						</view>
						<view class="num more-num" v-else>
							{{index+1}}
						</view>
						<view class="picTxt acea-row row-between-wrapper">
							<view class="pictrue">
								<image :src="item.avatar"></image>
							</view>
							<view class="text line1">{{$t(item.nickname)}}</view>
						</view>
						<view class="people font-color">{{$t(`￥`)}}{{item.brokerage_price}}</view>
					</view>
				</view>
			</view>
			<view v-if="rankList.length == 0">
				<emptyPage v-if="!loading" :title="$t(`暂无排名~`)"></emptyPage>
				<view class="loadingicon acea-row row-center-wrapper">
					<text class="loading iconfont icon-jiazai" :hidden="loading == false"></text>
				</view>
			</view>
		</view>
		<!-- #ifdef MP -->
		<!-- <authorize @onLoadFun="onLoadFun" :isAuto="isAuto" :isShowAuth="isShowAuth" @authColse="authColse"></authorize> -->
		<!-- #endif -->
	</view>
</template>

<script>
	import emptyPage from '@/components/emptyPage.vue';
	import {
		getBrokerageRank
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
	import colors from '@/mixins/color';
	export default {
		components: {
			// #ifdef MP
			authorize,
			// #endif
			emptyPage
		},
		mixins: [colors],
		data() {
			return {
				navList: [this.$t(`周排行`), this.$t(`月排行`)],
				active: 0,
				rankList: [],
				page: 1,
				limit: 20,
				loadend: false,
				loading: false,
				loadTitle: this.$t(`加载更多`),
				type: 'week',
				position: 0,
				isAuto: false, //没有授权的不会自动授权
				isShowAuth: false //是否隐藏授权
			};
		},
		computed: mapGetters(['isLogin']),
		watch: {
			isLogin: {
				handler: function(newV, oldV) {
					if (newV) {
						this.getBrokerageRankList();
					}
				},
				deep: true
			}
		},
		onLoad() {
			if (this.isLogin) {
				this.getBrokerageRankList();
			} else {
				toLogin();
			}
		},
		methods: {
			onLoadFun: function() {
				this.getBrokerageRankList();
			},
			// 授权关闭
			authColse: function(e) {
				this.isShowAuth = e
			},
			switchTap: function(index) {
				this.active = index;
				this.type = index ? 'month' : 'week';
				this.page = 1;
				this.loadend = false;
				this.$set(this, 'rankList', []);
				this.getBrokerageRankList();
			},
			getBrokerageRankList: function() {
				if (this.loadend) return;
				if (this.loading) return;
				this.loading = true;
				this.loadTitle = '';
				getBrokerageRank({
					page: this.page,
					limit: this.limit,
					type: this.type
				}).then(res => {
					let list = res.data.rank;
					let loadend = list.length < this.limit;
					this.rankList.push.apply(this.rankList, list);
					this.loading = false;
					this.loadend = loadend;
					this.loadTitle = loadend ? this.$t(`我也是有底线的`) : this.$t(`加载更多`);
					this.$set(this, 'rankList', this.rankList);
					this.position = res.data.position;
					this.page += 1
				}).catch(err => {
					this.loading = false;
					this.loadTitle = this.$t(`加载更多`);
				})
			}
		},
		onReachBottom() {
			this.getBrokerageRankList();
		}
	}
</script>

<style scoped lang="scss">
	.CommissionRank .header {
		background: url("https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/index_0.png") no-repeat;
		width: 100%;
		height: 344rpx;
		background-size: 100% 100%;
		background-color: var(--view-theme);
	}


	.CommissionRank .header .rank {
		font-size: 33rpx;
		color: #fff;
		position: absolute;
		top: 160rpx;
		left: 48rpx;
		display: flex;
		align-items: baseline;
	}

	.CommissionRank .header .rank .num {
		font-size: 51rpx;
		font-weight: bold;
		margin: 0 10rpx;
		line-height: 51rpx;
		span{
			line-height: 51rpx;
		}
	}

	.CommissionRank .wrapper {
		width: 710rpx;
		background-color: #fff;
		border-radius: 20rpx;
		margin: -76rpx auto 0 auto;
	}

	.CommissionRank .wrapper .nav {
		height: 99rpx;
		border-bottom: 2.5rpx solid #f3f3f3;
		font-size: 30rpx;
		font-weight: bold;
		color: #999;
		line-height: 99rpx;
	}

	.CommissionRank .wrapper .nav .item.font-color {
		border-bottom: 4rpx solid var(--view-theme);
	}

	.CommissionRank .wrapper .nav .item.fontcolor {
		color: var(--view-theme);
		border-bottom: 4rpx solid var(--view-theme);
	}

	.CommissionRank .wrapper .list {
		padding: 0 30rpx;
	}

	.CommissionRank .wrapper .list .item {
		border-bottom: 1px solid #f3f3f3;
		height: 101rpx;
		font-size: 28rpx;
	}

	.CommissionRank .wrapper .list .item .num {
		color: #666;
		width: 70rpx;
	}

	.CommissionRank .wrapper .list .item .num.more-num {
		padding-left: 8rpx;
	}

	.CommissionRank .wrapper .list .item .num image {
		width: 34rpx;
		height: 40rpx;
		display: block;
	}

	.CommissionRank .wrapper .list .item .picTxt {
		width: 350rpx;
	}

	.CommissionRank .wrapper .list .item .picTxt .pictrue {
		width: 68rpx;
		height: 68rpx;
	}

	.CommissionRank .wrapper .list .item .picTxt .pictrue image {
		width: 100%;
		height: 100%;
		display: block;
		border-radius: 50%;
	}

	.CommissionRank .wrapper .list .item .picTxt .text {
		width: 262rpx;
		color: #333;
	}

	.CommissionRank .wrapper .list .item .people {
		width: 175rpx;
		text-align: right;
	}
</style>
