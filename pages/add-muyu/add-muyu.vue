<template>
	<view class="container">
		<view class="header">
			<view class="back-btn" @click="goBack">
				<text class="back-icon">←</text>
			</view>
			<text class="title">添加自定义木鱼</text>
			<view class="placeholder"></view>
		</view>

		<view class="content">
			<view class="form-section">
				<text class="form-label">名称</text>
				<input
					class="form-input"
					v-model="newMuyu.name"
					placeholder="给木鱼起个名字"
					placeholder-class="input-placeholder"
				/>
			</view>

			<view class="form-section">
				<text class="form-label">图片</text>
				<view class="form-image-area">
					<view v-if="!newMuyu.image" class="form-upload-btn" @click="chooseImage">
						<text class="upload-icon">+</text>
						<text class="upload-text">从相册选择</text>
					</view>
					<view v-else class="form-preview-container">
						<image class="form-preview-image" :src="newMuyu.image" mode="aspectFit"></image>
						<text class="form-remove-btn" @click="removeTempImage">×</text>
					</view>
				</view>
			</view>

			<view class="form-section">
				<text class="form-label">显示文字</text>
				<input
					class="form-input"
					v-model="newMuyu.namesInput"
					placeholder="用逗号分隔，如:开心,快乐,发财"
					placeholder-class="input-placeholder"
				/>
				<text class="form-hint">用逗号分隔多个文字，点击时随机显示</text>
			</view>

			<view class="form-section">
				<text class="form-label">音频（选填）</text>
				<view class="form-audio-area">
					<view v-if="!newMuyu.audio" class="form-upload-btn" @click="chooseAudio">
						<text class="upload-icon">🎵</text>
						<text class="upload-text">选择音频（3秒内，MP3格式）</text>
					</view>
					<view v-else class="form-audio-preview">
						<text class="audio-name">{{ getAudioName(newMuyu.audio) }}</text>
						<text class="form-remove-btn" @click="removeTempAudio">×</text>
					</view>
				</view>
				<text class="form-hint">支持MP3格式，建议时长不超过3秒</text>
			</view>

			<view class="modal-actions">
				<view class="modal-btn cancel-btn" @click="goBack">
					<text class="btn-text">取消</text>
				</view>
				<view class="modal-btn save-btn-modal" @click="addMuyu">
					<text class="btn-text">添加</text>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				newMuyu: {
					name: '',
					image: '',
					namesInput: '',
					audio: ''
				}
			}
		},
		methods: {
			goBack() {
				uni.navigateBack()
			},
			chooseImage() {
				uni.chooseImage({
					count: 1,
					sizeType: ['compressed'],
					sourceType: ['album'],
					success: (res) => {
						this.newMuyu.image = res.tempFilePaths[0]
					}
				})
			},
			removeTempImage() {
				this.newMuyu.image = ''
			},
			chooseAudio() {
				const that = this
				uni.showActionSheet({
					itemList: ['从相册选择视频', '选择音频文件'],
					success: function(res) {
						if (res.tapIndex === 0) {
							that.chooseVideo()
						} else {
							that.chooseAudioFile()
						}
					},
					fail: function() {
						console.log('取消选择')
					}
				})
			},
			chooseVideo() {
				uni.chooseVideo({
					sourceType: ['album'],
					maxDuration: 3,
					success: function(res) {
						console.log('video selected:', res.tempFilePath)
						this.newMuyu.audio = res.tempFilePath
					}.bind(this),
					fail: function(err) {
						console.log('chooseVideo fail:', err)
					}
				})
			},
			chooseAudioFile() {
				if (typeof plus !== 'undefined' && plus.nativeUI && plus.nativeUI.filePicker) {
					plus.nativeUI.filePicker({
						title: '选择音频文件',
						mode: 'single',
						filter: ['mp3', 'wav', 'm4a']
					}, function(result) {
						if (result && result.files && result.files.length > 0) {
							this.handleAudioFile(result.files[0])
						}
					}.bind(this), function(err) {
						console.log('filePicker error:', err)
						uni.showToast({
							title: '无法访问文件',
							icon: 'none'
						})
					})
				} else if (typeof uni.chooseFile === 'function') {
					uni.chooseFile({
						count: 1,
						type: 'file',
						extension: ['.mp3', '.wav', '.m4a'],
						success: function(res) {
							this.handleAudioFile(res.tempFilePaths[0])
						}.bind(this),
						fail: function() {
						}
					})
				}
			},
			handleAudioFile(filePath) {
				if (!filePath) {
					return
				}
				
				const fileExt = filePath.toLowerCase().split('.').pop()
				if (fileExt !== 'mp3') {
					return
				}
				
				const audio = uni.createInnerAudioContext()
				audio.src = filePath
				audio.onLoadedMetadata = () => {
					if (audio.duration <= 3) {
						this.newMuyu.audio = filePath
					}
					audio.destroy()
				}
				audio.onError = (err) => {
					console.log('audio load error:', err)
					audio.destroy()
				}
				audio.load()
			},
			removeTempAudio() {
				this.newMuyu.audio = ''
			},
			getAudioName(path) {
				if (!path) return ''
				const parts = path.split('/')
				return parts[parts.length - 1] || 'audio.mp3'
			},
			addMuyu() {
				if (!this.newMuyu.name.trim()) {
					return
				}
				if (!this.newMuyu.image) {
					return
				}
				if (!this.newMuyu.namesInput.trim()) {
					return
				}

				const names = this.newMuyu.namesInput.split(/[,，]/).map(n => n.trim()).filter(n => n)
				if (names.length === 0) {
					return
				}

				// 保存图片为永久文件
				uni.saveFile({
					tempFilePath: this.newMuyu.image,
					success: (imgRes) => {
						const savedImagePath = imgRes.savedFilePath

						if (this.newMuyu.audio) {
							uni.saveFile({
								tempFilePath: this.newMuyu.audio,
								success: (audioRes) => {
									this.saveMuyuItem(savedImagePath, audioRes.savedFilePath, names)
								},
								fail: () => {
									this.saveMuyuItem(savedImagePath, '', names)
								}
							})
						} else {
							this.saveMuyuItem(savedImagePath, '', names)
						}
					}
				})
			},
			saveMuyuItem(imagePath, audioPath, names) {
				const newItem = {
					name: this.newMuyu.name.trim(),
					image: imagePath,
					names: names,
					audio: audioPath || ''
				}

				let muyuList = []
				const savedList = uni.getStorageSync('muyuList')
				if (savedList) {
					muyuList = JSON.parse(savedList)
				}
				muyuList.push(newItem)
				uni.setStorageSync('muyuList', JSON.stringify(muyuList))

				uni.navigateBack()
			}
		}
	}
</script>

<style>
	.container {
		min-height: 100vh;
		background-color: #000000;
	}

	.header {
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 60rpx 30rpx 20rpx 30rpx;
		background-color: #000000;
		border-bottom: 1rpx solid #222222;
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

	.content {
		padding: 40rpx 30rpx;
	}

	.form-section {
		margin-bottom: 40rpx;
	}

	.form-label {
		font-size: 28rpx;
		color: #888888;
		display: block;
		margin-bottom: 16rpx;
	}

	.form-input {
		width: 100%;
		height: 90rpx;
		background-color: #222222;
		border-radius: 16rpx;
		padding: 0 24rpx;
		font-size: 30rpx;
		color: #ffffff;
	}

	.input-placeholder {
		color: #555555;
	}

	.form-hint {
		font-size: 24rpx;
		color: #555555;
		display: block;
		margin-top: 12rpx;
	}

	.form-image-area {
		width: 100%;
	}

	.form-upload-btn {
		width: 100%;
		height: 260rpx;
		background-color: #222222;
		border-radius: 16rpx;
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
	}

	.upload-icon {
		font-size: 60rpx;
		color: #666666;
		margin-bottom: 10rpx;
	}

	.upload-text {
		font-size: 28rpx;
		color: #666666;
	}

	.form-preview-container {
		width: 100%;
		height: 260rpx;
		background-color: #222222;
		border-radius: 16rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		position: relative;
	}

	.form-preview-image {
		width: 200rpx;
		height: 200rpx;
		border-radius: 16rpx;
	}

	.form-audio-area {
		width: 100%;
	}

	.form-audio-preview {
		width: 100%;
		height: 90rpx;
		background-color: #222222;
		border-radius: 16rpx;
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding: 0 24rpx;
		position: relative;
	}

	.audio-name {
		font-size: 26rpx;
		color: #ffffff;
		flex: 1;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}

	.form-remove-btn {
		position: absolute;
		top: 16rpx;
		right: 16rpx;
		width: 56rpx;
		height: 56rpx;
		line-height: 52rpx;
		text-align: center;
		background-color: rgba(255, 68, 68, 0.8);
		border-radius: 28rpx;
		font-size: 36rpx;
		color: #ffffff;
	}

	.modal-actions {
		display: flex;
		justify-content: space-between;
		margin-top: 60rpx;
	}

	.modal-btn {
		flex: 1;
		height: 96rpx;
		border-radius: 48rpx;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.cancel-btn {
		background-color: #222222;
		margin-right: 20rpx;
	}

	.save-btn-modal {
		background-color: #ffffff;
	}

	.btn-text {
		font-size: 32rpx;
	}

	.cancel-btn .btn-text {
		color: #888888;
	}

	.save-btn-modal .btn-text {
		color: #000000;
	}
</style>
