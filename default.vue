<template>
	<div>
		<!-- 顶部导航栏 -->
		<div v-if="!$store.state.canvas.data.locked" class="headers">
			<el-menu mode="horizontal" @select="onMenu" background-color="#f8f8f8">
				<el-menu-item class="logo">
					<nuxt-link to="/">
						<img src="/favicon.ico" />
					</nuxt-link>
				</el-menu-item>
				<el-submenu index="file">
					<template slot="title">文件</template>
					<el-menu-item index="new">新建文件</el-menu-item>
					<el-menu-item index="open">打开本地文件（新建）</el-menu-item>
					<el-menu-item index="replace">导入本地文件...</el-menu-item>
					<el-menu-item class="separator"></el-menu-item>
					<el-menu-item index="save">保存到本地</el-menu-item>
					<el-menu-item index="savePng">下载为PNG</el-menu-item>
					<!-- <el-menu-item index="saveSvg">下载为SVG</el-menu-item> -->
				</el-submenu>
				<el-submenu index="edit">
					<template slot="title">编辑</template>
					<el-menu-item index="undo">撤消</el-menu-item>
					<el-menu-item index="redo">重做</el-menu-item>
					<el-menu-item class="separator"></el-menu-item>
					<el-menu-item index="copy">复制</el-menu-item>
					<el-menu-item index="cut">剪切</el-menu-item>
					<el-menu-item index="parse">粘贴</el-menu-item>
				</el-submenu>
			</el-menu>
			<el-menu mode="horizontal" class="full" background-color="#f8f8f8"></el-menu>
			<el-menu mode="horizontal" background-color="#f8f8f8">
				<el-menu-item>视图：{{scale}}%</el-menu-item>
				<el-submenu index="state" title="默认连线类型">
					<template slot="title">
						<i :class="`iconfont icon-${lineName}`"></i>
					</template>
					<el-menu-item v-for="(item, index) in lineNames" :key="index" :index="`line-${item}`" @click="onState('lineName', item)">
						<i :class="`iconfont icon-${item}`"></i>
					</el-menu-item>
				</el-submenu>
			</el-menu>
			<el-menu mode="horizontal" background-color="#f8f8f8">
				<el-submenu index="state" title="默认起点箭头">
					<template slot="title">
						<i :class="`iconfont icon-from-${fromArrowType}`"></i>
					</template>
					<el-menu-item v-for="(item, index) in arrowTypes" :key="index" :index="`fromArrow-${item}`" @click="onState('fromArrowType', item)">
						<i :class="`iconfont icon-from-${item}`"></i>
					</el-menu-item>
				</el-submenu>
			</el-menu>
			<el-menu mode="horizontal" background-color="#f8f8f8">
				<el-submenu index="state" title="默认终点箭头">
					<template slot="title">
						<i :class="`iconfont icon-to-${toArrowType}`"></i>
					</template>
					<el-menu-item v-for="(item, index) in arrowTypes" :key="index" :index="`toArrow-${item}`" @click="onState('toArrowType', item)">
						<i :class="`iconfont icon-to-${item}`"></i>
					</el-menu-item>
				</el-submenu>
			</el-menu>
			<!--  <el-menu mode="horizontal" background-color="#f8f8f8">
        <el-submenu index="user" v-if="user">
          <template slot="title">
            <el-avatar
              src="https://cube.elemecdn.com/0/88/03b0d39583f48206768a7534e55bcpng.png"
              :size="24"
            ></el-avatar>
            {{user.username}}
          </template>
          <el-menu-item @click="onSignOut">退出</el-menu-item>
        </el-submenu>
        <el-menu-item v-if="!user">
          <a @click="onLogin">注册 / 登录</a>
        </el-menu-item>
      </el-menu> -->
		</div>
		<!-- body部分 -->
		<div class="body" :class="{
			big: $store.state.canvas.data.locked,
		}">
			<nuxt />
		</div>
	</div>
</template>
<script>
import '~/assets/css/base.scss'

export default {
	data() {
		return {
			about: false,
			license: false,
			joinin: false,
			lineNames: ['curve', 'polyline', 'line'],
			arrowTypes: [
				'',
				'triangleSolid',
				'triangle',
				'diamondSolid',
				'diamond',
				'circleSolid',
				'circle',
				'line',
				'lineUp',
				'lineDown'
			],
			user: null
		}
	},
	computed: {
		scale() {
			return Math.floor(this.$store.state.canvas.data.scale * 100)
		},
		lineName() {
			return this.$store.state.canvas.data.lineName
		},
		fromArrowType() {
			return this.$store.state.canvas.data.fromArrowType
		},
		toArrowType() {
			return this.$store.state.canvas.data.toArrowType
		},
		error() {
			return this.$store.state.notice.error
		}
	},
	watch: {
		error(curVal) {
			this.$notify({
				title: '错误',
				type: 'error',
				message: curVal.text
			})
		}
	},
	methods: {
		onMenu(key, keyPath) {
			if (!key || key.indexOf('/') === 0) {
				return
			}

			switch (key) {
				case 'new':
					this.$router.push('/')
					break
				case 'open':
					this.$router.push('/')
					setTimeout(() => {
						this.$store.commit('event/emit', {
							name: key
						})
					}, 100)
					break
				case 'about':
				case 'about2':
					this.about = true
					break
				case 'license':
					this.license = true
					break
				case 'joinin':
					this.joinin = true
					break
				default:
					this.$store.commit('event/emit', {
						name: key
					})
					break
			}
		},
		onState(key, value) {
			this.$store.commit('event/emit', {
				name: 'state',
				data: {
					key,
					value
				}
			})
		},
		async getUser() {
			if (this.$cookies.get('token')) {
				this.user = await this.$axios.$get('/api/user/profile')
			}
		},
		onLogin() {
			if (process.client) {
				location.href = `https://account.le5le.com?cb=${encodeURIComponent(
          location.href
        )}`
			}
		},
		onSignOut() {
			this.$cookies.remove('token')
			this.user = null
		}
	},
	created() {
		this.getUser()
	}
}

</script>
<style lang="scss">
html {
	font-family: 'Source Sans Pro', -apple-system, BlinkMacSystemFont, 'Segoe UI',
		Roboto, 'Helvetica Neue', Arial, sans-serif;
	font-size: 100px;
	word-spacing: 1px;
	-ms-text-size-adjust: 100%;
	-webkit-text-size-adjust: 100%;
	-moz-osx-font-smoothing: grayscale;
	-webkit-font-smoothing: antialiased;
	box-sizing: border-box;
}

body {
	font-size: 0.12rem;
	line-height: 2;
}

*,
*:before,
*:after {
	box-sizing: border-box;
	margin: 0;
}

.headers {
	display: flex;
	background-color: #f8f8f8;
	font-size: 0.13rem;
	height: 0.4rem;

	.full {
		flex: 1;
	}

	.logo {
		img {
			height: 0.22rem;
			position: relative;
			top: -0.04rem;
		}
	}
}

.el-menu-item,
.el-submenu__title {
	color: #314659 !important;
	font-size: 0.13rem;
	padding: 0 0.1rem;
	height: 0.39rem !important;
	line-height: 0.39rem !important;

	&.separator {
		border-top: 1px solid #e8e8e8;
		margin: 0 0.1rem;
		height: 0.01rem !important;
		line-height: 0.01rem !important;
	}

	a {
		text-decoration: none;
		color: #314659;
	}

	.iconfont {
		color: #314659;
		font-size: 0.26rem;
	}
}

.body {
	height: calc(100vh - 0.4rem);
	&.big{
		height: 100vh;
	}
}

</style>
