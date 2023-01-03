<template>
	<view>
		<view class="search-box">
			<uni-search-bar @confirm="search" @input="input" :radius="100" cancelButton="none"></uni-search-bar>
		</view>
		
		<view class="sugg-list" v-if="searchResults.length !== 0">
			<view class="sugg-item" v-for="(item,index) in searchResults" :key="index" @click="goToDetail(item)">
				<view class="goods-name">{{item.goods_name}}</view>
				<uni-icons type="arrowright" size="16"></uni-icons>
			</view>
		</view>
		
		<view class="history-box" v-else>
			<view class="history-title">
				<text>搜索历史</text>
				<uni-icons type="trash" size="17" @click="clean"></uni-icons>
			</view>
			<view class="history-list">
				<uni-tag :text="item" v-for="(item,index) in histories" :key="index" @click="goToGoodsList(item)"></uni-tag>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				timer: null,
				keyword: '',
				searchResults: [],
				historyList: ['apple','a','bbbb']
			};
		},
		methods:{
			input(e){
				clearTimeout(this.timer)
				this.timer = setTimeout(() => {
					this.keyword = e
					console.log(this.keyword)
					this.getSearchList()
				},500)
			},
			async getSearchList(){
				
				if(this.keyword === ''){
					this.searchResults = []
					return
				}
				
				const {data : res} = await uni.$http.get('/api/public/v1/goods/qsearch',{query: this.keyword})
				console.log(res);
				//请求失败
				if(res.meta.status !== 200){
					return uni.$showMsg()
				}
				//请求成功
				this.searchResults = res.message
				this.saveSearchHistory()
			},
			goToDetail(item){
				uni.navigateTo({
					url:'/subpkg/goods_detail/goods_detail?goods_id=' + item.goods_id
				})
			},
			saveSearchHistory(){
				// this.historyList.push(this.keyword)
				const set = new Set(this.historyList)
				set.delete(this.keyword)
				set.add(this.keyword)
				this.historyList = Array.from(set)
				
				uni.setStorageSync('keyword',JSON.stringify(this.historyList))
			},
			clean(){
				this.historyList = []
				uni.setStorageSync('keyword','[]')
			},
			goToGoodsList(keyword){
				uni.navigateTo({
					url:"/subpkg/goods_list/goods_list?query=" + keyword
				})
			}
		},
		computed:{
			histories(){
				return [...this.historyList].reverse()
			}
		},
		onLoad() {
			this.historyList = JSON.parse(uni.getStorageSync('keyword') || '[]')
		}
	}
</script>

<style lang="scss">
.search-box{
	position: sticky;
	top: 0;
	z-index: 999;
}

.sugg-list{
	padding: 0 5px;
	
	.sugg-item{
		font-size: 12px;
		padding: 13px 0;
		border-bottom: 1px solid #efefef;
		display: flex;
		text-align: center;
		justify-content: center;
		
		.goods-name{
			white-space: nowrap;
			overflow: hidden;
			text-overflow: ellipsis;
			margin-right: 3px;
		}
	}
}

.history-box{
	padding: 0 5px;
	
	.history-title{
		display: flex;
		justify-content: space-between;
		height: 40px;
		align-items: center;
		font-size: 13px;
		border-bottom: 1px solid #efefef;
	}
	
	.history-list{
		display: flex;
		flex-wrap: wrap;
		.uni-tag{
			margin-top: 5px;
			margin-right: 5px;
		}
	}
}
</style>
