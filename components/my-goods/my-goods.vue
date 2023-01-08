<template>
	<view class="goods-item">
		<view class="goods-item-left">
			<radio :checked="goods.goods_state" color="#C00000" v-if="showRadio" @click="radioChangeHandler"></radio>
			<image :src="goods.goods_small_logo || defaultPic" class="goods-pic"></image>
		</view>
	
		<view class="goods-item-right">
			<view class="goods-name">
				{{goods.goods_name}}
			</view>
			<view class="goods-info-box">
				<view class="goods-price">
					￥{{goods.goods_price | toFixed}}
				</view>
				<uni-number-box :min="1" :value="goods.goods_count" v-if="showNum" @change="numberChangeHandler"></uni-number-box>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		name:"my-goods",
		props:{
			goods:{
				type: Object,
				default: {}
			},
			showRadio:{
				type: Boolean,
				default: false
			},
			showNum:{
				type: Boolean,
				default: false
			}
		},
		data() {
			return {
				defaultPic: 'https://hbimg.b0.upaiyun.com/5d91b400204034583e9c24dd4b10f225f3e59522ed9-zXPxPI_fw658'
			};
		},
		filters:{
			toFixed(num){
				return Number(num).toFixed(2)
			}
		},
		methods:{
			radioChangeHandler(){
				this.$emit('radio-change',{
					goods_id: this.goods.goods_id,
					goods_state: !this.goods.goods_state
				})
			},
			numberChangeHandler(val){
				this.$emit('num-change',{
					goods_id: this.goods.goods_id,
					goods_count: +val
				})
			}
		}
	}
</script>

<style lang="scss">
	.goods-item {
		width: 750rpx;
		box-sizing: border-box;
		display: flex;
		padding: 10px 5px;
		border-bottom: 1px solid #f0f0f0;

		.goods-item-left {
			margin-right: 5px;
			display: flex;
			justify-content: center;
			align-items: center;

			.goods-pic {
				height: 100px;
				width: 100px;
				display: block;
			}
		}

		.goods-item-right {
			display: flex;
			flex: 1;
			flex-direction: column;
			justify-content: space-between;

			.goods-name {
				font-size: 13px;
			}

			.goods-info-box {
				display: flex;
				align-items: center;
				justify-content: space-between;
				
				.goods-price {
					color: #C00000;
					font-size: 16px;
				}
				
			}
		}
	}
</style>