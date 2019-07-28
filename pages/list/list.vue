<template>
	<view class="uni-tab-bar">
		<scroll-view id="tab-bar" class="uni-swiper-tab" scroll-x :scroll-left="scrollLeft">
			<view v-for="(tab, index) in tabBars" :key="tab.ref" class="swiper-tab-list"
			 :id="tab.ref" :data-current="index" @click="tapTab(index)"><text :class="tabIndex==index ? 'active' : ''">{{tab.name}}</text></view>
		</scroll-view>
		<!-- #ifndef MP-BAIDU -->
		<scroll-view class="list" v-for="(tabItem, idx) in newsList" :key="tabItem.id" v-if="tabIndex === idx" scroll-y
		 @scrolltolower="loadMore(idx)">
			<block v-for="(newsItem, newsIndex) in tabItem.data" :key="newsIndex">
				<uni-media-list :options="newsItem" @click="goDetail(newsItem)"></uni-media-list>
			</block>
			<view class="uni-tab-bar-loading">
				<view class="loading-more">{{loadingText}}</view>
			</view>
		</scroll-view>
		<uni-drawer :visible="drawerVisibale" :mode='"right"' @close='drawerVisibale=false'>
	<view class="center">
			<view class="logo" @click="goLogin" :hover-class="!login ? 'logo-hover' : ''">
				<image class="logo-img" :src="login ? uerInfo.avatarUrl :avatarUrl"></image>
				<view class="logo-title">
					
					<text class="uer-name" v-if="login">{{ uerInfo.name }}</text>
					<text class="uer-name" v-else>未登录</text>
				</view>
			</view>
			<view class="center-list">
				<view class="center-list-item" @click="saoyisao">
					<uni-icon class='center-list-icon'  type="scan" size="18"></uni-icon>
					<text class="list-text" style="padding-left: 6px;">扫一扫</text>
				</view>
				<view class="center-list-item">
					<text class="list-icon">&#xe639;</text>
					<text class="list-text">新消息通知 </text>
				</view>
				<view class="center-list-item">
					<text class="list-icon">&#xe614;</text>
					<text class="list-text">关于应用</text>
				</view>
				<view class="center-list-item" @click="logout">
					<uni-icon class='center-list-icon' type="minus" size="20"></uni-icon>
					<text class="list-text" style="padding-left: 6px;">退出登录</text>
				</view>
			</view>
		</view>
		</uni-drawer>
		<!-- #endif -->
		<denglu v-if="showNamePop" @close='handlePopClose'></denglu>
	</view>
</template>
<script>
	import uniMediaList from '@/components/uni-media-list/uni-media-list.vue';
	import uniLoadMore from '@/components/uni-load-more/uni-load-more.vue';
	import uniDrawer from "@/components/uni-drawer/uni-drawer.vue";
	import uniIcon from "@/components/uni-icon/uni-icon.vue"
	import denglu from '@/components/denglu/denglu.vue';
	const tabTypeList = ['all','good','share','ask','job']
	import {
		friendlyDate
	} from '@/common/util.js';

	export default {
		components: {
			uniMediaList,
			uniLoadMore,
			uniDrawer,
			uniIcon,
			denglu
		},
		data() {
			return {
				loadingText: {
					contentdown: '上拉加载更多',
					contentrefresh: '正在加载...',
					contentnomore: '没有更多数据了'
				},
				scrollLeft: 0,
				refreshing: false,
				refreshText: '下拉可以刷新',
				newsList: [],
				tabIndex: 0,
				tabBars: [{
					name: '全部',
					id: 0,
					ref: 'new'
				}, {
					name: '精华',
					id: 23,
					ref: 'company'
				}, {
					name: '分享',
					id: 223,
					ref: 'content'
				}, {
					name: '问答',
					id: 221,
					ref: 'xiaofei'
				}, {
					name: '招聘',
					id: 225,
					ref: 'yule'
				}],
				pageSize:20,
				pullDown:false,
				drawerVisibale:false,
				login: false,
				avatarUrl: '/static/logo.png',
				uerInfo: {},
				accToken:'',
				showNamePop:false
			}
		},
		computed: {
			scrollViewHeight() {
				return 'height:' + (uni.getSystemInfoSync().windowHeight) + 'px';
			}
		},
		onLoad: function() {
			// 初始化列表信息
			this.tabBars.forEach((tabBar, index) => {
				this.newsList.push({
					id: 'tabBar' + index,
					data: [],
					requestParams: {
						tab: tabTypeList[index],
						limit : this.pageSize
					},
					loadingText: '加载中...'
				});
			});
			this.getList();
		},
		methods: {
			getList() {
				let activeTab = this.newsList[this.tabIndex];
				activeTab.requestParams.time = new Date().getTime() + '';
				if(this.pullDown){
					activeTab.requestParams.page =1 ;
				} else {
					activeTab.requestParams.page = parseInt(activeTab.data.length /  this.pageSize) +1;
				}
				
				this.loadingText = '加载中...';
				uni.showLoading({
					title: '正在加载,请稍后'
				});
				var self = this;
				// if(this.refreshing) return;
				uni.request({
					url: 'https://cnodejs.org/api/v1/topics',
					data: activeTab.requestParams,
					success: (result) => {
						uni.hideLoading()
						if (result.statusCode == 200) {
						const data = result.data.data.map((news) => {
								return {
									id: news.id,
									article_type: news.tab,
									datetime: news.last_reply_at,
									title: news.title,
									image_url: news.author.avatar_url,
									visit_count: news.visit_count,
									reply_count: news.reply_count,
									top:news.top,
									good:news.good,
									tabIndex:self.tabIndex,
									userName:news.author.loginname,
									dayTime: friendlyDate(new Date(news.create_at).getTime())
								};
							});
							//  下拉刷新
							if(self.pullDown){
								activeTab.data = data;
							}  else {
								data.forEach((news) => {
									activeTab.data.push(news);
								});
							}
							self.pullDown = false;
							self.refreshing = false;
							uni.stopPullDownRefresh()
						}
					}
				});
			},
			// 跳转详情
			goDetail(detail) {
				uni.navigateTo({
					url: '/pages/detail/detail?query=' + encodeURIComponent(JSON.stringify(detail))
				});
			},
			// 加载更多
			loadMore(idx) {
				if(!this.refreshing){
					this.refreshing = true;
					this.getList();
				}
			},
			// tab切换
			async changeTab(event) {
				let index = event.detail.current;
				if (this.isClickChange) {
					this.tabIndex = index;
					this.isClickChange = false;
					return;
				}
				let tabBar = await this.getElSize('tab-bar');
				let tabBarScrollLeft = tabBar.scrollLeft;
				let width = 0;

				for (let i = 0; i < index; i++) {
					let result = await this.getElSize(this.tabBars[i].ref);
					width += result.width;
				}
				let winWidth = uni.getSystemInfoSync().windowWidth,
					nowElement = await this.getElSize(this.tabBars[index].ref),
					nowWidth = nowElement.width;
				if (width + nowWidth - tabBarScrollLeft > winWidth) {
					this.scrollLeft = width + nowWidth - winWidth;
				}
				if (width < tabBarScrollLeft) {
					this.scrollLeft = width;
				}
				this.isClickChange = false;
				this.tabIndex = index;

				// 首次切换后加载数据
				const activeTab = this.newsList[this.tabIndex];
				if (activeTab.data.length === 0) {
					this.getList();
				}
			},
			getNodeSize(node) {
				return new Promise((resolve, reject) => {
					dom.getComponentRect(node, (result) => {
						resolve(result.size);
					});
				});
			},
			onRefresh(event) {
				this.refreshText = '正在刷新...';
				this.refreshing = true;
				this.getList();
			},
			getElSize(id) { //得到元素的size
				return new Promise((res, rej) => {
					uni.createSelectorQuery().select('#' + id).fields({
						size: true,
						scrollOffset: true
					}, (data) => {
						res(data);
					}).exec();
				});
			},
			async tapTab(index) { //点击tab-bar
				if (this.tabIndex === index) {
					return false;
				} else {
					this.tabIndex = index;
					// 首次切换后加载数据
					const activeTab = this.newsList[this.tabIndex];
					if (activeTab.data.length === 0) {
						this.getList();
					}
				}
			},
			// 扫码登录
			goLogin() {
				if (!this.login) {
					this.showNamePop = true;
				}
			},
			// 查询用户信息 
			queryUserInfo(){
			   uni.showLoading({
					title: '正在加载,请稍后'
				});
				 var username = uni.getStorageSync('CNode-username');
				 uni.request({
					url: 'https://cnodejs.org/api/v1/user/'+username,
					data: {
						accesstoken: this.accToken
					},
					success: (result) => {
						uni.hideLoading()
						 if(result.data.success){
							 let userData = result.data.data;
							 const uInfo ={
								 name:userData.loginname,
								 avatarUrl:userData.avatar_url
							 }
							 this.uerInfo = uInfo;
							 this.login = true;
							 uni.showToast({
							 	title:'登录成功！'
							 })
							 this.drawerVisibale = false;
							 uni.setStorageSync('CNode-userInfo',JSON.stringify(uInfo));
						 } else {
							 uni.showToast({
							 	icon:'none',
								title:'获取用户信息失败'
							 })
						 }
					 }
				});
			},
			handlePopClose(){
				this.showNamePop = false;
			},
			saoyisao(){
				var self = this;
				uni.scanCode({
					success: function (res) {
						if(res.result){
							var reg = /^((https|http|ftp|rtsp|mms){0,1}(:\/\/){0,1})www\.(([A-Za-z0-9-~]+)\.)+([A-Za-z0-9-~\/])+$/;
							if(reg.test(res.result)){
								plus.runtime.openURL(res.result);
							} else {
								self.accToken = res.result;
								uni.setStorageSync('CNode-token',res.result);
								self.queryUserInfo()
							}
	
						}
					}
				});
			},
			// 退出登录
			logout(){
				if(this.login){
				    uni.removeStorageSync('CNode-userInfo');
					uni.removeStorageSync('CNode-token');
					uni.removeStorageSync('CNode-username');
					this.login = false;
					this.uerInfo = {};
					this.drawerVisibale = false;
					uni.showToast({
						title:'操作成功！'
					})
				}
			}
		},
		mounted(){
			var userInfo = uni.getStorageSync('CNode-userInfo');
			if(userInfo){
				this.login  = true;
				this.uerInfo = JSON.parse(userInfo);
			} 
		},
		onPullDownRefresh() {
			// 初始化数据
			this.pullDown =  true;
			this.getList()
		},
		// 点击导航菜单
		onNavigationBarButtonTap(e) {
			this.drawerVisibale = true;
		},
	}
</script>
<style lang="scss">
	page {
		background-color: #FFFFFF;
		height: 100%;
		font-size: 11px;
		line-height: 1.8;
	}

	.uni-tab-bar {
		display: flex;
		flex: 1;
		flex-direction: column;
		overflow: hidden;
		height: 100%;
	}

	.uni-tab-bar .list {
		width: 750upx;
		height: calc(100% - 100upx);
		margin-top: 100upx;
	}

	.uni-swiper-tab {
		width: 100%;
		white-space: nowrap;
		line-height: 100upx;
		height: 100upx;
		position: fixed;
		background: #f7f7f7;
		z-index: 999;
		top: var(--window-top);
		left: 0;
	}

	.swiper-tab-list {
		font-size: 30upx;
		width: 150upx;
		display: inline-block;
		text-align: center;
		color: #90b23a;
		.active{
			padding: 10upx 15upx;
			background: #7cbf00;
			color: #FFFFFF;
			border-radius: 3px;
		}
	}

	.uni-tab-bar .swiper-box {
		flex: 1;
		width: 100%;
		height: calc(100% - 100upx);
		overflow: scroll;
	}

	.uni-tab-bar-loading {
		text-align: center;
		padding: 20upx 0;
		font-size: 14px;
		color: #CCCCCC;
	}
		@font-face {
		font-family: texticons;
		font-weight: normal;
		font-style: normal;
		src: url('https://at.alicdn.com/t/font_984210_5cs13ndgqsn.ttf') format('truetype');
	}
    .center {
		flex-direction: column;
	}
	
	.logo {
		// width: 750upx;
		height: 260upx;
		padding: 20upx  50upx;;
		box-sizing: border-box;
		flex-direction: row;
		align-items: center;
	}
	
	.logo-hover {
		opacity: 0.8;
	}
	
	.logo-img {
		width: 150upx;
		height: 150upx;
		border-radius: 150upx;
	}
	
	.logo-title {
		height: 150upx;
		flex: 1;
		align-items: center;
		justify-content: space-between;
		flex-direction: row;
		margin-left: 20upx;
	}
	
	.uer-name {
		height: 60upx;
		line-height: 60upx;
		font-size: 33upx;
		color: #555;
		
		// color: #FFFFFF;
	}
	
	.go-login.navigat-arrow {
		font-size: 38upx;
		color: #FFFFFF;
	}
	
	.login-title {
		height: 150upx;
		align-items: self-start;
		justify-content: center;
		flex-direction: column;
		margin-left: 20upx;
	}
	
	.center-list {
		background-color: #FFFFFF;
		margin-top: 20upx;
		width: 750upx;
		flex-direction: column;
	}
	
	.center-list-item {
		height: 90upx;
		width: 750upx;
		box-sizing: border-box;
		flex-direction: row;
		padding: 0upx 20upx;
		.center-list-icon{
			color: #555;
		    line-height:90upx;
		}
	}
	
	.border-bottom {
		border-bottom-width: 1upx;
		border-color: #c8c7cc;
		border-bottom-style: solid;
	}
	
	.list-icon {
		width: 40upx;
		height: 90upx;
		line-height: 90upx;
		font-size: 34upx;
		color: #4c4c4c;
		text-align: center;
		font-family: texticons;
		margin-right: 20upx;
	}
	// .uni-icon{
	// 	line-height: '';
	// }
	
	.list-text {
		height: 90upx;
		line-height: 90upx;
		font-size: 34upx;
		color: #555;
		flex: 1;
		text-align: left;
	}
	
	.navigat-arrow {
		height: 90upx;
		width: 40upx;
		line-height: 90upx;
		font-size: 34upx;
		color: #555;
		text-align: right;
		font-family: texticons;
		margin-left: 10upx;
	}
</style>
