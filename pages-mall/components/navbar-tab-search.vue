<template>
	<view class="navbar" :style="[{ minHeight: navBarHeight + 'px' }]">
		<view class="inner">
			<!-- 			<view class="navbar-slot">
				<view class="navbar-left" @click="$u.route({ type: 'navigateBack', delta: 1 })">
					<u-icon size="32" name="arrow-leftward" :color="$appTheme.appThemeTextBlackColor"></u-icon>
				</view>

				<view class="navbar-title">{{ title }}</view>

				<view class="navbar-right"></view>

			</view> -->

			<view>
				<u-navbar back-text="返回" :title="`${title}`" :background="{ backgroundColor: 'rgb(221, 195, 135)' }">
				</u-navbar>
			</view>


			<view class="search-slot" :style="{ paddingTop: menuTop + 'px' }">
				<u-search :placeholder="placeholder" :showAction="false" shape="square" borderRadius="0rpx"
					bg-color="#F4F5F8"></u-search>
			</view>



			<div class="tabs">
				<u-tabs :list="tabOps" :is-scroll="false" :current="current" @change="change" bar-height="12"
					bar-width="80" active-color="#dabd7b" gutter="10" inactive-color="grey" fontSize="30"
					letterSpacing="7">
				</u-tabs>
			</div>


			<!-- 

			<view class="tab-slot">
				<view class="tab" v-for="(tab, index) in tabOps" :key="index" :class="{ active: tabIndex == index }"
					@click="change(tab, index)">{{ tab.label }}</view>
			</view> -->
		</view>
	</view>
</template>

<script>
	const app = getApp();
	export default {
		name: 'navbar-tab-search',
		props: {
			// 标题
			title: {
				type: String,
				default: '标题'
			},
			// 占位内容
			placeholder: {
				type: String,
				default: '请输入'
			},
			// tab配置项
			tabOps: {
				type: Array,
				default: () => {
					return [];
				}
			}
		},
		data() {
			return {
				// 导航栏高度
				menuTop: app.globalData.menuTop,
				navBarHeight: app.globalData.navBarHeight,
				menuRight: app.globalData.menuRight,
				menuBotton: app.globalData.menuBotton,
				menuHeight: app.globalData.menuHeight,
				statusBarHeight: app.globalData.statusBarHeight,

				// 当前tab
				tabIndex: 0,

				current: 0,
			};
		},
		methods: {
			// 切换tab
			change(index) {
				console.log(index);
				this.current = index;
				this.tabIndex = index;
				const tab = this.tabOps[index];
				this.$emit('change', {
					tab,
					index
				});
			},

		}
	};
</script>

<style lang="scss" scoped>
	.navbar {
		width: 100vw;
		position: fixed;
		top: 0;
		left: 0;
		z-index: 899;
		overflow: hidden;
	}

	.inner {
		width: 100%;
		height: 100%;
		background-color: $app-theme-bg-color;
	}

	.navbar-slot {
		display: flex;
		align-items: center;
		justify-content: space-between;

		.navbar-title {
			font-size: 36rpx;
			font-family: PingFang SC;
			font-weight: 400;
			color: $app-theme-text-black-color;
		}
	}

	.tab-slot {
		display: flex;
		align-items: center;
		justify-content: space-between;
		padding-top: 20rpx;

		.tab {
			font-size: 26rpx;
			font-weight: 400;
			color: $app-theme-navbar-tab-color;

			&.active {
				color: $app-theme-navbar-tab-color-active;
				position: relative;

				&::before {
					content: '';
					position: absolute;
					bottom: 0;
					left: 50%;
					transform: translate(-50%, -50%);
					width: 20rpx;
					height: 4rpx;
					background: $app-theme-navbar-tab-color-active;
					border-radius: 2rpx;
				}
			}
		}
	}
	
.search-slot {
	margin: 20rpx 0;
}
</style>
