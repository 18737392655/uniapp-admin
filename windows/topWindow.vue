<template>
	<view class="header">
		<!-- #ifdef H5 -->
		<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" style="position: absolute; width: 0; height: 0">
			<symbol xmlns="http://www.w3.org/2000/svg" viewBox="0 0 128 128" id="icon-bug">
				<path d="M127.88 73.143c0 1.412-.506 2.635-1.518 3.669-1.011 1.033-2.209 1.55-3.592 1.55h-17.887c0 9.296-1.783 17.178-5.35 23.645l16.609 17.044c1.011 1.034 1.517 2.257 1.517 3.67 0 1.412-.506 2.635-1.517 3.668-.958 1.033-2.155 1.55-3.593 1.55-1.438 0-2.635-.517-3.593-1.55l-15.811-16.063a15.49 15.49 0 0 1-1.196 1.06c-.532.434-1.65 1.208-3.353 2.322a50.104 50.104 0 0 1-5.192 2.974c-1.758.87-3.94 1.658-6.546 2.364-2.607.706-5.189 1.06-7.748 1.06V47.044H58.89v73.062c-2.716 0-5.417-.367-8.106-1.102-2.688-.734-5.003-1.631-6.945-2.692a66.769 66.769 0 0 1-5.268-3.179c-1.571-1.057-2.73-1.94-3.476-2.65L33.9 109.34l-14.611 16.877c-1.066 1.14-2.344 1.711-3.833 1.711-1.277 0-2.422-.434-3.434-1.304-1.012-.978-1.557-2.187-1.635-3.627-.079-1.44.333-2.705 1.236-3.794l16.129-18.51c-3.087-6.197-4.63-13.644-4.63-22.342H5.235c-1.383 0-2.58-.517-3.592-1.55S.125 74.545.125 73.132c0-1.412.506-2.635 1.518-3.668 1.012-1.034 2.21-1.55 3.592-1.55h17.887V43.939L9.308 29.833c-1.012-1.033-1.517-2.256-1.517-3.669 0-1.412.505-2.635 1.517-3.668 1.012-1.034 2.21-1.55 3.593-1.55s2.58.516 3.593 1.55l13.813 14.106h67.396l13.814-14.106c1.012-1.034 2.21-1.55 3.592-1.55 1.384 0 2.581.516 3.593 1.55 1.012 1.033 1.518 2.256 1.518 3.668 0 1.413-.506 2.636-1.518 3.67l-13.814 14.105v23.975h17.887c1.383 0 2.58.516 3.593 1.55 1.011 1.033 1.517 2.256 1.517 3.668l-.005.01zM89.552 26.175H38.448c0-7.23 2.489-13.386 7.466-18.469C50.892 2.623 56.92.082 64 .082c7.08 0 13.108 2.541 18.086 7.624 4.977 5.083 7.466 11.24 7.466 18.469z"></path>
			</symbol>
		</svg>
		<!-- #endif -->
		<view class="navbar" :class="{'navbar-mini':!matchLeftWindow,'popup-menu':popupMenuOpened}">
			<view class="navbar-left pointer">
				<navigator class="logo" open-type="reLaunch" url="/">
					<image :src="logo" mode="heightFix"></image>
					<text>{{appName}}</text>
				</navigator>
				<uni-icons @click="toggleSidebar" type="bars" class="menu-icon" size="30" color="#999"></uni-icons>
			</view>
			<view class="navbar-middle">
				<text class="title-text">{{navigationBarTitleText}}</text>
			</view>
			<view class="navbar-right pointer">
				<view v-show="userInfo.username" @click="togglePopupMenu" class="navbar-user">
					<view class="username"><text>{{userInfo.username}}</text></view>
					<uni-icons class="arrowdown" type="arrowdown" color="#666" size="13"></uni-icons>
				</view>
				<view class="uni-mask" @click="togglePopupMenu"></view>
				<view class="navbar-menu">
					<!-- #ifdef H5 -->
					<view v-if="logs.length" @click="showErrorLogs" class="menu-item debug pointer">
						<svg class="svg-icon">
							<use xlink:href="#icon-bug"></use>
						</svg>
						<uni-badge class="debug-badge" :text="logs.length" type="error"></uni-badge>
					</view>
					<!-- #endif -->
					<view v-for="link in links" :key="link.url" class="menu-item">
						<uni-link :href="link.url" :text="link.text" color="#666" fontSize="13" style="font-size:12px;" />
					</view>
					<template v-if="userInfo.username">
						<view class="menu-item username">
							<uni-icons class="person" type="person" color="#666" size="13"></uni-icons>
							<text>{{userInfo.username}}</text>
						</view>
						<view class="menu-item" @click="chagePassword">
							<text>修改密码</text>
						</view>
						
						<view class="menu-item ">
							<text class="logout pointer" @click="logout">退出</text>
						</view>
					</template>
					<view class="popup-menu__arrow"></view>
				</view>
			</view>
		</view>
		<uni-popup ref="errorLogsPopup" type="center">
			<view class="modal">
				<scroll-view class="modal-content" scroll-y="true">
					<error-log class="error-table" />
				</scroll-view>
			</view>
		</uni-popup>
		<uni-popup ref="passwordPopup" type="center">
			<view class="modal" style="width:400px; padding: 20px;">
				<update-password class="password-popup" :isPhone="true" v-on:closePasswordPopup="closePasswordPopup" />
			</view>
		</uni-popup>
	</view>
</template>

<script>
	import {
		mapMutations,
		mapState
	} from 'vuex'

	import errorLog from '@/windows/components/error-log.vue'
	import updatePassword from '@/windows/components/update-password.vue'
	import config from '@/admin.config.js'

	export default {
		components: {
			errorLog,
			updatePassword
		},
		props: {
			navigationBarTitleText: {
				type: String
			},
			matchLeftWindow: {
				type: Boolean
			},
			showLeftWindow: {
				type: Boolean
			}
		},
		data() {
			return {
				...config.navBar,
				popupMenuOpened: false,
				mpCapsule: 0
			}
		},
		computed: {
			...mapState('app', ['appName']),
			...mapState('user', ['userInfo']),
			...mapState('error', ['logs'])
		},
		mounted() {
			// #ifdef MP
			let menuButtonInfo = uni.getMenuButtonBoundingClientRect()
			this.mpCapsule = menuButtonInfo.width
			// console.log(111111111,this.mpCapsule)
			// #endif
		},
		methods: {
			...mapMutations({
				removeToken(commit) {
					commit('user/REMOVE_TOKEN')
				}
			}),
			showErrorLogs() {
				if (this.popupMenuOpened) {
					this.popupMenuOpened = false
				}
				this.$refs.errorLogsPopup.open()
			},
			showPasswordPopup() {
				if (this.popupMenuOpened) {
					this.popupMenuOpened = false
				}
				this.$refs.passwordPopup.open()
			},
			logout() {
				this.removeToken()
				uni.reLaunch({
					url: config.login.url
				})
			},
			toggleSidebar() {
				if (!this.showLeftWindow) {
					uni.showLeftWindow()
				} else {
					uni.hideLeftWindow()
				}
			},
			togglePopupMenu() {
				this.popupMenuOpened = !this.popupMenuOpened
			},
			closePasswordPopup() {
				this.$refs.passwordPopup.close()
			},
			toPasswordPage() {
				uni.navigateTo({
					url: '/pages/changepwd/changepwd'
				})
			},
			chagePassword() {
				!this.matchLeftWindow ? this.toPasswordPage() : this.showPasswordPopup()
			}
		}
	}
</script>

<style lang="scss">
	.header {
		height: 60px;
		width: 100%;
		box-sizing: border-box;
		border-bottom: 1px solid darken($top-window-bg-color, 8%);
		background-color: $top-window-bg-color;
		color: $top-window-text-color;
	}

	.navbar {
		font-size: 14px;
		position: relative;
		height: 100%;
		padding: 0 20px;
		display: flex;
		justify-content: space-between;
		align-items: center;
	}

	.logo {

		display: flex;
		align-items: center;

		image {
			height: 30px;
		}

		text {
			margin-left: 8px;
		}
	}

	.menu-icon {
		width: 30px;
		height: 30px;
		line-height: 30px;
	}

	.navbar-left,
	.navbar-middle,
	.navbar-right {
		flex: 1;
		/* #ifdef MP */
		margin-right: 97px;
		/* #endif */
	}

	.navbar-middle,
	.username {
		display: flex;
		align-items: center;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}

	.navbar-middle {
		text-align: center;
	}

	.username {
		max-width: 150px;
	}

	.title-text {
		font-size: 14px;
		line-height: 30px;
	}

	.navbar-menu {
		display: flex;
	}

	.menu-item {
		padding: 8px;
		font-size: 13px;
		color: #666;
	}

	.debug {
		display: inline-block;
		position: relative;
	}

	.debug-badge {
		position: absolute;
		top: 5px;
		right: 14px;
		transform: translateY(-50%) translateX(100%) scale(0.8);
	}

	.arrowdown {
		margin-top: 4px;
		margin-left: 3px;
	}

	.person {
		margin-top: 2px;
		margin-right: 2px;
	}

	.navbar-right {
		display: flex;
		justify-content: flex-end;
	}

	.navbar-right .uni-mask {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background-color: rgba(255, 255, 255, 0);
		z-index: 999;
	}

	.popup-menu__arrow {
		position: absolute;
		top: -6px;
		right: 20px;
		border-width: 6px;
		margin-right: 3px;
		border-top-width: 0;
		border-bottom-color: #ebeef5;
		filter: drop-shadow(0 2px 12px rgba(0, 0, 0, .03));
	}

	.popup-menu__arrow::after {
		content: " ";
		position: absolute;
		display: block;
		width: 0;
		height: 0;
		border-color: transparent;
		border-style: solid;
		border-width: 6px;
		top: 1px;
		margin-left: -6px;
		border-top-width: 0;
		border-bottom-color: #fff;
	}

	/* 大屏时，隐藏的内容 */
	.menu-icon,
	.navbar-middle,
	.navbar-user,
	.popup-menu__arrow,
	.navbar-right .uni-mask {
		display: none;
	}

	/* 小屏，显示的内容 */
	.navbar-mini .menu-icon,
	.navbar-mini .navbar-middle {
		display: block;
	}

	.navbar-mini .navbar-user {
		display: flex;
	}

	/* 小屏时，隐藏的内容 */
	.navbar-mini .logo,
	.navbar-mini .debug,
	.navbar-mini .navbar-menu,
	.navbar-mini .navbar-menu .username {
		display: none;
	}

	.navbar-mini .navbar-menu {
		flex-direction: column;
		align-items: center;
		justify-content: center;
		position: fixed;
		right: 20px;
		/* #ifdef MP */
		right: 97px;
		/* #endif */
		top: var(--window-top);
		/* #ifndef H5 */
		top: 85px;
		/* #endif */
		background-color: #fff;
		z-index: 999;
		padding: 5px 15px;
		margin: 5px 0;
		background-color: #fff;
		border: 1px solid #ebeef5;
		border-radius: 4px;
		box-shadow: 0 2px 12px 0 rgba(0, 0, 0, .1);
	}

	/* 小屏时，弹出下拉菜单 */
	.navbar-mini.popup-menu .navbar-menu {
		display: flex;
	}

	.navbar-mini.popup-menu .popup-menu__arrow,
	.navbar-mini.popup-menu .navbar-right .uni-mask {
		display: block;
	}

	.logout:hover {
		color: $menu-text-color-actived;
	}

	.svg-icon {
		width: 1em;
		height: 1em;
		vertical-align: -.15em;
		fill: currentColor;
		overflow: hidden;
	}

	.modal {
		width: 100%;
		max-width: 980px;
		margin: 0 auto;
		background-color: #FFFFFF;
	}

	.modal-content {
		padding: 15px;
		height: 500px;
		box-sizing: border-box;
	}

	.password-popup {
		padding: 30px;
	}
	
	
</style>
