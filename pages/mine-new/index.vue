<template>
	<view class="container">
		<navbar-common rightIcon="https://cxj.zhangjiancong.top/images/cxj/mine/settings.png" rightLink="/pages-mine/pages/mine"></navbar-common>

		<div class="profile" @click="go('/pages-mine/pages/mine')">
			<div class="left">
				<u-image border-radius="25rpx" width="100%" height="100%" :src="userInfo.icon"></u-image>
			</div>
			<div class="right">
				<div class="nickname">{{ userInfo.nickname }}</div>
				<div class="account">账号：{{ userInfo.account }}</div>
				<div class="qrcode"></div>
			</div>
			<div class="more"> > </div>
		</div>

		<div class="cells">
			<u-cell-group>
				<div class="cell" v-for="(item, index) in mains" :key="index" @click="go(item.link)">
					<u-cell-item :arrow="false">
						<div slot="title" class="cell-title">{{ item.title }}</div>
						<image slot="icon" class="cell-icon" :src="item.src"></image>
					</u-cell-item>

				</div>
			</u-cell-group>
		</div>

		<div class="cells">
			<u-cell-group>
				<div class="cell" @click="openHelpModal">
					<u-cell-item :arrow="false">
						<div slot="title" class="cell-title">帮助</div>
						<image slot="icon" class="cell-icon" src='https://cxj.zhangjiancong.top/images/cxj/mine/info.png'></image>
					</u-cell-item>
						
				</div>
				<div class="cell">
					<u-cell-item :arrow="false">
						<u-button class="contact-button" open-type="contact"></u-button>
						<div slot="title" class="cell-title">联系我们</div>
						<image slot="icon" class="cell-icon" src='https://cxj.zhangjiancong.top/images/cxj/mine/phone.png'></image>
					</u-cell-item>
				</div>
				
			</u-cell-group>
		</div>
		
		<div class="logo">
			<div class="image"></div>
		</div>
		
		
		<u-modal v-model="showHelp" title="帮助" :title-style="{ color: themeColor }" :mask-close-able="true"
			:confirm-color="themeColor">
			<view class="slot-content">
				茶小橘化橘红茶为一款基于 Uniapp 开发，面向社区电商场景的微信小程序，涵盖商品、订单、用户、社区和积分商城等内容。
			</view>
		</u-modal>
		

	</view>
</template>

<script>
	export default {
		components: {

		},
		data() {
			return {
				userInfo: {},
				themeColor: this.$appTheme.appThemeColor,
				showHelp: false,
				mains: [{
						title: "我的数据",
						src: "https://cxj.zhangjiancong.top/images/cxj/mine/files.png",
						link: "/pages-mine/pages/user"
					},
					{
						title: "我的收藏",
						src: "https://cxj.zhangjiancong.top/images/cxj/mine/heart.png",
						link: "/pages-mine/pages/collection"
					},
					// {
					// 	title: "我的订单",
					// 	src: "https://cxj.zhangjiancong.top/images/cxj/mine/paper.png",
					// 	link: "/pages-mall/pages/order/list"
					// },
					{
						title: "我的优惠",
						src: "https://cxj.zhangjiancong.top/images/cxj/mine/price.png",
						link: "/pages-mine/pages/discounts"
					},
					{
						title: "我的积分",
						src: "https://cxj.zhangjiancong.top/images/cxj/mine/pack.png",
						link: "/pages-points/pages/index"
					}
				],

			}
		},
		onLoad(options) {
			// uni.setStorageSync("user_id", 1)
		},
		onShow() {
			// 处理首页需要渲染的数据
			// this.loadPageData(false);
			this.initUser();
		},
		methods: {
			// 去往对应页面
			go(link) {
				console.log(link);
				uni.navigateTo({
					url: link
				});
			},
			

			openHelpModal() {
				this.showHelp = true;
			},
	
			contact() {
				uni.makePhoneCall({
				    phoneNumber: '13544361063' //电话号码
				});

			},
			async initUser() {
				this.userInfo = await this.$u.api.getUserMes();
				uni.setStorageSync("userInfo", this.userInfo);
			},
		}
	}
</script>

<style lang="scss" scoped>
	@import './index.scss';
</style>
