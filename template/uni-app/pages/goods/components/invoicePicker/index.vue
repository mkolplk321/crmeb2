<template>
	<view>
		<view :class="{ mask: invShow }" @touchmove.stop.prevent @click="invClose"></view>
		<view class="popup" :class="{ on: invShow }">
			<view class="popup-hd">{{$t(`抬头选择`)}}<text class="iconfont icon-guanbi" @click="invClose"></text></view>
			<scroll-view class="popup-bd" scroll-y="true">
				<radio-group v-if="invList.length" name="inv" @change="invChange">
					<template v-for="(item, index) in invList">
						<label v-if="item.type === 1 || item.type === 2 && isSpecial" :key="item.id"
							class="acea-row row-middle item">
							<radio class="radio" :value="item.id" :checked="item.id === invChecked" />
							<view class="text">
								<view class="acea-row row-middle">
									<view class="name-wrap acea-row row-middle">
										<view class="name-group">
											<view class="name">{{item.name}}</view>
											<view v-if="item.is_default" class="default">{{$t(`默认`)}}</view>
										</view>
									</view>
									<view class="type" :class="{special: item.type === 2}">
										{{item.header_type === 1 ? $t(`个人`) : $t(`企业`)}}
										{{item.type === 1 ? $t(`普通`) : $t(`专用`)}}
									</view>
								</view>
								<view class="acea-row row-bottom">
									<view class="info-wrap">
										<view class="email">{{$t(`联系邮箱`)}} {{item.email}}</view>
										<view v-if="item.header_type === 1" class="tel">{{$t(`联系电话`)}}
											{{item.drawer_phone}}
										</view>
										<view v-else class="number">{{$t(`企业税号`)}}{{item.duty_number}}</view>
									</view>
									<navigator v-if="!isOrder" class="navigator"
										:url="`/pages/users/user_invoice_form/index?from=order_confirm&id=${item.id}&${urlQuery}`"
										hover-class="none"><text class="iconfont icon-bianji"></text>{{$t(`编辑`)}}
									</navigator>
									<navigator v-else class="navigator"
										:url="`/pages/users/user_invoice_form/index?from=order_details&id=${item.id}&order_id=${orderId}`"
										hover-class="none"><text class="iconfont icon-bianji"></text>{{$t(`编辑`)}}
									</navigator>
								</view>
							</view>
						</label>
					</template>
				</radio-group>
				<view v-else class="empty">
					<image :src="imgHost + '/statics/images/noInvoice.png'"></image>
					<view>{{$t(`您还没有添加发票信息哟`)}}~</view>
				</view>
			</scroll-view>
			<view class="popup-ft">
				<navigator v-if="!isOrder" class="navigator"
					:url="`/pages/users/user_invoice_form/index?from=order_confirm&${urlQuery}`" hover-class="none">
					<text class="iconfont icon-fapiao"></text>{{$t(`添加新的抬头`)}}
				</navigator>
				<navigator v-else class="navigator"
					:url="`/pages/users/user_invoice_form/index?order_id=${orderId}&from=order_details&${urlQuery}`"
					hover-class="none">
					<text class="iconfont icon-fapiao"></text>{{$t(`添加新的抬头`)}}
				</navigator>
				<button class="button" plain @click="invCancel">{{$t(`不开发票`)}}</button>
				<button class="button" plain @click="invSub">{{$t(`确认提交`)}}</button>
			</view>
		</view>
	</view>
</template>

<script>
	import {
		HTTP_REQUEST_URL
	} from '@/config/app';
	export default {
		data() {
			return {
				imgHost: HTTP_REQUEST_URL,
				invId: 0
			}
		},
		props: {
			invShow: {
				type: Boolean,
				default: false
			},
			invList: {
				type: Array,
				default () {
					return [];
				}
			},
			invChecked: {
				type: String,
				default: ''
			},
			isSpecial: {
				type: Boolean,
				default: false
			},
			urlQuery: {
				type: String,
				default: ''
			},
			isOrder: {
				type: Number,
				default: 0
			},
			orderId: {
				type: String,
				default: ''
			}
		},
		methods: {
			invClose(state) {
				this.$emit('inv-close');
			},
			invChange(e) {
				if (this.isOrder) {
					this.invId = e.detail.value
				} else {
					this.$emit('inv-change', e.detail.value);
				}
			},
			invSub() {
				this.$emit('inv-change', this.invId || this.invChecked);
			},
			invCancel() {
				this.$emit('inv-cancel');
			}
		},
	}
</script>

<style lang="scss" scoped>
	::v-deep uni-radio .uni-radio-input {
		margin-right: 0;
	}

	.popup {
		position: fixed;
		bottom: 0;
		left: 0;
		z-index: 9;
		width: 100%;
		border-top-left-radius: 16rpx;
		border-top-right-radius: 16rpx;
		background-color: #F5F5F5;
		transform: translateY(100%);
		transition: 0.3s;
	}

	.popup.on {
		transform: translateY(0);
	}

	.popup-hd {
		position: relative;
		height: 129rpx;
		font-size: 32rpx;
		line-height: 129rpx;
		text-align: center;
		color: #000000;

		.iconfont {
			position: absolute;
			top: 50%;
			right: 30rpx;
			transform: translateY(-50%);
			font-size: 32rpx;
			color: #707070;
		}
	}

	.popup-bd {
		height: 600rpx;
		padding-right: 30rpx;
		padding-left: 30rpx;
		box-sizing: border-box;

		.item {
			height: 194rpx;
			padding: 30rpx;
			margin-bottom: 14rpx;
			box-sizing: border-box;
			background: url("https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/pages_goods_components_invoicePicker_index_0.png") center/cover no-repeat;
		}

		.text {
			flex: 1;
			min-width: 0;
			margin-left: 30rpx;
		}

		.name-wrap {
			flex: 1;
			min-width: 0;
		}

		.name-group {
			display: flex;
			align-items: center;
			max-width: 100%;
		}

		.name {
			flex: 1;
			min-width: 0;
			overflow: hidden;
			white-space: nowrap;
			text-overflow: ellipsis;
			font-weight: bold;
			font-size: 30rpx;
			color: #282828;
			line-height: 28rpx;
		}

		.default {
			max-width: 100rpx;
			border: 1rpx solid var(--view-theme);
			margin-left: 20rpx;
			border-radius: 6rpx;
			padding: 0 4rpx;
			font-size: 20rpx;
			line-height: 28rpx;
			text-align: center;
			color: var(--view-theme);
		}

		.email {
			margin-top: 16rpx;
			font-size: 24rpx;
			color: #666666;
		}

		.tel {
			margin-top: 12rpx;
			font-size: 24rpx;
			color: #666666;
		}

		.number {
			margin-top: 12rpx;
			font-size: 24rpx;
			color: #666666;
		}

		.info-wrap {
			flex: 1;
		}

		.type {
			width: 162rpx;
			height: 42rpx;
			margin-left: 20rpx;
			background: url("https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/pages_goods_components_invoicePicker_index_1.png") center/cover no-repeat;
			font-size: 24rpx;
			line-height: 42rpx;
			text-align: center;
			color: #D67300;

			&.special {
				background-image: url("https://goyoto.oss-cn-beijing.aliyuncs.com/images/extracted/pages_goods_components_invoicePicker_index_2.png");
				color: #E93323;
			}
		}

		.navigator {
			margin-left: 20rpx;
			font-size: 26rpx;
			color: #282828;

			.iconfont {
				margin-right: 10rpx;
				font-size: 26rpx;
				color: #000000;
			}
		}
	}

	.popup-ft {
		padding: 14rpx 30rpx 44rpx;

		.navigator {
			height: 86rpx;
			border-radius: 43rpx;
			background-color: var(--view-theme);
			font-size: 30rpx;
			line-height: 86rpx;
			text-align: center;
			color: #FFFFFF;

			.iconfont {
				margin-right: 14rpx;
				font-size: 30rpx;
			}
		}

		.button {
			height: 86rpx;
			border: 1rpx solid var(--view-theme);
			border-radius: 43rpx;
			margin-top: 26rpx;
			font-size: 30rpx;
			line-height: 84rpx;
			color: var(--view-theme);
		}
	}

	.empty {
		padding-top: 58rpx;
		font-size: 26rpx;
		text-align: center;
		color: #999999;

		.image {
			width: 400rpx;
			height: 260rpx;
			margin-bottom: 20rpx;
		}
	}
</style>
