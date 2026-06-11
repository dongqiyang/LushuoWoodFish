<template>
	<view class="container">
		<view class="merit-container" @longpress="openSettings">
			<text class="merit-number">{{ merit }}</text>
			<text class="merit-label">{{ displayLabel }}</text>
			<text v-if="!hasEnteredSettings" class="merit-hint">长按数字进入设置</text>
		</view>
		
		<view 
			class="fish-container" 
			:class="{ 'fish-clicked': isClicked }"
			@click="tapFish"
		>
			<view v-if="!currentMode" class="fish-body">
				<view class="fish-top"></view>
				<view class="fish-bottom"></view>
				<view class="fish-gap"></view>
			</view>
			<image v-else class="fun-image" :src="currentImage" mode="aspectFit"></image>
			<view class="fish-glow"></view>
		</view>
		
		<view class="tap-hint">
			<text class="hint-text">点击木鱼</text>
		</view>
		
		<view class="animation-container">
			<view 
				v-for="(anim, index) in addAnimations" 
				:key="anim.id" 
				class="add-animation"
				:style="{ left: anim.offsetX, top: anim.offsetY }"
			>{{ anim.text }}</view>
		</view>
		
		<view class="heart-container">
			<view 
				v-for="(heart, index) in hearts" 
				:key="heart.id" 
				class="heart-animation"
				:style="{ 
					left: heart.x + '%', 
					top: heart.y + '%',
					animationDuration: heart.duration + 's',
					animationDelay: heart.delay + 's',
					fontSize: heart.size + 'rpx'
				}"
			>♥</view>
		</view>
		
		<view v-if="showPrivacyModal" class="privacy-modal-mask" @click="rejectPrivacy">
			<view class="privacy-modal" @click.stop>
				<view class="privacy-modal-header">
					<text class="privacy-modal-title">隐私政策</text>
					<text class="privacy-modal-subtitle">陆硕木鱼尊重您的隐私</text>
				</view>
				<scroll-view scroll-y class="privacy-modal-content" style="width: 100%;">
					<text class="privacy-text">陆硕木鱼是一款纯本地运行的工具类应用，我们高度重视您的隐私保护。</text>
					<text class="privacy-text">我们不会主动收集、上传您的任何个人信息，您在应用内自定义的内容（图片、文字、音效等）将仅存储于您的设备本地。</text>
					<text class="privacy-text">我们不会申请相机、麦克风、定位、通讯录等非必要权限，仅在您主动授权后使用存储权限用于读写本地文件。</text>
					<text class="privacy-text">陆硕木鱼不含第三方SDK、广告组件或统计工具，不会接入任何外部服务收集用户信息。</text>
					<view class="privacy-link" @click="openPrivacyPage">
						<text>查看完整隐私政策</text>
					</view>
				</scroll-view>
				<view class="privacy-modal-footer">
					<view class="privacy-btn privacy-btn-cancel" @click="rejectPrivacy">
						<text>拒绝</text>
					</view>
					<view class="privacy-btn privacy-btn-confirm" @click="agreePrivacy">
						<text>同意并继续</text>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				merit: 0,
				isClicked: false,
				funMode: false,
				yuhangMode: false,
				customMode: false,
				heartMode: false,
				displayLabel: '功德',
				currentMode: null,
				currentImage: '',
				currentAudio: '',
				funNames: ['贱死了', '不需要', '无名', '构造的小麦'],
				yuhangNames: ['陆硕不需要', '不需要', '纯情男大', '可爱', '陆硕你好美', '雨航', 'so'],
				customNames: [],
				addAnimations: [],
				animationId: 0,
				hearts: [],
				heartId: 0,
				heartInterval: null,
				audioList: [],
				showPrivacyModal: false,
				hasEnteredSettings: false
			}
		},
		onLoad() {
			const saved = uni.getStorageSync('merit')
			if (saved) {
				this.merit = parseInt(saved)
			}
			this.hasEnteredSettings = uni.getStorageSync('hasEnteredSettings') === 'true'
			this.checkPrivacyAgreement()
			this.checkMode()
			this.checkHeartMode()
		},
		onShow() {
			this.checkMode()
			this.checkHeartMode()
		},
		onHide() {
			this.stopHeartAnimation()
		},
		onUnload() {
			this.stopHeartAnimation()
		},
		methods: {
			checkPrivacyAgreement() {
				const agreed = uni.getStorageSync('privacyAgreed')
				if (agreed !== 'true') {
					this.showPrivacyModal = true
				}
			},
			agreePrivacy() {
				uni.setStorageSync('privacyAgreed', 'true')
				this.showPrivacyModal = false
			},
			rejectPrivacy() {
				uni.showModal({
					title: '提示',
					content: '您必须同意隐私政策才能使用本应用',
					showCancel: false,
					confirmText: '我知道了',
					success: () => {
					}
				})
			},
			openPrivacyPage() {
				uni.navigateTo({
					url: '/pages/privacy/privacy'
				})
			},
			checkMode() {
				const muyuListStr = uni.getStorageSync('muyuList')
				const muyuIndexStr = uni.getStorageSync('currentMuyuIndex')
				const useFun = uni.getStorageSync('funMode') === 'true'
				const useYuhang = uni.getStorageSync('yuhangMode') === 'true'
				
				let muyuList = []
				let muyuIndex = -1
				
				if (muyuListStr) {
					muyuList = JSON.parse(muyuListStr)
				}
				if (muyuIndexStr !== '' && muyuIndexStr !== null && muyuIndexStr !== undefined) {
					muyuIndex = parseInt(muyuIndexStr)
				}
				
				if (muyuIndex >= 0 && muyuIndex < muyuList.length) {
					const selectedMuyu = muyuList[muyuIndex]
					this.customMode = true
					this.funMode = false
					this.yuhangMode = false
					this.currentMode = 'custom'
					this.currentImage = selectedMuyu.image
					this.currentAudio = selectedMuyu.audio || ''
					this.customNames = selectedMuyu.names
					this.displayLabel = this.getRandomCustomName() || '功德'
				} else if (useYuhang) {
					this.customMode = false
					this.funMode = false
					this.yuhangMode = true
					this.currentMode = 'yuhang'
					this.currentImage = '/static/yuhang.png'
					this.displayLabel = this.getRandomYuhangName()
				} else if (useFun) {
					this.customMode = false
					this.funMode = true
					this.yuhangMode = false
					this.currentMode = 'fun'
					this.currentImage = '/static/null_edit_61069105747431.png'
					this.displayLabel = this.getRandomName()
				} else {
					this.customMode = false
					this.funMode = false
					this.yuhangMode = false
					this.currentMode = null
					this.displayLabel = '功德'
				}
			},
			checkHeartMode() {
				const saved = uni.getStorageSync('heartMode')
				this.heartMode = saved === 'true'
				if (this.heartMode) {
					this.startHeartAnimation()
				} else {
					this.stopHeartAnimation()
				}
			},
			startHeartAnimation() {
				if (this.heartInterval) return
				this.heartInterval = setInterval(() => {
					this.createHeart()
				}, 800)
				for (let i = 0; i < 5; i++) {
					setTimeout(() => this.createHeart(), i * 150)
				}
			},
			stopHeartAnimation() {
				if (this.heartInterval) {
					clearInterval(this.heartInterval)
					this.heartInterval = null
				}
				this.hearts = []
			},
			createHeart() {
				const newHeart = {
					id: this.heartId++,
					x: Math.random() * 100,
					y: 30 + Math.random() * 70,
					size: 20 + Math.random() * 40,
					duration: 2.5 + Math.random() * 2.5,
					delay: 0
				}
				
				this.hearts.push(newHeart)
				
				if (this.hearts.length > 15) {
					this.hearts.shift()
				}
				
				setTimeout(() => {
					const index = this.hearts.findIndex(h => h.id === newHeart.id)
					if (index > -1) {
						this.hearts.splice(index, 1)
					}
				}, (newHeart.duration + newHeart.delay) * 1000)
			},
			getRandomName() {
				return this.funNames[Math.floor(Math.random() * this.funNames.length)]
			},
			getRandomYuhangName() {
				return this.yuhangNames[Math.floor(Math.random() * this.yuhangNames.length)]
			},
			getRandomCustomName() {
				if (this.customNames.length === 0) return null
				return this.customNames[Math.floor(Math.random() * this.customNames.length)]
			},
			playSound(src) {
				if (this.audioList.length >= 10) {
					const firstAudio = this.audioList.shift()
					if (firstAudio) {
						firstAudio.destroy()
					}
				}
				
				const audio = uni.createInnerAudioContext()
				audio.src = src
				audio.volume = 1
				
				audio.onEnded = () => {
					const index = this.audioList.indexOf(audio)
					if (index > -1) {
						this.audioList.splice(index, 1)
					}
					audio.destroy()
				}
				
				audio.onError = () => {
					const index = this.audioList.indexOf(audio)
					if (index > -1) {
						this.audioList.splice(index, 1)
					}
					audio.destroy()
				}
				
				this.audioList.push(audio)
				audio.play()
			},
			openSettings() {
			if (!this.hasEnteredSettings) {
				this.hasEnteredSettings = true
				uni.setStorageSync('hasEnteredSettings', 'true')
			}
			uni.navigateTo({
				url: '/pages/settings/settings'
			})
		},
			tapFish() {
				this.isClicked = true
				this.merit++
				
				uni.vibrateShort({ success: function() {} })
				
				if (this.funMode) {
					this.playSound('/static/ga.mp3')
				} else if (this.yuhangMode) {
					this.playSound('/static/so.mp3')
				} else if (this.customMode && this.currentAudio) {
					this.playSound(this.currentAudio)
				} else if (!this.currentMode) {
					this.playSound('/static/muyu.mp3')
				}
				
				uni.setStorageSync('merit', this.merit.toString())
				
				let text = '功德+1'
				if (this.customMode && this.customNames.length > 0) {
					text = this.getRandomCustomName()
					this.displayLabel = this.getRandomCustomName() || '功德'
				} else if (this.yuhangMode) {
					text = this.getRandomYuhangName()
					this.displayLabel = this.getRandomYuhangName()
				} else if (this.funMode) {
					text = this.getRandomName()
					this.displayLabel = this.getRandomName()
				}
				
				const randomX = (Math.random() - 0.5) * 200
				const randomY = (Math.random() - 0.5) * 80
				
				const newAnim = {
					id: this.animationId++,
					text: text,
					offsetX: `${50 + randomX / 30}%`,
					offsetY: `calc(30% + ${randomY}rpx)`
				}
				
				this.addAnimations.push(newAnim)
				
				if (this.addAnimations.length > 8) {
					this.addAnimations.shift()
				}
				
				setTimeout(() => {
					this.isClicked = false
				}, 100)
				
				setTimeout(() => {
					const index = this.addAnimations.findIndex(a => a.id === newAnim.id)
					if (index > -1) {
						this.addAnimations.splice(index, 1)
					}
				}, 850)
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
		align-items: center;
		justify-content: center;
		padding: 0;
		margin: 0;
		overflow: hidden;
		position: fixed;
		top: 40rpx;
		left: 0;
	}
	
	.merit-container {
		display: flex;
		flex-direction: column;
		align-items: center;
		position: absolute;
		top: calc(env(safe-area-inset-top) + 60rpx);
	}
	
	.merit-number {
		font-size: 96rpx;
		font-weight: 100;
		color: #ffffff;
		letter-spacing: 4rpx;
	}
	
	.merit-label {
		font-size: 28rpx;
		color: rgba(255, 255, 255, 0.6);
		letter-spacing: 20rpx;
		margin-top: 16rpx;
	}
	
	.merit-hint {
		font-size: 22rpx;
		color: rgba(255, 255, 255, 0.3);
		margin-top: 20rpx;
		letter-spacing: 4rpx;
	}
	
	.fish-container {
		position: relative;
		cursor: pointer;
		transition: transform 0.12s cubic-bezier(0.4, 0, 0.2, 1);
		margin-top: 260rpx;
	}
	
	.fish-container.fish-clicked {
		transform: scale(0.92);
	}
	
	.fish-body {
		position: relative;
		width: 380rpx;
		height: 280rpx;
		display: flex;
		flex-direction: column;
		align-items: center;
	}
	
	.fish-top {
		position: absolute;
		top: 0;
		width: 360rpx;
		height: 160rpx;
		background: linear-gradient(180deg, #ffffff 0%, #e8e8e8 100%);
		border-radius: 180rpx 180rpx 40rpx 40rpx;
		box-shadow: 
			0 -10rpx 30rpx rgba(255, 255, 255, 0.3),
			inset 0 10rpx 20rpx rgba(255, 255, 255, 0.6);
	}
	
	.fish-bottom {
		position: absolute;
		bottom: 0;
		width: 380rpx;
		height: 180rpx;
		background: linear-gradient(180deg, #f0f0f0 0%, #d8d8d8 100%);
		border-radius: 190rpx 190rpx 60rpx 60rpx;
		box-shadow: 
			0 10rpx 40rpx rgba(0, 0, 0, 0.3),
			inset 0 5rpx 15rpx rgba(255, 255, 255, 0.5);
	}
	
	.fish-gap {
		position: absolute;
		top: 110rpx;
		width: 320rpx;
		height: 20rpx;
		background-color: #000000;
		border-radius: 10rpx;
		transform: rotate(-5deg);
		opacity: 0.85;
	}
	
	.fun-image {
		width: 380rpx;
		height: 450rpx;
		border-radius: 20rpx;
	}
	
	.fish-glow {
		position: absolute;
		top: -20rpx;
		left: -20rpx;
		right: -20rpx;
		bottom: -20rpx;
		background: radial-gradient(ellipse at center, rgba(255, 255, 255, 0.15) 0%, transparent 70%);
		border-radius: 50%;
		pointer-events: none;
	}
	
	.tap-hint {
		margin-top: 80rpx;
	}
	
	.hint-text {
		font-size: 24rpx;
		color: rgba(255, 255, 255, 0.4);
		letter-spacing: 8rpx;
	}
	
	.animation-container {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		pointer-events: none;
		z-index: 100;
	}
	
	.add-animation {
		position: absolute;
		font-size: 56rpx;
		font-weight: 200;
		color: #ffffff;
		animation: floatUp 0.8s ease-in-out forwards;
		text-shadow: 0 0 20rpx rgba(255, 255, 255, 0.3);
		white-space: nowrap;
		transform: translateX(-50%);
	}
	
	@keyframes floatUp {
		0% {
			opacity: 0;
			transform: translateX(-50%) translateY(0rpx) scale(0.6);
		}
		20% {
			opacity: 1;
		}
		100% {
			opacity: 0;
			transform: translateX(-50%) translateY(-200rpx) scale(0.8);
		}
	}
	
	.heart-container {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		pointer-events: none;
		z-index: 99;
		overflow: hidden;
	}
	
	.heart-animation {
		position: absolute;
		color: rgba(255, 255, 255, 0.8);
		animation: heartFloat 2s ease-in-out forwards;
		text-shadow: 
			0 0 10rpx rgba(255, 255, 255, 0.8),
			0 0 20rpx rgba(255, 255, 255, 0.5),
			0 0 30rpx rgba(255, 255, 255, 0.3);
		opacity: 0;
	}
	
	@keyframes heartFloat {
		0% {
			opacity: 0;
			transform: translateY(0) scale(0.3) rotate(-10deg);
		}
		20% {
			opacity: 1;
			transform: translateY(-30rpx) scale(1) rotate(0deg);
		}
		50% {
			opacity: 0.8;
			transform: translateY(-100rpx) scale(0.9) rotate(10deg);
		}
		80% {
			opacity: 0.3;
		}
		100% {
			opacity: 0;
			transform: translateY(-250rpx) scale(0.5) rotate(-5deg);
		}
	}
	
	.privacy-modal-mask {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background-color: rgba(0, 0, 0, 0.85);
		display: flex;
		align-items: center;
		justify-content: center;
		z-index: 1000;
		animation: fadeIn 0.3s ease-out;
	}
	
	@keyframes fadeIn {
		from {
			opacity: 0;
		}
		to {
			opacity: 1;
		}
	}
	
	.privacy-modal {
		width: 90%;
		max-width: 640rpx;
		background: linear-gradient(145deg, #1a1a1a, #0d0d0d);
		border-radius: 32rpx;
		overflow: hidden;
		box-shadow: 
			0 20rpx 60rpx rgba(0, 0, 0, 0.5),
			0 0 40rpx rgba(255, 255, 255, 0.05),
			inset 0 1rpx 0 rgba(255, 255, 255, 0.1);
		animation: slideUp 0.35s cubic-bezier(0.34, 1.56, 0.64, 1);
	}
	
	@keyframes slideUp {
		from {
			opacity: 0;
			transform: translateY(40rpx) scale(0.95);
		}
		to {
			opacity: 1;
			transform: translateY(0) scale(1);
		}
	}
	
	.privacy-modal-header {
		padding: 48rpx 40rpx 32rpx;
		text-align: center;
		border-bottom: 1rpx solid rgba(255, 255, 255, 0.08);
	}
	
	.privacy-modal-title {
		font-size: 40rpx;
		font-weight: 600;
		color: #ffffff;
		display: block;
		margin-bottom: 12rpx;
		letter-spacing: 4rpx;
	}
	
	.privacy-modal-subtitle {
		font-size: 26rpx;
		color: rgba(255, 255, 255, 0.5);
		display: block;
	}
	
	.privacy-modal-content {
		max-height: 400rpx;
		padding: 32rpx 56rpx;
		box-sizing: border-box;
		width: 100%;
	}
	
	.privacy-text {
		font-size: 24rpx;
		color: rgba(255, 255, 255, 0.7);
		line-height: 1.8;
		display: block;
		margin-bottom: 20rpx;
		word-break: break-all;
		white-space: normal;
		overflow-wrap: break-word;
		text-align: justify;
	}
	
	.privacy-link {
		margin-top: 20rpx;
		margin-bottom: 40rpx;
		padding: 20rpx;
		background-color: rgba(255, 255, 255, 0.05);
		border-radius: 16rpx;
		text-align: center;
		transition: all 0.2s ease;
	}
	
	.privacy-link text {
		font-size: 26rpx;
		color: #4a9eff;
		text-decoration: underline;
	}
	
	.privacy-link:active {
		background-color: rgba(255, 255, 255, 0.1);
		transform: scale(0.98);
	}
	
	.privacy-modal-footer {
		display: flex;
		border-top: 1rpx solid rgba(255, 255, 255, 0.08);
	}
	
	.privacy-btn {
		flex: 1;
		height: 100rpx;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 28rpx;
		transition: all 0.2s ease;
	}
	
	.privacy-btn-cancel {
		color: rgba(255, 255, 255, 0.6);
		border-right: 1rpx solid rgba(255, 255, 255, 0.08);
	}
	
	.privacy-btn-cancel:active {
		background-color: rgba(255, 255, 255, 0.05);
	}
	
	.privacy-btn-confirm {
		color: #ffffff;
		background: linear-gradient(135deg, #4a4a4a, #3a3a3a);
	}
	
	.privacy-btn-confirm:active {
		background: linear-gradient(135deg, #3a3a3a, #2a2a2a);
		transform: scale(0.98);
	}
</style>