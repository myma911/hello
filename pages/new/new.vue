<template>
	<view>
		<block v-for="item in list" :key="item.img_src">
			<view class="example-box">
				<uni-card :is-shadow="true" :title="item.title" mode="title" 
				thumbnail="https://img-cdn-qiniu.dcloud.net.cn/new-page/uni.png"
				extra="技术没有上限" note="true" @click="clickCard">
					<view>
						<view class="image-box">
							<image class="image" mode="aspectFill" :src="item.img_src" />
						</view>
						<view class="content-box">
							<text class="content-box-text">
								uni-app 是一个使用 Vue.js 开发所有前端应用的框架，开发者编写一套代码，可编译到iOS、Android、H5、以及各种小程序等多个平台。
								即使不跨端，uni-app同时也是更好的小程序开发框架。
							</text>
						</view>
					</view>
					<template slot="footer">
						<view class="footer-box">
							<view @click.stop="footerClick('喜欢')"><text class="footer-box__item"> 喜欢</text></view>
							<view @click.stop="footerClick('评论')"><text class="footer-box__item"> 评论</text></view>
							<view @click.stop="footerClick('收藏')"><text class="footer-box__item"> 收藏</text></view>
						</view>
					</template>
				</uni-card>
			</view>
		</block>
	</view>
</template>

<script>
	import uniCard from '@/components/uni-card/uni-card.vue'
	export default {
		components: {
			uniCard
		},
		data() {
			return {
				refreshing: false,
				list: [],
				fetchPageNum: 1
			}
		},
		onLoad() {
			this.getData();
		},
		onReachBottom() {
			console.log('滑动到页面底部')
			this.getData();
		},
		onPullDownRefresh() {
			console.log('下拉刷新');
			this.refreshing = true;
			this.getData();
		},
		methods: {
			getData() {
				uni.request({
					url: this.$serverUrl + '/api/picture/posts.php?page=' + (this.refreshing ? 1 : this.fetchPageNum) +
						'&per_page=5',
					success: (ret) => {
						console.log('data', ret);
						if (ret.statusCode !== 200) {
							console.log('失败!');
						} else {
							if (this.refreshing && ret.data[0].id === this.list[0].id) {
								uni.showToast({
									title: '已经最新',
									icon: 'none',
								})
								this.refreshing = false;
								uni.stopPullDownRefresh();
								return;
							}
							if (this.refreshing) {
								this.refreshing = false;
								uni.stopPullDownRefresh()
								this.list = ret.data;
								this.fetchPageNum = 2;
							} else {
								this.list = this.list.concat(ret.data);
								this.fetchPageNum += 1;
							}
						}
					}
				});
			},
			clickCard() {
				uni.showToast({
					title: '点击卡片',
					icon: 'none'
				})
			},
			footerClick(types) {
				uni.showToast({
					title: types,
					icon: 'none'
				})
			}
		}
	}
</script>

<style>
	/* 头条小程序组件内不能引入字体 */
	/* #ifdef MP-TOUTIAO */
	@font-face {
		font-family: uniicons;
		font-weight: normal;
		font-style: normal;
		src: url('~@/static/uni.ttf') format('truetype');
	}

	/* #endif */

	/* #ifndef APP-NVUE */
	page {
		display: flex;
		flex-direction: column;
		box-sizing: border-box;
		background-color: #efeff4;
		min-height: 100%;
		height: auto;
	}

	view {
		font-size: 28rpx;
		line-height: inherit;
	}

	.example {
		padding: 0 30rpx 30rpx;
	}

	.example-info {
		padding: 30rpx;
		color: #3b4144;
		background: #ffffff;
	}

	.example-body {
		flex-direction: row;
		flex-wrap: wrap;
		justify-content: center;
		padding: 0;
		font-size: 14rpx;
		background-color: #ffffff;
	}

	/* #endif */
	.example {
		padding: 0 30rpx;
	}

	.example-info {
		/* #ifndef APP-NVUE */
		display: block;
		/* #endif */
		padding: 30rpx;
		color: #3b4144;
		background-color: #ffffff;
		font-size: 30rpx;
	}

	.example-info-text {
		font-size: 28rpx;
		line-height: 36rpx;
	}


	.example-body {
		flex-direction: column;
		padding: 30rpx;
		background-color: #ffffff;
	}

	.word-btn-white {
		font-size: 18px;
		color: #FFFFFF;
	}

	.word-btn {
		/* #ifndef APP-NVUE */
		display: flex;
		/* #endif */
		flex-direction: row;
		align-items: center;
		justify-content: center;
		border-radius: 6px;
		height: 48px;
		margin: 15px;
		background-color: #007AFF;
	}

	.word-btn--hover {
		background-color: #4ca2ff;
	}


	.example-body {
		padding: 20rpx 0;
		padding-bottom: 0;

	}

	.example-box {
		/* margin-bottom: 30rpx;
	 */
	}

	.image-box {
		/* #ifndef APP-NVUE */
		display: flex;
		flex-direction: column;
		/* #endif */
		height: 350rpx;
		overflow: hidden;
	}

	.image {
		/* #ifndef APP-NVUE */
		width: 100%;
		height: 100%;
		/* #endif */
		flex: 1;
	}

	.content-box {
		padding-top: 20rpx;
	}

	.content-box-text {
		font-size: 30rpx;
	}

	.footer-box {
		/* #ifndef APP-NVUE */
		display: flex;
		/* #endif */
		justify-content: space-between;
		flex-direction: row;

	}

	.footer-box__item {
		align-items: center;
		padding: 10rpx 0;
		font-size: 30rpx;
		color: #666;
	}
</style>
