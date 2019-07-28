<template>
	<view>
		<view class="list-cell" hover-class="uni-list-cell-hover" @click="bindClick">
			<view class="media-list" v-if="options.title">
				<view class="media-image-left">
					<view class="media-title">
						<text class="font-title">{{options.title}}</text>
						<view class="media-foot">
							<view class="media-info">
								<text class="info-text">{{options.reply_count}} / {{options.visit_count}}</text>
							</view>
							<view class="max-close-view">
								<view>{{options.dayTime}}</view>
							</view>
						</view>
					</view>
					<view v-if="options.image_list || options.image_url" class="image-section image-section-left" >
						<image class="image-list1" 
						 v-if="options.image_url" :src="options.image_url" :alt='options.userName'></image>
						 <text v-if="options.top" class="isTop">置顶</text>
						 <text v-if="!options.top && !options.good && options.tabIndex==0 " class="isNtop">{{options.article_type | formatType}}</text>
						 <text v-if="options.good && options.tabIndex!=1 " class="isTop">精华</text>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		props: {
			options: {
				type: Object,
				default: function(e) {
					return {}
				}
			}
		},
		methods: {
			bindClick() {
				this.$emit('click');
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
			}
		}
	}
</script>

<style lang="scss">
	view {
		display: flex;
		flex-direction: column;
		box-sizing: border-box;
	}

	.list-cell {
		width: 750upx;
		padding: 0 30upx;
	}

	.uni-list-cell-hover {
		background-color: #eeeeee;
	}

	.media-list {
		flex: 1;
		flex-direction: column;
		border-bottom-width: 1upx;
		border-bottom-style: solid;
		border-bottom-color: #f2f2f2;
		padding: 20upx 0;
		height: 130upx;
	}

	.media-image-right {
		flex-direction: row;
	}

	.media-image-left {
		flex-direction: row-reverse;
	}

	.media-title {
		flex: 1;
		overflow: hidden;
		padding-left:10upx;
		.font-title{
			height: 70upx;
			overflow: hidden;
			text-overflow: ellipsis;
			white-space: nowrap;
			font-size: 32upx;
			padding-left: 5upx;
		}
	}

	.media-title2 {
		flex: 1;
	/* 	margin-top: 6upx; */
		line-height: 70upx;
		
	}

	.image-section {
		margin-top: 20upx;
		flex-direction: row;
		justify-content: space-between;
	}

	.image-section-right {
		margin-top: 0upx;
		margin-left: 10upx;
		width: 225upx;
		height: 146upx;
	}

	.image-section-left {
		margin-top: 0upx;
		margin-right: 10upx;
		height: 146upx;
	}

	.image-list1 {
		width: 80upx;
		height: 80upx;
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
		margin-left: 25upx;
	}
	.isNtop{
		width: 80rpx;
		height: 48rpx;
		line-height: 48rpx;
		text-align: center;
		background: #e3e3e3;
		color: #a0a0a0;
		border-radius: 3px;
		margin-top: 10upx;
		margin-left: 25upx;
	}

	.image-list3 {
		width: 225upx;
		height: 146upx;
	}

	.media-info {
		flex-direction: row;
	}

	.info-text {
		margin-right: 20upx;
		color: #999999;
		font-size: 24upx;
	}

	.media-foot {
		flex-direction: row;
		justify-content: space-between;
	}

	.max-close-view {
		align-items: center;
		justify-content: flex-end;
		flex-direction: row;
		height: 40upx;
		width: 120upx;
		color: #737373;
		font-size: 24upx;
	}
</style>
