<template>
	<view>
		<!-- 弹出层 -->
		<view class="uni-banner" style="background:#FFFFFF;">
			<view style="justify-content:flex-end;">
				<view style="justify-content:flex-end; text-align:right; padding:20upx;" @tap="closeBanner">
					<uni-icon type="close" size="22"/>
				</view>
			</view>
			  <view class="user-contain">
                <view class="label-title">用户名:</view>
                <input class="uni-input"  placeholder="用户名/github账号" v-model="username" />
				<view class="button-box" v-if="username">
					<button size="mini" class="btn"  @click="saveUserName">好了</button>
				</view>
            </view>
		</view>
		<view class="uni-mask"></view>
		<!-- 弹出层 -->
	</view>
</template>
<script>
    //保存登陆态
    var SESSION_KEY = 'denglutai'
	import uniIcon from "@/components/uni-icon/uni-icon.vue"
	export default {
		data() {
			return {
			   username:''
			}
		},
		components:{
			uniIcon
		},
		methods: {
			closeBanner: function() {
				this.$emit('close')
			},
			saveUserName(){
				uni.setStorageSync('CNode-username', this.username.replace(/\s+/g,""));
				this.closeBanner()
				uni.showToast({
					title:'保存成功！可通过扫一扫进行登录',
					icon:'none',
					duration:2000
				})
			}
		}
	}
</script>

<style lang="scss">
	@font-face {
		font-family: texticons;
		font-weight: normal;
		font-style: normal;
		src: url('https://at.alicdn.com/t/font_984210_5cs13ndgqsn.ttf') format('truetype');
	}
	/* 遮罩层 */
	.uni-mask {
		background: rgba(0, 0, 0, 0.6);
		position: fixed;
		width: 100%;
		height: 100%;
		left: 0;
		top: 0;
		z-index: 9998;
	}

	/* 弹出层形式的广告 */
	.uni-banner {
		width: 80%;
		position: fixed;
		left: 50%;
		top: 50%;
		background: #FFF;
		border-radius: 10upx;
		z-index: 9999;
		transform: translate(-50%, -50%);
	}
	.user-contain{
		padding-left: 100rpx;
		width: 420rpx;
		padding-bottom: 100rpx;
		font-size: 25rpx;
		.label-title{
			padding: 20rpx 0;
		}
		.uni-input{
			border:1px #ededed solid;
			padding: 10rpx 20rpx;
			border-radius: 5px;
		}
		.button-box{
			padding-top: 30rpx;
			position: relative;
			height: 50rpx;
			.btn{
				color: #fff;
				background: #83c300;
				text-align: left;
				position: absolute;
				left: 0;
			}
		}
	}
</style>
