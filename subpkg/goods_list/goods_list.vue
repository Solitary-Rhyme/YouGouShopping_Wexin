<template>
	<view>
		<view class="goods-list">
			<view v-for="(goods,index) in goodsList" :key="index" @click="goToDetail(goods)">
				<my-goods :goods="goods"></my-goods>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				queryObj: {
					query: '',
					cid: '',
					pagenum: 1,
					pagesize: 10
				},
				goodsList: [],
				total: 0,
				isLoading: false
			};
		},
		onLoad(options) {
			this.queryObj.query = options.query || ''
			this.queryObj.cid = options.cid || ''

			this.getGoodsList()
		},
		methods: {
			async getGoodsList(cb) {
				
				this.isLoading = true
				
				const {data: res} = await uni.$http.get('/api/public/v1/goods/search', this.queryObj)
				
				this.isLoading = false
				
				cb && cb()
				
				console.log(res);
				//请求失败
				if (res.meta.status !== 200) {
					return uni.$showMsg()
				}
				//请求成功
				this.goodsList = [...this.goodsList, ...res.message.goods]
				this.total = res.message.total
			},
			goToDetail(goods){
				uni.navigateTo({
					url: '/subpkg/goods_detail/goods_detail?goods_id=' + goods.goods_id
				})
			}
		},
		onReachBottom() {
			
			if(this.queryObj.pagenum * this.queryObj.pagesize >= this.total) return uni.$showMsg('数据已加载完毕')
			
			if(this.isLoading) return
			
			this.queryObj.pagenum += 1
			this.getGoodsList()
		},
		onPullDownRefresh() {
			this.queryObj.pagenum = 1
			this.total = 0
			this.isLoading = false
			this.goodsList = []
			
			this.getGoodsList(() => uni.stopPullDownRefresh())
		}
	}
</script>

<style lang="scss">

</style>
