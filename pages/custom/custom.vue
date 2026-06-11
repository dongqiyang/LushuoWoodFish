<template>
	<view class="container">
		<view class="header">
			<view class="back-btn" @click="goBack">
				<text class="back-icon">←</text>
			</view>
			<text class="title">自定义木鱼</text>
			<view class="placeholder"></view>
		</view>

		<scroll-view scroll-y class="scroll-container">
			<view v-if="muyuList.length === 0" class="empty-state">
				<text class="empty-text">还没有自定义木鱼</text>
				<text class="empty-hint">点击右下角 + 添加</text>
			</view>

			<view class="muyu-list">
				<view v-for="(item, index) in muyuList" :key="index" class="muyu-item">
					<view class="muyu-info">
						<image v-if="item.image" class="muyu-image" :src="item.image" mode="aspectFit"></image>
						<view v-else class="muyu-placeholder">
							<text class="placeholder-icon">🎯</text>
						</view>
						<view class="muyu-detail">
							<text class="muyu-name">{{ item.name }}</text>
							<text class="muyu-names">{{ item.names.join(', ') }}</text>
						</view>
					</view>
					<view class="muyu-right">
						<switch
							class="muyu-switch"
							:checked="currentIndex === index"
							@change="toggleMuyu(index)"
							color="#ffffff"
						/>
						<text class="delete-btn" @click="deleteMuyu(index)">删除</text>
					</view>
				</view>
			</view>
		</scroll-view>

		<view class="add-btn" @click="goAddPage">
			<text class="add-icon">+</text>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				muyuList: [],
				currentIndex: -1
			}
		},
		onShow() {
			this.loadSettings()
		},
		methods: {
			loadSettings() {
				const savedList = uni.getStorageSync('muyuList')
				if (savedList) {
					this.muyuList = JSON.parse(savedList)
				}
				const savedIndex = uni.getStorageSync('currentMuyuIndex')
				if (savedIndex !== '' && savedIndex !== null && savedIndex !== undefined) {
					this.currentIndex = parseInt(savedIndex)
				}
			},
			saveSettings() {
				uni.setStorageSync('muyuList', JSON.stringify(this.muyuList))
				uni.setStorageSync('currentMuyuIndex', this.currentIndex.toString())
			},
			goBack() {
				this.saveSettings()
				uni.navigateBack()
			},
			goAddPage() {
				uni.navigateTo({
					url: '/pages/add-muyu/add-muyu'
				})
			},
			toggleMuyu(index) {
				if (this.currentIndex === index) {
					this.currentIndex = -1
				} else {
					this.currentIndex = index
					// 选择自定义木鱼时，关闭其他模式（漫天爱心不受影响）
					uni.setStorageSync('funMode', 'false')
					uni.setStorageSync('yuhangMode', 'false')
				}
				this.saveSettings()
			},
			deleteMuyu(index) {
				uni.showModal({
					title: '提示',
					content: '确定删除这个木鱼吗？',
					success: (res) => {
						if (res.confirm) {
							this.muyuList.splice(index, 1)
							if (this.currentIndex === index) {
								this.currentIndex = -1
							} else if (this.currentIndex > index) {
								this.currentIndex--
							}
							this.saveSettings()
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
		display: flex;
		flex-direction: column;
		position: relative;
	}

	.header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 60rpx 30rpx 20rpx 30rpx;
		background-color: #000000;
		border-bottom: 1rpx solid #222222;
		flex-shrink: 0;
	}

	.scroll-container {
		flex: 1;
		overflow-y: auto;
		padding-bottom: 120rpx;
	}

	.back-btn {
		width: 80rpx;
		height: 80rpx;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.back-icon {
		font-size: 44rpx;
		color: #ffffff;
	}

	.title {
		font-size: 34rpx;
		color: #ffffff;
		font-weight: 500;
	}

	.placeholder {
		width: 80rpx;
	}

	.empty-state {
		padding: 200rpx 30rpx;
		display: flex;
		flex-direction: column;
		align-items: center;
	}

	.empty-text {
		font-size: 32rpx;
		color: #666666;
		margin-bottom: 20rpx;
	}

	.empty-hint {
		font-size: 26rpx;
		color: #444444;
	}

	.muyu-list {
		padding: 20rpx 30rpx;
	}

	.muyu-item {
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 30rpx;
		background-color: #111111;
		border-radius: 20rpx;
		margin-bottom: 20rpx;
	}

	.muyu-info {
		display: flex;
		align-items: center;
		flex: 1;
	}

	.muyu-image {
		width: 120rpx;
		height: 120rpx;
		border-radius: 20rpx;
		background-color: #222222;
	}

	.muyu-placeholder {
		width: 120rpx;
		height: 120rpx;
		border-radius: 20rpx;
		background-color: #222222;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.placeholder-icon {
		font-size: 48rpx;
	}

	.muyu-detail {
		margin-left: 24rpx;
		flex: 1;
	}

	.muyu-name {
		font-size: 30rpx;
		color: #ffffff;
		font-weight: 500;
		display: block;
		margin-bottom: 10rpx;
	}

	.muyu-names {
		font-size: 24rpx;
		color: #888888;
		display: block;
	}

	.muyu-right {
		display: flex;
		align-items: center;
	}

	.muyu-switch {
		margin-right: 24rpx;
	}

	.delete-btn {
		font-size: 24rpx;
		color: #ff4444;
	}

	.add-btn {
		position: fixed;
		right: 40rpx;
		bottom: 80rpx;
		width: 120rpx;
		height: 120rpx;
		background-color: #ffffff;
		border-radius: 60rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		box-shadow: 0 4rpx 20rpx rgba(0, 0, 0, 0.3);
	}

	.add-icon {
		font-size: 64rpx;
		color: #000000;
		font-weight: 300;
		line-height: 1;
	}
</style>
