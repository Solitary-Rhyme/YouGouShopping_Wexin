<template>
	<view>
		<my-search @click="goToSearch"></my-search>
		
		<view class="scroll-view-container">
			<scroll-view class="left-scroll-view" scroll-y="true" :style="{height: wh + 'px'}">
				<block v-for="(item,index) in cateList" :key="index">
					<view :class="['left-scroll-view-item', index === active ? 'active' : '']" @click="activeChanged(index)">{{item.cat_name}}</view>
				</block>
			</scroll-view>
			<scroll-view class="right-scroll-view" scroll-y="true" :style="{height: wh + 'px'}" :scroll-top="scrollTop">
				<view class="cate-lv2" v-for="(item2,index2) in cateLevel2" :key="index2">
					<view class="cate-lv2-title">
						/ {{item2.cat_name}} /
					</view>
					
					<view class="cate-lv3-list">
						<view class="cate-lv3-item" v-for="(item3,index3) in item2.children" :key="index3" @click="goToGoodsList(item3)">
							<image :src="item3.cat_icon.replace('dev','web')"></image>
							<text>{{item3.cat_name}}</text>
						</view>
					</view>
				</view>
			</scroll-view>
		</view>
	</view>
</template>

<script>
	import badgeMix from '@/mixins/tabbar-badge.js'
	
	export default {
		mixins: [badgeMix],
		data() {
			return {
				wh:0,
				cateList:[],
				cateLevel2:[],
				active:0,
				scrollTop:0
			};
		},
		onLoad() {
			const sysInfo = uni.getSystemInfoSync()
			this.wh = sysInfo.windowHeight - 50
			
			this.getCateList();
		},
		methods:{
			async getCateList(){
				const {data : res} = await uni.$http.get('/api/public/v1/categories')
				console.log(res);
				//请求失败
				if(res.meta.status !== 200){
					return uni.$showMsg()
				}
				//请求成功
				this.cateList = res.message
				this.cateLevel2 = res.message[0].children
			},
			activeChanged(index){
				this.active = index;
				this.cateLevel2 = this.cateList[index].children
				this.scrollTop = this.scrollTop === 0 ? 1 : 0
			},
			goToGoodsList(item3){
				uni.navigateTo({
					url:'/subpkg/goods_list/goods_list?cid=' + item3.cat_id
				})
			},
			goToSearch(){
				console.log("hi");
			}
		}
	}
</script>

<style lang="scss">
.scroll-view-container{
	display: flex;
	
	.left-scroll-view{
		width: 120px;
		
		.left-scroll-view-item{
			background-color: #f7f7f7;
			line-height: 60px;
			text-align: center;
			font-size: 12px;
			
			&.active{
				background-color: #FFFFFF;
				position: relative;
				
				&::before{
					content: '';
					display: block;
					width: 3px;
					height: 30px;
					background-color: #C00000;
					position: absolute;
					top: 50%;
					left: 0;
					transform: translateY(-50%);
				}
			}
		}
	}
	
	.cate-lv2-title{
		font-size: 12px;
		font-weight: bold;
		text-align: center;
		padding: 15px 0;
	}
	
	.cate-lv3-list{
		display: flex;
		flex-wrap: wrap;
		
		.cate-lv3-item{
			width: 33.33%;
			display: flex;
			flex-direction: column;
			justify-content: center;
			align-items: center;
			margin-bottom: 10px;
			
			image{
				height: 60px;
				width: 60px;
			}
			
			text{
				font-size: 12px;
			}
		}
	}
}
</style>
