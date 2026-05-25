<template>
	<view class="product-con" :style="colorStyle">
		<view class="navbar" :style="{ height: navH + 'rpx', opacity: opacity }">
			<view class="navbarH" :style="'height:' + navH + 'rpx;'">
				<view class="navbarCon acea-row row-center-wrapper" :style="{ paddingRight: navbarRight + 'px' }">
					<view class="header acea-row row-center-wrapper">
						<view class="item" :class="navActive === index ? 'on' : ''" v-for="(item, index) in navList"
							:key="index" @tap="tap(index)">{{ item }}</view>
					</view>
				</view>
			</view>
		</view>
		<!-- <view class='iconfont icon-xiangzuo' :style="{top:navH/2+'rpx',opacity:(1-opacity)}" @tap='returns'></view> -->
		<view id="home" class="home acea-row row-center-wrapper iconfont icon-xiangzuo" :class="opacity>0.5?'on':''"
			:style="{ top: homeTop + 'rpx' }" v-if="returnShow" @tap="returns">
			<!-- <view class="line" v-if="returnShow"></view>
			<navigator url="/pages/index/index" class="iconfont icon-shouye4"></navigator> -->
		</view>
		<view>
			<scroll-view :scroll-top="scrollTop" scroll-y="true" scroll-with-animation="true"
				:style="'height:' + height + 'px;'" @scroll="scroll">
				<view id="past0">
					<productConSwiper :imgUrls="storeInfo.images">
					</productConSwiper>
					<view class="wrapper">
						<view class="share acea-row row-between row-bottom">
							<view class="money font-color">
								{{$(`￥`)}}
								<text class="num" v-text="storeInfo.price || 0"></text>
								<text v-if="storeInfo.spec_type">{{$t(`起`)}}</text>
								<text class="price_text">{{$t(`预售价`)}}</text>
							</view>
							<view class="iconfont icon-fenxiang" @click="listenerActionSheet"></view>
						</view>
						<view class="label acea-row row-between-wrapper" style="padding-bottom: 20rpx;">
							<view class="delete-line" v-text="$t(`￥`) + (storeInfo.ot_price || 0)"></view>
							<view v-text="$t(`已预订`)':' + (storeInfo.sales || 0) + (storeInfo.unit_name || '')"></view>
						</view>
						<view class="introduce" v-text="storeInfo.title"></view>
						<view v-if="!is_money_level && storeInfo.vip_price && storeInfo.is_vip"
							class="svip acea-row row-between-wrapper">
							<view class="">{{$t(`开通“超级会员”立省`)}}{{ diff }}{{$t(`元`)}}</view>
							<navigator url="/pages/annex/vip_paid/index">
								{{$t(`立即开通`)}}
								<text class="iconfont icon-jiantou"></text>
							</navigator>
						</view>
						<view class="presell_count">
							<view>
								<view>{{$t(`预售活动时间`)}}：</view>
								<view v-if="storeInfo.start_time && storeInfo.stop_time" class="presell_time">
									<view class='iconfont icon-shijian1'></view>
									{{storeInfo.start_time}}
									<span class='area_line'>~</span>
									{{storeInfo.stop_time}}
								</view>
							</view>
							<view>{{$t(`预售结束后`)}}{{ storeInfo.deliver_time }}{{$t(`天内发货`)}}</view>
						</view>
						<view v-if="couponList.length" class="coupon acea-row row-between-wrapper" @click="couponTap"
							style="margin-top: 0rpx;">
							<view class="hide line1 acea-row">
								{{$t(`优惠券`)}}：
								<template v-for="(item, index) in couponList">
									<view v-if="index < 2" class="activity" :key="index">
										{{$t(`满`)}}{{ item.use_min_price }}{{$t(`减`)}}{{ item.coupon_price }}</view>
								</template>
							</view>
							<view class="iconfont icon-jiantou"></view>
						</view>
						<view class="coupon acea-row row-between-wrapper" v-if="activity.length">
							<view class="line1 acea-row">
								<text>{{$t(`活动`)}}：</text>
								<view v-for="(item, index) in activity" :key="index" @click="goActivity(item)">
									<view v-if="item.type === '1' && $permission('seckill')"
										:class="index == 0 ? 'activity_pin' : '' || index == 1 ? 'activity_miao' : '' || index == 2 ? 'activity_kan' : ''">
										<text class="iconfonts iconfont icon-pintuan"></text>
										<text class="activity_title">{{$t(`参与秒杀`)}}</text>
									</view>
									<view
										:class="index == 0 ? 'activity_pin' : '' || index == 1 ? 'activity_miao' : '' || index == 2 ? 'activity_kan' : ''"
										v-if="item.type === '2' && $permission('bargain')">
										<text class="iconfonts iconfont icon-shenhezhong"></text>
										<text class="activity_title">{{$t(`参与砍价`)}}</text>
									</view>
									<view
										:class="index == 0 ? 'activity_pin' : '' || index == 1 ? 'activity_miao' : '' || index == 2 ? 'activity_kan' : ''"
										v-if="item.type === '3' && $permission('combination')">
										<text class="iconfonts iconfont icon-kanjia"></text>
										<text class="activity_title">{{$t(`参与拼团`)}}</text>
									</view>
								</view>
							</view>
						</view>
					</view>
					<view class="attribute acea-row row-between-wrapper" @click="selecAttr"
						v-if="attr.productAttr.length">
						<!-- <view style="display: flex; align-items: center; width: 90%;">
							{{ attrTxt }}：
							<view class="atterTxt line1" style="width: 82%;">{{ attrValue }}</view>
						</view>
						<view class="iconfont icon-jiantou"></view> -->
						<view class="flex">
							<view style="display: flex; align-items: center; width: 90%">
								<view class="attr-txt"> {{ attrTxt }}： </view>
								<view class="atterTxt line1" style="width: 82%">{{
						      attrValue
						    }}</view>
							</view>
							<view class="iconfont icon-jiantou"></view>
						</view>
						<view class="acea-row row-between-wrapper" style="margin-top: 7px; padding-left: 70px"
							v-if="skuArr.length > 1">
							<view class="flexs">
								<image :src="item.image" v-for="(item, index) in skuArr.slice(0, 4)" :key="index"
									class="attrImg"></image>
							</view>
							<view class="switchTxt">{{$t(`共`)}}{{ skuArr.length }}{{$t(`种规格可选`)}}</view>
						</view>
					</view>
				</view>
				<view class="userEvaluation" id="past1">
					<view class="title acea-row row-between-wrapper">
						<view>{{$t(`用户评价`)}}({{ replyCount }})</view>
						<navigator class="praise" hover-class="none"
							:url="'/pages/goods/goods_comment_list/index?product_id=' + storeInfo.product_id">
							<text class="font-color">{{ replyChance }}%</text>
							{{$t(`好评率`)}}
							<text class="iconfont icon-jiantou"></text>
						</navigator>
					</view>
					<block v-if="replyCount">
						<userEvaluation :reply="reply"></userEvaluation>
					</block>
				</view>
				<view class="product-intro" id="past3">
					<view class="title">{{$t(`产品介绍`)}}</view>
					<view class="conter">
						<jyf-parser :html="description" ref="article" :tag-style="tagStyle"></jyf-parser>
					</view>
					<!-- <rich-text :nodes="description" class="conter"></rich-text> -->
				</view>
				<view class="uni-p-b-98"></view>
			</scroll-view>
		</view>
		<view class="uni-p-b-98"></view>
		<view class="footer acea-row row-between-wrapper">
			<!-- <button open-type="contact" hover-class='none' class='item'>
				<view class='iconfont icon-kefu'></view>
				<view>客服</view>
			</button> -->

			<navigator hover-class="none" open-type="switchTab" class="item" url="/pages/index/index">
				<view class="iconfont icon-shouye6"></view>
				<view class="p_center">{{$t(`首页`)}}</view>
			</navigator>
			<view @click="setCollect" class="item">
				<view class="iconfont icon-shoucang1" v-if="storeInfo.userCollect"></view>
				<view class="iconfont icon-shoucang" v-else></view>
				<view class="p_center">{{$t(`收藏`)}}</view>
			</view>
			<view class="bnt acea-row" v-if="pay_status === 1 || pay_status === 3">
				<form @submit="goBuy" class="buy bnts bg-color-hui"><button class="buy bnts"
						form-type="submit">{{pay_status === 1?$t(`未开始`):$t(`已结束`)}}</button></form>
			</view>
			<view class="bnt acea-row"
				v-else-if="attr.productSelect.quota <= 0 || attr.productSelect.quota < attr.productSelect.cart_num">
				<form class="buy bnts bg-color-hui"><button class="buy bnts bg-color-hui"
						form-type="submit">{{$t(`已售罄`)}}</button></form>
			</view>
			<view class="bnt acea-row" v-else-if="pay_status === 2">
				<form @submit="goBuy" class="buy bnts"><button class="buy bnts" form-type="submit">{{$t(`立即购买`)}}</button></form>
			</view>
		</view>
		<!-- 		<shareRedPackets :sharePacket="sharePacket" @listenerActionSheet="listenerActionSheet"
			@closeChange="closeChange"></shareRedPackets> -->
		<!-- 组件 -->
		<productWindow :attr="attr" :isShow="0" :limitNum="1" :iSplus="1" @myevent="onMyEvent" @ChangeAttr="ChangeAttr"
			@ChangeCartNum="ChangeCartNum" @attrVal="attrVal" @iptCartNum="iptCartNum" id="product-window"
			:is_vip="is_vip" @getImg="showImg"></productWindow>
		<cus-previewImg ref="cusPreviewImg" :list="skuArr" @changeSwitch="changeSwitch"
			@shareFriend="listenerActionSheet" />
		<couponListWindow :coupon="coupon" v-if="coupon" @ChangCouponsClone="ChangCouponsClone"
			@ChangCoupons="ChangCoupons" @ChangCouponsUseState="ChangCouponsUseState" @tabCouponType="tabCouponType">
		</couponListWindow>
		<!-- 分享按钮 -->
		<view class="generate-posters acea-row row-middle" :class="posters ? 'on' : ''">
			<!-- #ifndef MP -->
			<button class="item" hover-class="none" v-if="weixinStatus === true" @click="H5ShareBox = true">
				<view class="iconfont icon-weixin3"></view>
				<view class="">{{$t(`发送给朋友`)}}</view>
			</button>
			<!-- #endif -->
			<!-- #ifdef MP -->
			<button class="item" open-type="share" hover-class="none" @click="goFriend">
				<view class="iconfont icon-weixin3"></view>
				<view class="">{{$t(`发送给朋友`)}}</view>
			</button>
			<!-- #endif -->
			<!-- #ifdef H5  -->
			<div class="item copy-data" v-if="storeInfo.command_word" :data-clipboard-text="storeInfo.command_word">
				<view class="iconfont icon-fuzhikouling"></view>
				<text>{{$t(`复制口令`)}}</text>
			</div>
			<!-- #endif -->
			<button class="item" hover-class="none" @click="goPoster">
				<view class="iconfont icon-haibao"></view>
				<view class="">{{$t(`生成海报`)}}</view>
			</button>
		</view>
		<view class="mask" v-if="posters" @click="listenerActionClose"></view>
		<!-- #ifdef MP -->
		<!-- <authorize @onLoadFun="onLoadFun" :isAuto="isAuto" :isShowAuth="isShowAuth" @authColse="authColse"></authorize> -->
		<!-- #endif -->
		<!-- 海报展示 -->
		<view class="poster-pop" v-if="posterImageStatus">
			<image src="https://goyoto.oss-cn-beijing.aliyuncs.com/images/poster-close.png" class="close" @click="posterImageClose"></image>
			<image class="poster-img" :src="posterImage"></image>
			<!-- #ifndef H5  -->
			<view class="save-poster" @click="savePosterPath">{{$t(`保存到手机`)}}</view>
			<!-- #endif -->
			<!-- #ifdef H5 -->
			<view class="keep">{{$t(`长按图片可以保存到手机`)}}</view>
			<!-- #endif -->
		</view>
		<view class="mask" v-if="posterImageStatus"></view>
		<canvas class="canvas" canvas-id="myCanvas" v-if="canvasStatus"></canvas>
		<!-- 发送给朋友图片 -->
		<view class="share-box" v-if="H5ShareBox">
			<image :src="imgHost + '/statics/images/share-info.png'" @click="H5ShareBox = false"></image>
		</view>
		<kefuIcon :ids='parseInt(id)' :routineContact="routineContact"></kefuIcon>
	</view>
</template>

<script>
	import {
		getPresellProductDetail,
		getProductCode,
		collectAdd,
		collectDel,
		postCartAdd
	} from '@/api/store.js';
	import {
		getUserInfo,
		userShare
	} from '@/api/user.js';
	import {
		getCoupons
	} from '@/api/api.js';
	import {
		getCartCounts
	} from '@/api/order.js';
	import {
		toLogin
	} from '@/libs/login.js';
	import {
		mapGetters
	} from 'vuex';
	import {
		imageBase64
	} from '@/api/public';
	import productConSwiper from '../components/productConSwiper';
	import couponListWindow from '../components/couponListWindow';
	import productWindow from '../components/productWindow';
	import userEvaluation from '../components/userEvaluation';
	import shareRedPackets from '../components/shareRedPackets';
	import kefuIcon from '../components/kefuIcon';
	import parser from '../components/jyf-parser/jyf-parser';
	import ClipboardJS from '@/plugin/clipboard/clipboard.js';
	// #ifdef MP
	import authorize from '@/components/Authorize';
	// #endif
	import colors from "@/mixins/color";
	import {HTTP_REQUEST_URL} from '@/config/app';
	import cusPreviewImg from "../components/cusPreviewImg/index.vue";
	let app = getApp();
	export default {
		components: {
			productConSwiper,
			couponListWindow,
			productWindow,
			userEvaluation,
			shareRedPackets,
			kefuIcon,
			'jyf-parser': parser,
			cusPreviewImg,
			// #ifdef MP
			authorize
			// #endif
		},
		mixins: [colors],
		directives: {
			trigger: {
				inserted(el, binging) {
					el.click();
				}
			}
		},
		data() {
			let that = this;
			return {
				imgHost:HTTP_REQUEST_URL,
				//属性是否打开
				coupon: {
					coupon: false,
					type: -1,
					list: [],
					count: []
				},
				attrTxt: that.$t(`请选择`), //属性页面提示
				attrValue: '', //已选属性
				animated: false, //购物车动画
				id: 0, //商品id
				replyCount: 0, //总评论数量
				reply: [], //评论列表
				storeInfo: {}, //商品详情
				productValue: [], //系统属性
				couponList: [], //优惠券
				cart_num: 1, //购买数量
				isAuto: false, //没有授权的不会自动授权
				isShowAuth: false, //是否隐藏授权
				isOpen: false, //是否打开属性组件
				actionSheetHidden: true,
				posterImageStatus: false,
				storeImage: '', //海报产品图
				PromotionCode: '', //二维码图片
				canvasStatus: false, //海报绘图标签
				posterImage: '', //海报路径
				posterbackgd: 'https://goyoto.oss-cn-beijing.aliyuncs.com/images/posterbackgd.png',
				sharePacket: {
					isState: true //默认不显示
				}, //分销商详细
				uid: 0, //用户uid
				circular: false,
				autoplay: false,
				interval: 3000,
				duration: 500,
				clientHeight: '',
				systemStore: {}, //门店信息
				good_list: [],
				replyChance: 0,
				CartCount: 0,
				isDown: true,
				storeSelfMention: true,
				posters: false,
				weixinStatus: false,
				attr: {
					cartAttr: false,
					productAttr: [],
					productSelect: {}
				},
				description: '',
				navActive: 0,
				H5ShareBox: false, //公众号分享图片
				activity: [],
				navH: '',
				navList: [],
				opacity: 0,
				scrollY: 0,
				topArr: [],
				toView: '',
				height: 0,
				heightArr: [],
				lock: false,
				scrollTop: 0,
				tagStyle: {
					img: 'width:100%;display:block;',
					table: 'width:100%',
					video: 'width:100%'
				},
				returnShow: true, //判断顶部返回是否出现
				diff: '',
				is_money_level: 1,
				is_vip: 0, //是否是会员
				navbarRight: 0,
				homeTop: 20,
				routineContact: '0',
				pay_status: 1,
				skuArr: [],
				selectSku: {},
			};
		},
		computed: mapGetters(['isLogin']),
		watch: {
			isLogin: {
				handler: function(newV, oldV) {
					if (newV == true) {
						this.getCouponList();
						this.getCartCount();
					}
				},
				deep: true
			},
			storeInfo: {
				handler: function() {
					this.$nextTick(() => {});
				},
				immediate: true
			}
		},
		onLoad(options) {
			let that = this;
			var pages = getCurrentPages();
			that.returnShow = pages.length === 1 ? false : true;
			// #ifdef MP
			that.navH = app.globalData.navHeight;
			// #endif
			// #ifndef MP
			that.navH = 96;
			// #endif
			that.id = options.id;
			uni.getSystemInfo({
				success: function(res) {
					that.height = res.windowHeight;
					//res.windowHeight:获取整个窗口高度为px，*2为rpx；98为头部占据的高度；
					// #ifndef APP-PLUS || H5 || MP-ALIPAY
					that.navbarRight = res.windowWidth - uni.getMenuButtonBoundingClientRect().left;
					// #endif
				}
			});
			//扫码携带参数处理
			// #ifdef MP
			if (options.scene) {
				let value = that.$util.getUrlParams(decodeURIComponent(options.scene));
				if (value.id) options.id = value.id;
				//记录推广人uid
				if (value.pid) app.globalData.spid = value.pid;
			}
			if (!options.id) {
				return that.$util.Tips({
					title: that.$t(`缺少参数无法查看商品`)
				}, {
					tab: 3,
					url: 1
				});
			} else {
				that.id = options.id;
			}
			//记录推广人uid
			if (options.pid) app.globalData.spid = options.pid;
			if (options.spid) app.globalData.spid = options.spid;
			// #endif
			that.getGoodsDetails();
		},
		onReady: function() {
			this.$nextTick(function() {
				// #ifdef MP
				const menuButton = uni.getMenuButtonBoundingClientRect();
				const query = uni.createSelectorQuery().in(this);
				query
					.select('#home')
					.boundingClientRect(data => {
						this.homeTop = menuButton.top * 2 + menuButton.height - data.height;
					})
					.exec();
				// #endif
				// #ifdef H5
				const clipboard = new ClipboardJS('.copy-data');
				clipboard.on('success', () => {
					this.$util.Tips({
						title: this.$t(`复制成功`)
					});
				});
				// #endif
			});
		},
		/**
		 * 用户点击右上角分享
		 */
		// #ifdef MP
		onShareAppMessage: function() {
			let that = this;
			that.$set(that, 'actionSheetHidden', !that.actionSheetHidden);
			userShare();
			return {
				title: that.storeInfo.store_name || '',
				imageUrl: that.storeInfo.image || '',
				path: '/pages/goods_details/index?id=' + that.id + '&pid=' + that.uid
			};
		},
		// #endif
		methods: {
			closeChange: function() {
				this.$set(this.sharePacket, 'isState', true);
			},
			goActivity: function(e) {
				let item = e;
				if (item.type === '1') {
					uni.navigateTo({
						url: `/pages/activity/goods_seckill_details/index?id=${item.id}&time_id=${item.time_id}`
					});
				} else if (item.type === '2') {
					uni.navigateTo({
						url: `/pages/activity/goods_bargain_details/index?id=${item.id}&bargain=${this.uid}`
					});
				} else {
					uni.navigateTo({
						url: `/pages/activity/goods_combination_details/index?id=${item.id}`
					});
				}
			},
			/**
			 * 购物车手动填写
			 *
			 */
			iptCartNum: function(e) {
				this.$set(this.attr.productSelect, 'cart_num', e);
			},
			// 后退
			returns: function() {
				uni.navigateBack();
			},
			tap: function(index) {
				var id = 'past' + index;
				var index = index;
				var that = this;
				// if (!this.data.good_list.length && id == "past2") {
				//   id = "past3"
				// }
				this.$set(this, 'toView', id);
				this.$set(this, 'navActive', index);
				this.$set(this, 'lock', true);
				this.$set(this, 'scrollTop', index > 0 ? that.topArr[index] - app.globalData.navHeight / 2 : that
					.topArr[index]);
			},
			scroll: function(e) {
				var that = this,
					scrollY = e.detail.scrollTop;
				var opacity = scrollY / 200;
				opacity = opacity > 1 ? 1 : opacity;
				that.$set(that, 'opacity', opacity);
				that.$set(that, 'scrollY', scrollY);
				if (that.lock) {
					that.$set(that, 'lock', false);
					return;
				}
				for (var i = 0; i < that.topArr.length; i++) {
					if (scrollY < that.topArr[i] - app.globalData.navHeight / 2 + that.heightArr[i]) {
						that.$set(that, 'navActive', i);
						break;
					}
				}
			},
			/*
			 *去商品详情页
			 */
			goDetail(item) {
				if (item.activity.length == 0) {
					uni.redirectTo({
						url: '/pages/goods_details/index?id=' + item.id
					});
					return;
				}
				// 砍价
				if (item.activity && item.activity.type == 2) {
					uni.redirectTo({
						url: `/pages/activity/goods_bargain_details/index?id=${item.activity.id}&bargain=${this.uid}`
					});
					return;
				}
				// 拼团
				if (item.activity && item.activity.type == 3) {
					uni.redirectTo({
						url: `/pages/activity/goods_combination_details/index?id=${item.activity.id}`
					});
					return;
				}
				// 秒杀
				if (item.activity && item.activity.type == 1) {
					uni.redirectTo({
						url: `/pages/activity/goods_seckill_details/index?id=${item.activity.id}&time_id=${item.activity.time_id}`
					});
					return;
				}
			},
			// 微信登录回调
			onLoadFun: function(e) {
				// this.getUserInfo();
				// this.get_product_collect();
			},
			ChangCouponsClone: function() {
				this.$set(this.coupon, 'coupon', false);
			},
			/*
			 * 获取用户信息
			 */
			getUserInfo: function() {
				let that = this;
				getUserInfo().then(res => {
					that.$set(that.sharePacket, 'isState', that.sharePacket.priceName != 0 ? false : true);
					that.$set(that, 'uid', res.data.uid);
					that.$set(that, 'is_money_level', res.data.is_money_level);
				});
			},
			/**
			 * 购物车数量加和数量减
			 *
			 */
			ChangeCartNum: function(changeValue) {
				//changeValue:是否 加|减
				//获取当前变动属性
				let productSelect = this.productValue[this.attrValue];
				//如果没有属性,赋值给商品默认库存
				if (productSelect === undefined && !this.attr.productAttr.length) productSelect = this.attr
					.productSelect;
				//无属性值即库存为0；不存在加减；
				if (productSelect === undefined) return;
				let stock = productSelect.stock || 0;
				let num = this.attr.productSelect;
				if (changeValue) {
					num.cart_num++;
					if (num.cart_num > stock) {
						this.$set(this.attr.productSelect, 'cart_num', stock ? stock : 1);
						this.$set(this, 'cart_num', stock ? stock : 1);
					}
				} else {
					num.cart_num--;
					if (num.cart_num < 1) {
						this.$set(this.attr.productSelect, 'cart_num', 1);
						this.$set(this, 'cart_num', 1);
					}
				}
			},
			attrVal(val) {
				this.$set(this.attr.productAttr[val.indexw], 'index', this.attr.productAttr[val.indexw].attr_values[val
					.indexn]);
			},
			/**
			 * 属性变动赋值
			 *
			 */
			ChangeAttr: function(res) {
				let productSelect = this.productValue[res];
				this.$set(this, "selectSku", productSelect);
				if (productSelect && productSelect.stock > 0) {
					this.$set(this.attr.productSelect, 'image', productSelect.image);
					this.$set(this.attr.productSelect, 'price', productSelect.price);
					this.$set(this.attr.productSelect, 'stock', productSelect.stock);
					this.$set(this.attr.productSelect, 'unique', productSelect.unique);
					this.$set(this.attr.productSelect, 'quota', productSelect.quota);
					this.$set(this.attr.productSelect, 'cart_num', 1);
					this.$set(this.attr.productSelect, 'vip_price', productSelect.vip_price);
					this.$set(this, 'attrValue', res);
					this.$set(this, 'attrTxt', this.$t(`已选择`));
				} else {
					this.$set(this.attr.productSelect, 'image', productSelect.image);
					this.$set(this.attr.productSelect, 'price', this.storeInfo.price);
					this.$set(this.attr.productSelect, 'stock', 0);
					this.$set(this.attr.productSelect, 'unique', '');
					this.$set(this.attr.productSelect, 'cart_num', 0);
					this.$set(this.attr.productSelect, 'quota', productSelect.quota);
					this.$set(this.attr.productSelect, 'vip_price', this.storeInfo.vip_price);
					this.$set(this, 'attrValue', '');
					this.$set(this, 'attrTxt', this.$t(`请选择`));
				}
			},
			/**
			 * 领取完毕移除当前页面领取过的优惠券展示
			 */
			ChangCoupons: function(e) {
				let coupon = e;
				let couponList = this.$util.ArrayRemove(this.couponList, 'id', coupon.id);
				this.$set(this, 'couponList', couponList);
				this.getCouponList();
			},

			setClientHeight: function() {
				let that = this;
				if (!that.good_list.length) return;
				let view = uni
					.createSelectorQuery()
					.in(this)
					.select('#list0');
				view.fields({
						size: true
					},
					data => {
						that.$set(that, 'clientHeight', data.height + 20);
					}
				).exec();
			},
			/**
			 * 获取产品详情
			 *
			 */
			getGoodsDetails: function() {
				let that = this;
				getPresellProductDetail(that.id)
					.then(res => {
						let storeInfo = res.data.storeInfo;
						let goodArray = new Array();
						that.$set(that, 'storeInfo', storeInfo);
						that.$set(that, 'description', storeInfo.description);
						that.$set(that, 'reply', res.data.reply ? [res.data.reply] : []);
						that.$set(that, 'replyCount', res.data.replyCount);
						that.$set(that, 'pay_status', res.data.pay_status); // 1未开始;2进行中;3已结束
						that.$set(that, 'replyChance', res.data.replyChance);
						that.$set(that.attr, 'productAttr', res.data.productAttr);
						that.$set(that, 'productValue', res.data.productValue);
						that.$set(that, 'PromotionCode', storeInfo.code_base);
						that.$set(that, 'routineContact', Number(res.data.routine_contact_type));
						uni.setNavigationBarTitle({
							title: storeInfo.title.substring(0, 7) + '...'
						});
						for (let key in res.data.productValue) {
							let obj = res.data.productValue[key];
							that.skuArr.push(obj);
						}
						that.$set(that, "selectSku", that.skuArr[0]);
						var navList = [that.$t(`商品`), that.$t(`评价`), that.$t(`详情`)];
						if (goodArray.length) {
							navList.splice(2, 0, that.$t(`推荐`));
						}
						that.$set(that, 'navList', navList);
						// #ifdef H5
						that.$set(that, 'storeImage', that.storeInfo.image);
						that.getImageBase64();
						if (that.isLogin) {
							that.ShareInfo();
						}
						// #endif
						if (that.isLogin) {
							that.getUserInfo();
						}
						this.$nextTick(() => {
							// if (good_list.length) {
							// 	that.setClientHeight();
							// }
						});
						setTimeout(function() {
							that.infoScroll();
						}, 500);
						// #ifdef APP-PLUS
						uni.downloadFile({
							url: that.setDomain(res.data.storeInfo.code_base),
							success: function(res) {
								that.PromotionCode = res.tempFilePath;
							},
							fail: function() {
								return that.$util.Tips({
									title: that.$t(`二维码获取失败`)
								});
							},
						});


						// that.PromotionCode = res.data.storeInfo.code_base

						that.downloadFilestoreImage();
						// #endif
						// #ifndef H5 || APP-PLUS
						that.downloadFilestoreImage();
						that.downloadFilePromotionCode();
						// #endif
						that.DefaultSelect();
						that.getCartCount();
					})
					.catch(err => {
						return that.$util.Tips({
							title: err.toString()
						}, {
							tab: 3,
							url: 1
						});
					});
			},
			infoScroll: function() {
				var that = this,
					topArr = [],
					heightArr = [];
				for (var i = 0; i < that.navList.length; i++) {
					//productList
					//获取元素所在位置
					var query = uni.createSelectorQuery().in(this);
					var idView = '#past' + i;
					// if (!that.data.good_list.length && i == 2) {
					//   var idView = "#past" + 3;
					// }
					query.select(idView).boundingClientRect();
					query.exec(function(res) {
						var top = res[0].top;
						var height = res[0].height;
						topArr.push(top);
						heightArr.push(height);
						that.$set(that, 'topArr', topArr);
						that.$set(that, 'heightArr', heightArr);
					});
				}
			},
			/**
			 * 拨打电话
			 */
			makePhone: function() {
				uni.makePhoneCall({
					phoneNumber: this.systemStore.phone
				});
			},
			/**
			 * 打开地图
			 *
			 */
			showMaoLocation: function() {
				if (!this.systemStore.latitude || !this.systemStore.longitude)
					return this.$util.Tips({
						title: this.$t(`缺少经纬度信息无法查看地图`)
					});
				uni.openLocation({
					latitude: parseFloat(this.systemStore.latitude),
					longitude: parseFloat(this.systemStore.longitude),
					scale: 8,
					name: this.systemStore.name,
					address: this.systemStore.address + this.systemStore.detailed_address,
					success: function() {}
				});
			},
			/**
			 * 默认选中属性
			 *
			 */
			DefaultSelect: function() {
				let productAttr = this.attr.productAttr;
				let value = [];
				for (var key in this.productValue) {
					if (this.productValue[key].stock > 0) {
						value = this.attr.productAttr.length ? key.split(',') : [];
						break;
					}
				}
				for (let i = 0; i < productAttr.length; i++) {
					this.$set(productAttr[i], 'index', value[i]);
				}
				//sort();排序函数:数字-英文-汉字；
				let productSelect = this.productValue[value.join(',')];
				if (productSelect && productAttr.length) {
					this.$set(this.attr.productSelect, 'store_name', this.storeInfo.store_name);
					this.$set(this.attr.productSelect, 'image', productSelect.image);
					this.$set(this.attr.productSelect, 'price', productSelect.price);
					this.$set(this.attr.productSelect, 'stock', productSelect.stock);
					this.$set(this.attr.productSelect, 'unique', productSelect.unique);
					this.$set(this.attr.productSelect, 'cart_num', 1);
					this.$set(this.attr.productSelect, 'quota', productSelect.quota);
					this.$set(this, 'attrValue', value.join(','));
					this.$set(this.attr.productSelect, 'vip_price', productSelect.vip_price);
					this.$set(this, 'attrTxt', this.$t(`已选择`));
				} else if (!productSelect && productAttr.length) {
					this.$set(this.attr.productSelect, 'store_name', this.storeInfo.store_name);
					this.$set(this.attr.productSelect, 'image', this.storeInfo.image);
					this.$set(this.attr.productSelect, 'price', this.storeInfo.price);
					this.$set(this.attr.productSelect, 'stock', 0);
					this.$set(this.attr.productSelect, 'unique', '');
					this.$set(this.attr.productSelect, 'cart_num', 0);
					this.$set(this.attr.productSelect, 'quota', productSelect.quota);
					this.$set(this.attr.productSelect, 'vip_price', this.storeInfo.vip_price);
					this.$set(this, 'attrValue', '');
					this.$set(this, 'attrTxt', this.$t(`请选择`));
				} else if (!productSelect && !productAttr.length) {
					this.$set(this.attr.productSelect, 'store_name', this.storeInfo.store_name);
					this.$set(this.attr.productSelect, 'image', this.storeInfo.image);
					this.$set(this.attr.productSelect, 'price', this.storeInfo.price);
					this.$set(this.attr.productSelect, 'stock', this.storeInfo.stock);
					this.$set(this.attr.productSelect, 'unique', this.storeInfo.unique || '');
					this.$set(this.attr.productSelect, 'cart_num', 1);
					this.$set(this.attr.productSelect, 'vip_price', this.storeInfo.vip_price);
					this.$set(this, 'attrValue', '');
					this.$set(this, 'attrTxt', this.$t(`请选择`));
				}
			},
			/**
			 * 获取优惠券
			 *
			 */
			getCouponList(type) {
				let that = this,
					obj = {
						page: 1,
						limit: 20,
						product_id: that.id
					};
				if (type !== undefined || type !== null) {
					obj.type = type;
				}
				getCoupons(obj).then(res => {
					that.$set(that.coupon, 'count', res.data.count);
					if (type === undefined || type === null) {
						let count = [...that.coupon.count],
							indexs = '';
						let index = count.findIndex(item => item);
						let delCount = that.coupon.count,
							newDelCount = [];
						let countIndex = 0;
						delCount.forEach((item, index) => {
							if (item === 0) {
								countIndex = index;
							} else {
								newDelCount.push(item)
							}
						});
						if (newDelCount.length == 3) {
							indexs = 2;
						} else if (newDelCount.length == 2) {
							if (countIndex === 2) {
								indexs = 1;
							} else {
								indexs = 2;
							}
						} else {
							indexs = delCount.findIndex(item => item === count[index]);
						}
						that.$set(that.coupon, 'type', indexs);
						that.getCouponList(indexs);
					} else {
						that.$set(that.coupon, 'list', res.data.list);
					}
				});
			},
			ChangCouponsUseState(index) {
				let that = this;
				that.coupon.list[index].is_use = true;
				that.$set(that.coupon, 'list', that.coupon.list);
				that.$set(that.coupon, 'coupon', false);
			},
			/**
			 *
			 *
			 * 收藏商品
			 */
			setCollect: function() {
				if (this.isLogin === false) {
					toLogin();
				} else {
					let that = this;
					if (this.storeInfo.userCollect) {
						collectDel([this.storeInfo.product_id]).then(res => {
							that.$set(that.storeInfo, 'userCollect', !that.storeInfo.userCollect);
							return that.$util.Tips({
								title: res.msg
							});
						});
					} else {
						collectAdd(this.storeInfo.product_id).then(res => {
							that.$set(that.storeInfo, 'userCollect', !that.storeInfo.userCollect);
							return that.$util.Tips({
								title: res.msg
							});
						});
					}
				}
			},
			/**
			 * 打开属性插件
			 */
			selecAttr: function() {
				this.$set(this.attr, 'cartAttr', true);
				this.$set(this, 'isOpen', true);
			},
			/**
			 * 打开优惠券插件
			 */
			couponTap: function() {
				let that = this;
				if (that.isLogin === false) {
					toLogin();
				} else {
					that.getCouponList();
					that.$set(that.coupon, 'coupon', true);
				}
			},
			onMyEvent: function() {
				this.$set(this.attr, 'cartAttr', false);
				this.$set(this, 'isOpen', false);
			},
			/**
			 * 打开属性加入购物车
			 *
			 */
			joinCart: function(e) {
				//是否登录
				if (this.isLogin === false) {
					toLogin();
				} else {
					this.goCat();
				}
			},
			/*
			 * 加入购物车
			 */
			goCat: function(news) {
				let that = this,
					productSelect = that.productValue[this.attrValue];
				//打开属性
				if (that.attrValue) {
					//默认选中了属性，但是没有打开过属性弹窗还是自动打开让用户查看默认选中的属性
					that.attr.cartAttr = !that.isOpen ? true : false;
				} else {
					if (that.isOpen) that.attr.cartAttr = true;
					else that.attr.cartAttr = !that.attr.cartAttr;
				}
				//只有关闭属性弹窗时进行加入购物车
				if (that.attr.cartAttr === true && that.isOpen === false) return (that.isOpen = true);
				//如果有属性,没有选择,提示用户选择
				if (that.attr.productAttr.length && productSelect === undefined && that.isOpen === true)
					return that.$util.Tips({
						title: that.$t(`产品库存不足，请选择其它属性`)
					});
				let q = {
					productId: that.storeInfo.product_id,
					advanceId: that.id,
					cartNum: that.attr.productSelect.cart_num,
					uniqueId: that.attr.productSelect !== undefined ? that.attr.productSelect.unique : '',
					'new': 1
				};
				postCartAdd(q)
					.then(function(res) {
						that.isOpen = false;
						that.attr.cartAttr = false;
						uni.navigateTo({
							url: '/pages/goods/order_confirm/index?new=1&cartId=' + res.data.cartId
						});
					})
					.catch(err => {
						that.isOpen = false;
						return that.$util.Tips({
							title: err
						});
					});
			},
			/**
			 * 获取购物车数量
			 * @param boolean 是否展示购物车动画和重置属性
			 */
			getCartCount: function(isAnima) {
				let that = this;
				const isLogin = that.isLogin;
				if (isLogin) {
					getCartCounts().then(res => {
						that.CartCount = res.data.count;
						//加入购物车后重置属性
						if (isAnima) {
							that.animated = true;
							setTimeout(function() {
								that.animated = false;
							}, 500);
						}
					});
				}
			},
			/**
			 * 立即购买
			 */
			goBuy: function(e) {
				if (this.isLogin === false) {
					toLogin();
				} else {
					this.goCat(true);
				}
			},
			// 授权关闭
			authColse: function(e) {
				this.isShowAuth = e;
			},
			/**
			 * 分享打开
			 *
			 */
			listenerActionSheet: function() {
				if (this.isLogin === false) {
					toLogin();
				} else {
					// #ifdef H5
					if (this.$wechat.isWeixin() === true) {
						this.weixinStatus = true;
					}
					// #endif
					this.posters = true;

				}
			},
			// 分享关闭
			listenerActionClose: function() {
				this.posters = false;
			},
			//隐藏海报
			posterImageClose: function() {
				this.posterImageStatus = false;
			},
			//替换安全域名
			setDomain: function(url) {
				url = url ? url.toString() : '';
				//本地调试打开,生产请注销
				if (url.indexOf('https://') > -1) return url;
				else return url.replace('http://', 'https://');
			},
			//获取海报产品图
			downloadFilestoreImage: function() {
				let that = this;
				uni.downloadFile({
					url: that.setDomain(that.storeInfo.image),
					success: function(res) {
						that.storeImage = res.tempFilePath;
					},
					fail: function() {
						return that.$util.Tips({
							title: ''
						});
						that.storeImage = '';
					}
				});
			},
			/**
			 * 获取产品分销二维码
			 * @param function successFn 下载完成回调
			 *
			 */
			downloadFilePromotionCode: function(successFn) {
				let that = this;
				getProductCode(that.storeInfo.product_id)
					.then(res => {
						uni.downloadFile({
							url: that.setDomain(res.data.code),
							success: function(res) {
								that.$set(that, 'isDown', false);
								if (typeof successFn == 'function') successFn && successFn(res
									.tempFilePath);
								else that.$set(that, 'PromotionCode', res.tempFilePath);
							},
							fail: function() {
								that.$set(that, 'isDown', false);
								that.$set(that, 'PromotionCode', '');
							}
						});
					})
					.catch(err => {
						that.$set(that, 'isDown', false);
						that.$set(that, 'PromotionCode', '');
					});
			},
			getImageBase64: function() {
				let that = this;
				imageBase64(that.storeImage, that.PromotionCode)
					.then(res => {
						that.storeImage = res.data.image;
						that.PromotionCode = res.data.code;
					})
					.catch(() => {});
			},
			// 小程序关闭分享弹窗；
			goFriend: function() {
				this.posters = false;
			},
			/**
			 * 生成海报
			 */
			goPoster: function() {
				let that = this;
				that.posters = false;
				that.$set(that, 'canvasStatus', true);
				let arr2 = [that.posterbackgd, that.storeImage, that.PromotionCode];
				uni.getImageInfo({
					src: that.PromotionCode,
					fail: function(res) {
						// #ifdef H5
						return that.$util.Tips({
							title: res,
						});
						// #endif
						// #ifdef MP
						return that.$util.Tips({
							title: that.$t(`正在下载海报,请稍后再试`),
						});
						// #endif
					},
					success() {
						if (arr2[2] == '') {
							//海报二维码不存在则从新下载
							that.downloadFilePromotionCode(function(msgPromotionCode) {
								arr2[2] = msgPromotionCode;
								if (arr2[2] == '')
									return that.$util.Tips({
										title: that.$t(`海报二维码生成失败`)
									});
								that.$util.PosterCanvas(arr2, that.storeInfo.title, that.storeInfo
									.price, that.storeInfo.ot_price,
									function(tempFilePath) {
										that.$set(that, 'posterImage', tempFilePath);
										that.$set(that, 'posterImageStatus', true);
										that.$set(that, 'canvasStatus', false);
										that.$set(that, 'actionSheetHidden', !that
											.actionSheetHidden);
									});
							});

						} else {
							//生成推广海报
							that.$util.PosterCanvas(arr2, that.storeInfo.title, that.storeInfo.price, that
								.storeInfo.ot_price,
								function(tempFilePath) {
									that.$set(that, 'posterImage', tempFilePath);
									that.$set(that, 'posterImageStatus', true);
									that.$set(that, 'canvasStatus', false);
									that.$set(that, 'actionSheetHidden', !that.actionSheetHidden);
								});
						}
					},
				});
			},

			/*
			 * 保存到手机相册
			 */
			// #ifdef MP
			copyCommand: function() {
				if (wx.navigateToMiniProgram) {
					wx.navigateToMiniProgram({
						appId: 'wxb036cafe2994d7d0',
						path: '/publish/ugc-publish/ugc-publish',
						extraData: {
							productInfo: {
								title: this.storeInfo.store_name,
								path: '/pages/goods_details/index?id=' + this.storeInfo.id,
								thumbUrl: this.storeInfo.image
							}
						}
					});
				}
			},
			savePosterPath: function() {
				let that = this;
				uni.getSetting({
					success(res) {
						if (!res.authSetting['scope.writePhotosAlbum']) {
							uni.authorize({
								scope: 'scope.writePhotosAlbum',
								success() {
									uni.saveImageToPhotosAlbum({
										filePath: that.posterImage,
										success: function(res) {
											that.posterImageClose();
											that.$util.Tips({
												title: that.$t(`保存成功`),
												icon: 'success'
											});
										},
										fail: function(res) {
											that.$util.Tips({
												title: that.$t(`保存失败`)
											});
										}
									});
								}
							});
						} else {
							uni.saveImageToPhotosAlbum({
								filePath: that.posterImage,
								success: function(res) {
									that.posterImageClose();
									that.$util.Tips({
										title: that.$t(`保存成功`),
										icon: 'success'
									});
								},
								fail: function(res) {
									that.$util.Tips({
										title: that.$t(`保存失败`)
									});
								}
							});
						}
					}
				});
			},
			// #endif
			//#ifdef H5
			ShareInfo() {
				let data = this.storeInfo;
				let href = location.href;
				if (this.$wechat.isWeixin()) {
					getUserInfo().then(res => {
						href = href.indexOf('?') === -1 ? href + '?spread=' + res.data.uid : href + '&spread=' +
							res.data.uid;

						let configAppMessage = {
							desc: data.store_info,
							title: data.store_name,
							link: href,
							imgUrl: data.image
						};
						this.$wechat
							.wechatEvevt(['updateAppMessageShareData', 'updateTimelineShareData',
								'onMenuShareAppMessage', 'onMenuShareTimeline'
							], configAppMessage)
							.then(res => {
							})
							.catch(err => {
							});
					});
				}
			},
			//#endif
			tabCouponType: function(type) {
				this.$set(this.coupon, 'type', type);
				this.getCouponList(type);
			},
			//点击sku图片打开轮播图
			showImg(index) {
				this.$refs.cusPreviewImg.open(this.selectSku.suk);
			},
			//滑动轮播图选择商品
			changeSwitch(e) {
				let productSelect = this.skuArr[e];
				this.$set(this, "selectSku", productSelect);
				var skuList = productSelect.suk.split(",");
				this.$set(this.attr.productAttr[0], "index", skuList[0]);
				if (skuList.length == 2) {
					this.$set(this.attr.productAttr[0], "index", skuList[0]);
					this.$set(this.attr.productAttr[1], "index", skuList[1]);
				} else if (skuList.length == 3) {
					this.$set(this.attr.productAttr[0], "index", skuList[0]);
					this.$set(this.attr.productAttr[1], "index", skuList[1]);
					this.$set(this.attr.productAttr[2], "index", skuList[2]);
				} else if (skuList.length == 4) {
					this.$set(this.attr.productAttr[0], "index", skuList[0]);
					this.$set(this.attr.productAttr[1], "index", skuList[1]);
					this.$set(this.attr.productAttr[2], "index", skuList[2]);
			 	this.$set(this.attr.productAttr[3], "index", skuList[3]);
				}
				if (productSelect) {
					this.$set(this.attr.productSelect, "image", productSelect.image);
					this.$set(this.attr.productSelect, "price", productSelect.price);
			 	this.$set(this.attr.productSelect, "stock", productSelect.stock);
					this.$set(this.attr.productSelect, "unique", productSelect.unique);
					this.$set(this.attr.productSelect, "vipPrice", productSelect.vipPrice);
					this.$set(this, "attrTxt", this.$t(`已选择`));
					this.$set(this, "attrValue", productSelect.suk);
				}
			},
		}
	};
</script>

<style scoped lang="scss">
@import "@/plugin/animate/animate.min.css";
	.activity_pin {
		width: auto;
		height: 44rpx;
		line-height: 44rpx;
		background: linear-gradient(90deg, rgba(233, 51, 35, 1) 0%, rgba(250, 101, 20, 1) 100%);
		opacity: 1;
		border-radius: 22rpx;
		padding: 0 15rpx;
		margin-left: 19rpx;
	}

	.activity_miao {
		width: auto;
		height: 44rpx;
		line-height: 44rpx;
		padding: 0 15rpx;
		background: linear-gradient(90deg, rgba(250, 102, 24, 1) 0%, rgba(254, 161, 15, 1) 100%);
		opacity: 1;
		border-radius: 22rpx;
		margin-left: 19rpx;
	}

	.iconfonts {
		color: #fff !important;
		font-size: 28rpx;
	}

	.activity_title {
		font-size: 24rpx;
		color: #fff;
	}

	.activity_kan {
		width: auto;
		height: 44rpx;
		line-height: 44rpx;
		padding: 0 15rpx;
		background: linear-gradient(90deg, rgba(254, 159, 15, 1) 0%, rgba(254, 178, 15, 1) 100%);
		opacity: 1;
		border-radius: 22rpx;
		margin-left: 19rpx;
	}

	.mask {
		z-index: 300 !important;
	}

	.head-bar {
		background: #fff;
	}

	.generate-posters {
		width: 100%;
		height: 170rpx;
		background-color: #fff;
		position: fixed;
		left: 0;
		bottom: 0;
		z-index: 388;
		transform: translate3d(0, 100%, 0);
		transition: all 0.3s cubic-bezier(0.25, 0.5, 0.5, 0.9);
		border-top: 1rpx solid #eee;
	}

	.generate-posters.on {
		transform: translate3d(0, 0, 0);
	}

	.generate-posters .item {
		flex: 1;
		text-align: center;
		font-size: 30rpx;
	}

	.generate-posters .item .iconfont {
		font-size: 80rpx;
		color: #5eae72;
	}

	.generate-posters .item .iconfont.icon-haibao {
		color: #5391f1;
	}

	.generate-posters .item .iconfont.icon-haowuquan1 {
		color: #ff954d;
	}

	.product-con .footer {
		padding: 0 20rpx 0 0rpx;
		position: fixed;
		bottom: 0;
		width: 100%;
		box-sizing: border-box;
		background-color: #fff;
		z-index: 277;
		border-top: 1rpx solid #f0f0f0;
		height: 100rpx;
		display: flex;
		flex-wrap: nowrap;
		height: calc(100rpx + constant(safe-area-inset-bottom)); ///兼容 IOS<11.2/
		height: calc(100rpx + env(safe-area-inset-bottom)); ///兼容 IOS>11.2/
	}

	.product-con .footer .item {
		font-size: 18rpx;
		padding: 0 20rpx;
		color: #666;
	}

	.product-con .footer .item .iconfont {
		text-align: center;
		font-size: 40rpx;
	}

	.product-con .footer .item .iconfont.icon-shoucang1 {
		color: var(--view-theme);
	}

	.product-con .footer .item .iconfont.icon-gouwuche1 {
		font-size: 40rpx;
		position: relative;
	}

	.product-con .footer .item .iconfont.icon-gouwuche1 .num {
		color: #fff;
		position: absolute;
		font-size: 18rpx;
		padding: 2rpx 10rpx 3rpx;
		border-radius: 200rpx;
		top: -10rpx;
		right: -10rpx;
	}

	.product-con .footer .bnt {
		flex: 1;
		height: 76rpx;
	}

	.product-con .footer .bnt .bnts {
		width: 100%;
		text-align: center;
		line-height: 76rpx;
		color: #fff;
		font-size: 28rpx;
		border-radius: 50rpx;
		background-color: var(--view-theme);
	}

	.product-con .store-info {
		margin-top: 20rpx;
		background-color: #fff;
	}

	.product-con .store-info .title {
		padding: 0 30rpx;
		font-size: 28rpx;
		color: #282828;
		height: 80rpx;
		line-height: 80rpx;
		border-bottom: 1px solid #f5f5f5;
	}

	.product-con .store-info .info {
		padding: 0 30rpx;
		height: 126rpx;
	}

	.product-con .store-info .info .picTxt {
		width: 615rpx;
	}

	.product-con .store-info .info .picTxt .pictrue {
		width: 76rpx;
		height: 76rpx;
	}

	.product-con .store-info .info .picTxt .pictrue image {
		width: 100%;
		height: 100%;
		border-radius: 6rpx;
	}

	.product-con .store-info .info .picTxt .text {
		width: 522rpx;
	}

	.product-con .store-info .info .picTxt .text .name {
		font-size: 30rpx;
		color: #282828;
	}

	.product-con .store-info .info .picTxt .text .address {
		font-size: 24rpx;
		color: #666;
		margin-top: 3rpx;
	}

	.product-con .store-info .info .picTxt .text .address .iconfont {
		color: #707070;
		font-size: 18rpx;
		margin-left: 10rpx;
	}

	.product-con .store-info .info .picTxt .text .address .addressTxt {
		max-width: 480rpx;
	}

	.product-con .store-info .info .iconfont {
		font-size: 40rpx;
	}

	.product-con .superior {
		background-color: #fff;
		margin-top: 20rpx;
		padding-bottom: 10rpx;
	}

	.product-con .superior .title {
		height: 98rpx;
	}

	.product-con .superior .title image {
		width: 30rpx;
		height: 30rpx;
	}

	.product-con .superior .title .titleTxt {
		margin: 0 20rpx;
		font-size: 30rpx;
		background-image: linear-gradient(to right, #f57a37 0%, #f21b07 100%);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
	}

	.product-con .superior .slider-banner {
		width: 690rpx;
		margin: 0 auto;
		position: relative;
	}

	.product-con .superior .slider-banner swiper {
		height: 100%;
		width: 100%;
	}

	.product-con .superior .slider-banner swiper-item {
		height: 100%;
	}

	.product-con .superior .slider-banner .list {
		width: 100%;
	}

	.product-con .superior .slider-banner .list .item {
		width: 215rpx;
		margin: 0 22rpx 30rpx 0;
		font-size: 26rpx;
	}

	.product-con .superior .slider-banner .list .item:nth-of-type(3n) {
		margin-right: 0;
	}

	.product-con .superior .slider-banner .list .item .pictrue {
		position: relative;
		width: 100%;
		height: 215rpx;
		border-radius: 20rpx;
	}

	.product-con .superior .slider-banner .list .item .pictrue image {
		width: 100%;
		height: 100%;
		border-radius: 20rpx;
	}

	.product-con .superior .slider-banner .list .item .name {
		color: #282828;
		margin-top: 12rpx;
	}

	.product-con .superior .slider-banner .swiper-pagination-bullet {
		background-color: #999;
	}

	.product-con .superior .slider-banner .swiper-pagination-bullet-active {
		background-color: #e93323;
	}

	button {
		padding: 0;
		margin: 0;
		line-height: normal;
		background-color: #fff;
	}

	button::after {
		border: 0;
	}

	action-sheet-item {
		padding: 0;
		height: 240rpx;
		align-items: center;
		display: flex;
	}

	.contact {
		font-size: 16px;
		width: 50%;
		background-color: #fff;
		padding: 8rpx 0;
		border-radius: 0;
		margin: 0;
		line-height: 2;
	}

	.contact::after {
		border: none;
	}

	.action-sheet {
		font-size: 17px;
		line-height: 1.8;
		width: 50%;
		position: absolute;
		top: 0;
		right: 0;
		padding: 25rpx 0;
	}

	.canvas {
		z-index: 300;
		width: 750px;
		height: 1190px;
	}

	.poster-pop {
		width: 450rpx;
		height: 714rpx;
		position: fixed;
		left: 50%;
		transform: translateX(-50%);
		z-index: 399;
		top: 50%;
		margin-top: -357rpx;
	}

	.poster-pop image {
		width: 100%;
		height: 100%;
		display: block;
	}

	.poster-pop .close {
		width: 46rpx;
		height: 75rpx;
		position: fixed;
		right: 0;
		top: -73rpx;
		display: block;
	}

	.poster-pop .save-poster {
		background-color: #df2d0a;
		font-size: ：22rpx;
		color: #fff;
		text-align: center;
		height: 76rpx;
		line-height: 76rpx;
		width: 100%;
	}

	.poster-pop .keep {
		color: #fff;
		text-align: center;
		font-size: 25rpx;
		margin-top: 10rpx;
	}

	.mask {
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		background-color: rgba(0, 0, 0, 0.6);
		z-index: 9;
	}
	.navbar .header {
		height: 96rpx;
		font-size: 30rpx;
		color: #050505;
		background-color: #fff;
	}

	.home {
		/* #ifdef H5 */
		top: 20rpx !important;
		/* #endif */
	}

	.navbar .header .item {
		position: relative;
		margin: 0 25rpx;
	}

	.navbar .header .item.on:before {
		position: absolute;
		width: 60rpx;
		height: 5rpx;
		background-repeat: no-repeat;
		content: '';
		background-image: linear-gradient(to right, #ff3366 0%, #ff6533 100%);
		bottom: -10rpx;
		left: 50%;
		margin-left: -28rpx;
	}

	.navbar {
		position: fixed;
		background-color: #fff;
		top: 0;
		left: 0;
		z-index: 99;
		width: 100%;
	}

	.navbar .navbarH {
		position: relative;
	}

	.navbar .navbarH .navbarCon {
		position: absolute;
		bottom: 0;
		height: 100rpx;
		width: 100%;
		/* #ifndef APP-PLUS || H5 || MP-ALIPAY */
		justify-content: flex-end;
		/* #endif */
	}

	.home {
		color: #fff;
		position: fixed;
		width: 56rpx;
		height: 56rpx;
		z-index: 99;
		left: 33rpx;
		background: rgba(190, 190, 190, 0.5);
		border-radius: 50%;
		font-size: 33rpx;

		&.on {
			background: unset;
			color: #333;
		}
	}

	.home .line {
		width: 1rpx;
		height: 24rpx;
		background: rgba(255, 255, 255, 0.25);
	}

	.home .icon-xiangzuo {
		font-size: 28rpx;
	}

	.share-box {
		z-index: 1000;
		position: fixed;
		left: 0;
		top: 0;
		width: 100%;
		height: 100%;

		image {
			width: 100%;
			height: 100%;
		}
	}

	.product-con .conter {
		display: block;
	}

	.product-con .conter img {
		display: block;
	}

	.svip {
		height: 64rpx;
		padding: 0 26rpx 0 60rpx;
		margin: 24rpx 30rpx 0;
		background: url('https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/pages_activity_presell_details_index_0.png') center/100% 100% no-repeat;
		font-size: 24rpx;
		color: #ae5a2a;
	}

	.svip .iconfont {
		margin-left: 12rpx;
		font-size: 24rpx;
	}

	.product-con .wrapper .share .money .jvip {
		width: 46rpx;
		height: 22rpx;
	}

	.product-con .wrapper .share .money image {
		width: 66rpx;
		height: 26rpx;
	}

	.presell_count {
		margin-top: 20rpx;
		font-size: 26rpx;
		color: #999999;
		padding: 20rpx 30rpx;

		.presell_time {
			margin: 8rpx 0 4rpx;

			.area_line {
				display: inline-block;
				margin: 0 6rpx;
			}
		}

		.icon-shijian1 {
			display: inline-block;
			margin-right: 4rpx;
		}
	}

	.price_text {
		width: 90rpx;
		height: 30rpx;
		background-size: 100%;
		color: var(--view-bntColor);
		font-size: 22rpx;
		line-height: 30rpx;
		text-align: center;
		margin-left: 27rpx;
		padding: 4rpx 6rpx;
		border-radius: 4rpx;
		background-color: var(--view-op-ten);
	}

	.attrImg {
		width: 66rpx;
		height: 66rpx;
		border-radius: 6rpx;
		display: block;
		margin-right: 14rpx;
	}

	.switchTxt {
		height: 60rpx;
		flex: 1;
		line-height: 60rpx;
		box-sizing: border-box;
		background: #eeeeee;
		padding: 0 10rpx;
		border-radius: 8rpx;
		text-align: center;
	}

	.attribute {
		padding: 10rpx 30rpx;

		.line1 {
			width: 600rpx;
		}
	}

	.flex {
		display: flex;
		justify-content: space-between;
		width: 100%;
	}

	.flexs {
		display: flex;
	}

	.attr-txt {
		display: flex;
		flex-wrap: nowrap;
		width: 130rpx;
	}
	
	.delete-line {
		text-decoration: line-through;
	}
</style>
