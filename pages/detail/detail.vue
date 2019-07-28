<template>
	<view class="detail-contain">
		<view class="main-head">
			<view class="title">
					<text class="isTop" v-if="banner.top || banner.good">置顶</text><text>{{banner.title}}</text>
			</view>
			<view class="detail-mark-contian">
				<text class="detail-mark">· 发布于 {{banner.dayTime}} · 作者 {{banner.userName}} · {{banner.visit_count}}次浏览 · 来自 {{banner.article_type | formatType}}</text>
				<button size="mini" class="saveBtn" v-if="token && !saved" @click="collectTopic">收藏</button>	
				<button size="mini" class="unsaveBtn" v-if="token && saved" @click="uncollectTopic">取消</button>
			</view>
		</view>
		<view class="article-content" v-if="article.content">
			<u-parse :content="article.content" @preview="preview" @navigate="navigate" />
		</view>
		<view class="reply-count" v-if="article.content">{{article.reply_count}} 回复</view>
		<view class="comment-wrap"></view>
		<view class="article-content" v-if="article.content">
			<view  v-for="(item,index) in article.replies" :key='index'>
				<view class="reply-author">
					 <image :src="item.author.avatar_url"></image>
					 <view class="h60">{{item.author.loginname}}</view>
					 <view class="h60 light">{{index+1}}楼·{{item.last_reply_at || item.create_at |formatTime}}</view>
				</view>
				
				<u-parse :content="item.content" />
			</view>
		</view>
	</view>
</template>

<script>
	import uParse from '@/components/gaoyia-parse/parse.vue';
	import { friendlyDate } from '@/common/util.js';
	
	export default {
		data() {
			return {
				banner: {},
				content: '<p>获取信息失败</p>',
				article:{},
				saved:false,
				token:''
			}
		},
		components:{
			uParse
		},
		onShareAppMessage() {
			return {
				title: this.banner.title,
				path: '/pages/detail/detail?query=' + JSON.stringify(this.banner)
			}
		},
		onLoad(event) {
			// 目前在某些平台参数会被主动 decode，暂时这样处理。
			try {
				this.banner = JSON.parse(decodeURIComponent(event.query));
			} catch (error) {
				this.banner = JSON.parse(event.query);
			}
			this.token = uni.getStorageSync('CNode-token');
			this.getDetail();
		},
		methods: {
			getDetail() {
				uni.showLoading({
					title: '正在加载,请稍后'
				});
				uni.request({
					url: 'https://cnodejs.org/api/v1/topic/' + this.banner.id,
					success: (result) => {
						if(result.data.success){
							this.article = result.data.data;
							this.saved = this.article.is_collect;
							uni.hideLoading()
						} else{
							uni.showToast({
								icon:'none',
								title:'查询详情失败'
							})
							uni.hideLoading()
						}
					}
				});
			},
			  preview(src, e) {
			  console.log('preview')
			},
				navigate(href, e) {
				 plus.runtime.openURL(href);
			},
			// 收藏主题
			collectTopic(){
				uni.showLoading({
					title: '正在加载,请稍后'
				});
				uni.request({
					url: 'https://cnodejs.org/api/v1/topic_collect/collect',
					data:{
						accesstoken :this.token,
						topic_id :this.banner.id
					},
					method:'POST',
					success: (result) => {
						uni.hideLoading();
						if(result.data.success){
							this.saved = true;
							uni.showToast({
								title:'已收藏'
							})
						} else{
							uni.showToast({
								icon:'none',
								title:'收藏失败'
							})
						}
					}
				});
			},
			// 取消收藏
			uncollectTopic(){
				uni.showLoading({
					title: '正在加载,请稍后'
				});
			   uni.request({
					url: 'https://cnodejs.org/api/v1/topic_collect/de_collect',
					data:{
						accesstoken :this.token,
						topic_id :this.banner.id
					},
					method:'POST',
					success: (result) => {
						uni.hideLoading()
						if(result.data.success){
							this.saved = false;
						} 
					}
				});
			}
		},
		filters:{
			formatType(type){
				switch (type){
					case 'ask':
					   return '问答'
						
					case 'share':
					     return '分享'
						 
					case 'job':
					     return '招聘'
						 
					case 'good':
					     return '精品'
					default:
						return '问答'
				}
			},
			formatTime(time){
				if(time){
					return 	friendlyDate(new Date(time).getTime())
				}
				return ''
			}
		}
	}
</script>

<style lang="scss">
	@import url("../../components/gaoyia-parse/parse.css");
	page{
		background: #f7f7f7;
		overflow: hidden;
	}
	.detail-contain{
		background: #fff;
		border-radius: 3px;
		width: 680rpx;
		margin: 20rpx auto;
		padding: 30rpx 0rpx;
		.main-head{
			padding: 0 25rpx;
			border-bottom: 1px #e7e7e7 solid;
			padding-bottom: 60rpx;
			.title{
				font-size: 35rpx;
			}
			.detail-mark-contian{
				height: 40rpx;
				line-height: 40rpx;
				position: relative;
				margin-top: 20rpx;
				.detail-mark{
					color: #828282;
					font-size: 21rpx;
				}
				.saveBtn{
					background: #83C300;
					color: #fff;
					margin-left: 20rpx;
					position: absolute;
					right: 0;
				}
				.unsaveBtn{
					background: #b4b3b3;
					color: #fff;
					margin-left: 20rpx;
					position: absolute;
					right: 0;
				}
			}

		}
	}
	.isTop{
		width: 80rpx;
		height: 48rpx;
		line-height: 48rpx;
		text-align: center;
		background: #83c300;
		color: #FFFFFF;
		border-radius: 3px;
		margin-top: 10upx;
		margin-right: 25upx;
		font-size: 25rpx;
		padding: 10rpx;
	}
	.article-content {
		padding: 30upx;
		overflow: hidden;
		font-size: 30upx;
		padding-bottom: 30upx;
		.reply-author{
			height: 60rpx;
			line-height: 60rpx;
			display: flex;
			image{
				width: 60rpx;
				height: 60rpx;
			}
			.h60{
				height: 60rpx;
				line-height: 60rpx;
				padding-left: 10rpx;
				font-size: 25rpx;
			}
			.light{
				color: #304d5b;
			}
		}
	}
		.reply-count{
			height: 80rpx;
			background: #f7f7f7;
			line-height: 80rpx;
			color: #3c3c3c;
			font-size: 28rpx;
			padding-left: 20rpx;
		}
</style>
