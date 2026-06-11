<template>
	<view class="container">
		<view class="header">
			<view class="back-btn" @click="goBack">
				<text class="back-icon">←</text>
			</view>
			<text class="title">设置</text>
		</view>
		
		<view class="setting-item">
			<text class="setting-label">趣味模式</text>
			<switch class="setting-switch" :checked="funMode" @change="toggleFunMode" color="#ffffff" />
		</view>
		
		<view class="setting-desc">
			<text class="desc-text">开启后，点击会显示随机名称，木鱼也会变成趣味图片</text>
		</view>
		
		<view class="setting-item">
			<text class="setting-label">雨航模式</text>
			<switch class="setting-switch" :checked="yuhangMode" @change="toggleYuhangMode" color="#ffffff" />
		</view>
		
		<view class="setting-desc">
			<text class="desc-text">开启后，木鱼会变成雨航图片，点击有专属音效</text>
		</view>
		
		<view class="setting-item">
			<text class="setting-label">漫天爱心</text>
			<switch class="setting-switch" :checked="heartMode" @change="toggleHeartMode" color="#ffffff" />
		</view>
		
		<view class="setting-desc">
			<text class="desc-text">开启后，屏幕会自动浮现爱心动画</text>
		</view>
		
		<view class="setting-item" @click="goToCustom">
			<text class="setting-label">自定义木鱼</text>
			<text class="arrow-icon">›</text>
		</view>
		
		<view class="setting-desc">
			<text class="desc-text">选择自己的图片和文字</text>
		</view>
		
		<view class="setting-item" @click="goToOtherApps">
			<text class="setting-label">试试陆硕系列其他APP</text>
			<text class="arrow-icon">›</text>
		</view>
		
		<view class="setting-desc">
			<text class="desc-text">探索更多有趣的陆硕系列应用</text>
		</view>
		
		<view class="setting-item reset-btn" @click="resetApp">
			<text class="setting-label reset-text">重置APP</text>
			<text class="arrow-icon">›</text>
		</view>
		
		<view class="setting-desc">
			<text class="desc-text">清除所有数据，恢复到初始状态</text>
		</view>
		
		<view class="setting-item" @click="goToPrivacy">
			<text class="setting-label">隐私政策</text>
			<text class="arrow-icon">›</text>
		</view>
		
		<view class="setting-desc">
			<text class="desc-text">查看隐私政策与用户协议</text>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			funMode: false,
			yuhangMode: false,
			heartMode: false
		}
	},
	onLoad() {
		const savedFun = uni.getStorageSync('funMode')
		this.funMode = savedFun === 'true'
		const savedYuhang = uni.getStorageSync('yuhangMode')
		this.yuhangMode = savedYuhang === 'true'
		const savedHeart = uni.getStorageSync('heartMode')
		this.heartMode = savedHeart === 'true'
	},
	onShow() {
		// 每次显示时同步状态
		const savedFun = uni.getStorageSync('funMode')
		this.funMode = savedFun === 'true'
		const savedYuhang = uni.getStorageSync('yuhangMode')
		this.yuhangMode = savedYuhang === 'true'
		const savedHeart = uni.getStorageSync('heartMode')
		this.heartMode = savedHeart === 'true'
	},
	methods: {
		goBack() {
			uni.navigateBack()
		},
		toggleFunMode(e) {
			this.funMode = e.detail.value
			uni.setStorageSync('funMode', this.funMode.toString())
			if (this.funMode) {
				// 关闭其他模式（漫天爱心不受影响）
				this.yuhangMode = false
				uni.setStorageSync('yuhangMode', 'false')
				uni.setStorageSync('currentMuyuIndex', '-1')
			}
		},
		toggleYuhangMode(e) {
			this.yuhangMode = e.detail.value
			uni.setStorageSync('yuhangMode', this.yuhangMode.toString())
			if (this.yuhangMode) {
				// 关闭其他模式（漫天爱心不受影响）
				this.funMode = false
				uni.setStorageSync('funMode', 'false')
				uni.setStorageSync('currentMuyuIndex', '-1')
			}
		},
		toggleHeartMode(e) {
			this.heartMode = e.detail.value
			uni.setStorageSync('heartMode', this.heartMode.toString())
		},
		goToCustom() {
			uni.navigateTo({ url: '/pages/custom/custom' })
		},
		goToPrivacy() {
			uni.navigateTo({ url: '/pages/privacy/privacy' })
		},
		goToOtherApps() {
			const url = 'https://jolly-florentine-6cf020.netlify.app/'
			if (typeof plus !== 'undefined') {
				plus.runtime.openURL(url)
			} else if (typeof window !== 'undefined') {
				window.open(url, '_blank')
			} else {
				uni.setClipboardData({
					data: url,
					success: () => {
						uni.showToast({
							title: '链接已复制',
							icon: 'success'
						})
					}
				})
			}
		},
		resetApp() {
			uni.showModal({
				title: '重置APP',
				content: '确定要重置APP吗？这将清除所有数据，包括功德数、自定义木鱼等，并回到首次打开状态。',
				confirmText: '确定重置',
				cancelText: '取消',
				success: (res) => {
					if (res.confirm) {
						uni.clearStorageSync()
						uni.reLaunch({
							url: '/pages/index/index'
						})
					}
				}
			})
		}
	}
}
</script>

<style>
.container {
	width: 100%;
	height: 100vh;
	background-color: #000000;
	padding-top: 40rpx;
	position: fixed;
	top: 0;
	left: 0;
	overflow-y: auto;
	touch-action: none;
	user-select: none;
	overflow-x: hidden;
}

.header {
	display: flex;
	align-items: center;
	padding: 20rpx 30rpx;
	border-bottom: 1rpx solid rgba(255, 255, 255, 0.1);
}

.back-btn {
	width: 60rpx;
	height: 60rpx;
	display: flex;
	align-items: center;
	justify-content: center;
}

.back-icon {
	font-size: 40rpx;
	color: #ffffff;
}

.title {
	font-size: 34rpx;
	color: #ffffff;
	margin-left: 20rpx;
}

.setting-item {
	display: flex;
	align-items: center;
	justify-content: space-between;
	padding: 40rpx 30rpx;
	border-bottom: 1rpx solid rgba(255, 255, 255, 0.05);
}

.setting-label {
	font-size: 32rpx;
	color: #ffffff;
}

.setting-switch {
	transform: scale(1.2);
}

.arrow-icon {
	font-size: 40rpx;
	color: rgba(255, 255, 255, 0.5);
}

.setting-desc {
	padding: 0 30rpx 30rpx;
}

.desc-text {
	font-size: 26rpx;
	color: rgba(255, 255, 255, 0.5);
	line-height: 1.6;
}

.reset-text {
	color: #ff4d4f;
}

.reset-btn:active {
	background-color: rgba(255, 77, 79, 0.1);
}
</style>