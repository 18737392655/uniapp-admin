<template>
	<scroll-view class="sidebar" scroll-y="true">
		<view style="display: flex;height: 100%;overflow: hidden;">
			<view style="width: 26%;height: 100%;padding: 10px 0px;background: #304156;border-right: 3px solid rgb(64, 158, 255);">
				<uni-nav-menu :uniqueOpened="true"  :active="splitFullPath(active)" activeKey="url" textColor="#666" activeTextColor="#409eff" @select="select">
					<uni-menu-sidebar :isactivenum="isactivenum" @getChild1="getChild1" :data="staticMenu"></uni-menu-sidebar>
				</uni-nav-menu>
			</view>
			<view style="flex: 1;height: 100%;padding: 10px 0px;">
				<uni-nav-menu :clastyle="clastyle" :uniqueOpened="true" :active="splitFullPath(active)" activeKey="url" textColor="#666" activeTextColor="#409eff" @select="select">
					<uni-menu-sidebar :data="staticChildrenMenu"></uni-menu-sidebar>
				</uni-nav-menu>
			</view>
		</view>
		
		<!-- <uni-nav-menu :uniqueOpened="true" :active="splitFullPath(active)" activeKey="url" textColor="#666" activeTextColor="#409eff" @select="select">
			<uni-menu-sidebar :data="navMenu"></uni-menu-sidebar>
			<uni-menu-sidebar :data="staticMenu"></uni-menu-sidebar>
		</uni-nav-menu> -->
	</scroll-view>
</template>

<script>
	import {
		mapState,
		mapActions
	} from 'vuex'
	import config from '@/admin.config.js'
	export default {
		data() {
			return {
				...config.sideBar,
				defaultValue: '',
				staticChildrenMenu:[],
				current: '',
				isactivenum:0,
				clastyle: 'on',
				field: 'url as value, name as text, menu_id, parent_id, sort, icon, permission'
			}
		},
		computed: {
			...mapState('app', ['inited', 'navMenu', 'active']),
			...mapState('user', ['userInfo']),
			staticChildrenMenu(){
				return config.sideBar.staticMenu[0].children
			}
		},
		// #ifdef H5
		watch: {
			$route: {
				immediate: true,
				handler(newRoute, oldRoute) {
					if (newRoute.fullPath !== (oldRoute && oldRoute.fullPath)) {
						this.changeMenuActive(newRoute.fullPath)
					}
				}
			},
			// navMenu(val) {
			// 	if (val.length) return
			// 	let content
			// 	if (this.userInfo.role.indexOf('admin') !== -1) {
			// 		content = '菜单表中没有数据，请对 db_init.json 文件右键，初始化数据库'
			// 	} else {
			// 		content = '该用户未被授权访问任何菜单表中的菜单,请使用管理员账户为该用户赋权,可在权限管理、角色管理、菜单管理中操作,详见uniCloud admin文档'
			// 	}
			// 	setTimeout(() => {
			// 		uni.showModal({
			// 			title: '提示',
			// 			showCancel: false,
			// 			content: content
			// 		})
			// 	}, 16)
			// }
			userInfo: {
				immediate: true,
				handler(newVal, oldVal) {
					if (newVal) {
						this.$nextTick(function(){
							this.$refs.menu.load()
						})
					}
				}
			}
		},
		created() {
			this.staticChildrenMenu = config.sideBar.staticMenu[0].children
		},
		// #endif
		methods: {
			...mapActions({
				changeMenuActive: 'app/changeMenuActive'
			}),
			getChild1(e,_index){
				this.staticChildrenMenu = e;
				this.isactivenum = _index;
			},
			select(e) {
				let url = e.value
				if (!url) {
					url = this.active
				}
				this.clickMenuItem(url)
			},
			clickMenuItem(url) {
				// #ifdef H5
				if (url.indexOf('http') === 0) {
					return window.open(url)
				}
				// #endif

				// url 开头可用有 / ，也可没有
				if (url[0] !== '/' && url.indexOf('http') !== 0) {
					url = '/' + url
				}
				// TODO 后续要调整
				uni.redirectTo({
					url: url,
					fail: () => {
						uni.showModal({
							title: '提示',
							content: '页面 ' + url + ' 跳转失败',
							showCancel: false
						})
					}
				})
			},
			splitFullPath(path) {
				if (!path) path = '/'
				return path.split('?')[0]
			},
		}
	}
</script>

<style lang="scss">
	.sidebar {
		position: fixed;
		top: var(--top-window-height);
		width: 300px;
		height: calc(100vh - (var(--top-window-height)));
		box-sizing: border-box;
		border-right: 1px solid darken($left-window-bg-color, 8%);
		background-color: $left-window-bg-color;
		padding-bottom: 10px;
	}

	.title {
		margin-left: 5px;
	}
</style>
