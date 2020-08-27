<template>
	<view>
		<carousel :img-list="imgList" url-key="url" @selected="selectedBanner" />
		
		
		
		<scroll-view scroll-top="0" scroll-y="true" class="scroll-Y" @scrolltoupper="upper" @scrolltolower="lower" @scroll="scroll" :style="{height:scrollHeight+'px'}">
			<uni-list >
					<!-- 自定义右侧内容 -->
					<uni-list-chat v-for="(item,index) in list" :key="index" title="辛晓琪-味道" 
						avatar="https://img-cdn-qiniu.dcloud.net.cn/new-page/uni.png" note="好听" :clickable="true" @click="play(item.id)">
						<view class="chat-custom-right"  @click.stop="favourite(item.id)">
							<text class="chat-custom-text">03：05</text>
							<uni-icons type="star-filled" color="#999" size="18"></uni-icons>
						</view>
					</uni-list-chat>
			</uni-list>
		</scroll-view>
		
		
		<imt-audio continue :src="audio[now].src" :duration="audio[now].duration" @prev="now = now === 0?audio.length-1:now-1"
		 @next="now = now === audio.length-1?0:now+1"></imt-audio>
		<!--
			 src: String 音频链接*必须*
			 duration: Number 总时长（单位：s）*必须*
			 autoplay: Boolean 是否自动播放*默认false*
			 control: Boolean 是否需要上一曲/下一曲按钮*默认true*
			 continue:Boolean 播放完成后是否继续播放下一首，需定义@next事件*默认false*
			 color: String 主色调*默认#169af3*
			 @prev:点击上一首按钮
			 @next:点击下一首按钮
		  -->
	</view>
	
</template>

<script>
	import carousel from '@/components/vear-carousel/vear-carousel'
	export default {
		components: {
			carousel
		},
		data() {
			return {
				forcedLogin: false, 
				scrollHeight:0, //元素的所需高度
				list: [],
				fetchPageNum: 1,
				imgList: [{
					url: 'https://img9.51tietu.net/pic/2019-091200/vgkpidei2tjvgkpidei2tj.jpg',
					id: 1
				}, {
					url: 'https://img9.51tietu.net/pic/2019-091200/euzekmi5m23euzekmi5m23.jpg',
					id: 2
				}, {
					url: 'https://img9.51tietu.net/pic/2019-091200/143tt0ta4sr143tt0ta4sr.jpg',
					id: 3
				}, {
					url: 'https://img9.51tietu.net/pic/2019-091200/ff1vqwm3q33ff1vqwm3q33.jpg',
					id: 4
				}, ],
				
				audio: [{
						src: 'http://49.232.194.201:9000/music/song/Josh%20Ritter%20-%20Darlin.mp3',
						duration: 212
					},
					{
						src: 'http://mouyizhan.com/2.mp3',
						duration: 189
					},
					{
						src: 'http://mouyizhan.com/3.mp3',
						duration: 214
					},
					{
						src: 'http://mouyizhan.com/4.mp3',
						duration: 205
					},
					{
						src: 'http://mouyizhan.com/5.mp3',
						duration: 228
					}
				],
				now: 0
			}
		},

		onLoad() {
			this.getMusicList();

		},
		
		onReady() {
			let _this = this;
			uni.getSystemInfo({ //调用uni-app接口获取屏幕高度
				success(res) { //成功回调函数
					let wHeight=res.windowHeight //windoHeight为窗口高度，主要使用的是这个
					let titleH=uni.createSelectorQuery().select(".scroll-Y"); //想要获取高度的元素名（class/id）
					titleH.boundingClientRect(data=>{
						_this.scrollHeight=wHeight-data.top-110 //计算高度：元素高度=窗口高度-元素距离顶部的距离（data.top）
					}).exec()
				}
			})
			uni.onWindowResize((res) => {
			    uni.getSystemInfo({ //调用uni-app接口获取屏幕高度
			    	success(res) { //成功回调函数
			    		let wHeight=res.windowHeight //windoHeight为窗口高度，主要使用的是这个
			    		let titleH=uni.createSelectorQuery().select(".scroll-Y"); //想要获取高度的元素名（class/id）
			    		titleH.boundingClientRect(data=>{
			    			_this.scrollHeight=wHeight-data.top-110 //计算高度：元素高度=窗口高度-元素距离顶部的距离（data.top）
			    		}).exec()
			    	}
			    })
			})
		},
		methods: {
			selectedBanner(item, index) {
				console.log('🥒', item, index)
			},
			
			// 点击播放列表中某条进行播放
			play($param){
				uni.showToast({
					title: '要开始播放'+$param
				});
			},
			
			// 点击收藏
			favourite($param){
				if (!localStorage.getItem(this.userLogin)) {
					this.guideToLogin()
				}else{
					uni.showToast({
						title: '收藏成功'
					});
				}
			},
			
			// 登录导航
			guideToLogin() {
				uni.showModal({
					title: '未登录',
					content: '您未登录，需要登录后才能继续',
					/**
					 * 如果需要强制登录，不显示取消按钮
					 */
					showCancel: !this.forcedLogin,
					success: (res) => {
						if (res.confirm) {
							/**
							 * 如果需要强制登录，使用reLaunch方式
							 */
							if (this.forcedLogin) {
								uni.reLaunch({
									url: '../login/login'
								});
							} else {
								uni.navigateTo({
									url: '../login/login'
								});
							}
						}
					}
				});
			},
			
			upper: function(e) {
				console.log(e)
			},
			lower: function(e) {
				console.log(e)
			},
			scroll: function(e) {
				console.log(e)
			},
			
			getMusicList() {
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
				})
			}
			
		}
	}
</script>

<style>
	.chat-custom-right {
		flex: 1;
		/* #ifndef APP-NVUE */
		display: flex;
		/* #endif */
		flex-direction: column;
		justify-content: space-between;
		align-items: flex-end;
	}

	.chat-custom-text {
		font-size: 12px;
		color: #999;
	}
	
	
	.scroll-Y {
		height: 300rpx;
	}
	
	
	
	
	
	
	
	
	
	
	
	
	
</style>
