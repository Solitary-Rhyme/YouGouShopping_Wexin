<template>
	<view class="my-settle-container">
		<label class="radio">
			<radio color="#C00000" :checked="isFullCheck" @click="changeAllState"/><text>全选</text>
		</label>
		
		<view class="amount-box">
			合计: <text class="amount">￥{{checkedGoodsAmount}}</text>
		</view>
		
		<view class="btn-settle" @click="settlement">结算({{checkedCount}})</view>
	</view>
</template>

<script>
	import {mapGetters, mapMutations, mapState} from 'vuex'
	
	export default {
		name:"my-settle",
		data() {
			return {
				seconds: 3,
				timer: null
			};
		},
		computed:{
			...mapState('m_cart',['cart']),
			...mapGetters('m_cart',['checkedCount','total','checkedGoodsAmount']),
			...mapState('m_user',['token']),
			...mapGetters('m_user',['addstr']),
			isFullCheck(){
				return this.total === this.checkedCount
			}
		},
		methods:{
			...mapMutations('m_cart',['updateAllGoodsState']),
			...mapMutations('m_user',['updateRedirectinfo']),
			changeAllState(){
				this.updateAllGoodsState(!this.isFullCheck)
			},
			settlement(){
				if(!this.checkedCount){
					return uni.$showMsg("请选择要结算的商品")
				}
				console.log(this.addstr)
				if(!this.addstr){
					return uni.$showMsg("请选择收货地址")
				}
				if(!this.token){
					return this.delayNavigate()
				}
				
				this.payOrder()
			},
			showTips(n){
				uni.showToast({
					icon: 'none',
					title: '请登录后再结算\n' + n + '秒后自动跳转到登录页',
					mask: 'true',
					duration: 1500
				})
			},
			delayNavigate(){
				this.seconds = 3
				this.showTips(this.seconds)
				
				this.timer = setInterval(() => {
					this.seconds--
					
					if(this.seconds <= 0){
						clearInterval(this.timer)
						uni.switchTab({
							url:'/pages/my/my',
							success: () => {
								this.updateRedirectinfo({
									openType: 'switchTab',
									from: '/pages/cart/cart'
								})
							}
						})
						return
					}
					
					this.showTips(this.seconds)
				},1000)
			},
			async payOrder(){
				const orderInfo = {
					order_price: 0.01,
					consignee_addr: this.addstr,
					goods: this.cart.filter(x => x.goods_state).map(x => ({
						goods_id: x.goods_id,
						goods_number:x.goods_count,
						goods_price: x.goods_price,
					}))
				}
				
				const {data: res} = await uni.$http.post('/api/public/v1/my/orders/create', orderInfo)
				// console.log(res);
				// console.log(this.token)
				if(res.meta.status !== 200) return uni.$showMsg('创建订单失败')
				const orderNumber = res.message.order_number
				
				const {data: res2} = await uni.$http.post('/api/public/v1/my/orders/req_unifiedorder',{order_number : orderNumber})
				console.log(res2);
				if(res2.meta.status !== 200) return uni.$showMsg('预付订单生成失败')
				const payInfo = res2.message.pay
				console.log(payInfo);
			}
		}
	}
</script>

<style lang="scss">
.my-settle-container{
	position: fixed;
	left: 0;
	bottom: 0;
	width: 100%;
	height: 50px;
	background-color: white;
	display: flex;
	justify-content: space-between;
	align-items: center;
	font-size: 14px;
	padding-left: 5px;
	
	.radio{
		display: flex;
		align-items: center;
	}
	
	.amount-box{
		.amount{
			color: #c00000;
			font-weight: bold;
		}
	}
	
	.btn-settle{
		background-color: #c00000;
		height: 50px;
		color: white;
		line-height: 50px;
		padding: 0 10px;
		min-width: 100px;
		text-align: center;
	}
}
</style>