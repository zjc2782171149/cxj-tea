<template>
	<view class="page" style="padding-top: 0vh">
		<view>
			<u-navbar :custom-back="toMine" title="我的订单" :background="{ backgroundColor: 'rgb(221, 195, 135)' }">
			</u-navbar>
		</view>

		<view class="search-slot">
			<u-search placeholder="搜索全部订单" :showAction="false" shape="round" v-model="searchValue" 
			bg-color="#F4F5F8" @custom="search" @search="search" @clear="clear" @change="change">
			</u-search>
		</view>

		<div class="tabs">
			<u-tabs :list="tabOps" :current="tabIndex" @change="change" bar-height="12" bar-width="80"
				active-color="#dabd7b" gutter="10" inactive-color="grey" fontSize="30" letterSpacing="7">
			</u-tabs>
		</div>

		<!-- 列表 -->
		<view class="list">
			<OrderCard v-for="(item, index) in showList" :key="index" :data="item" :this="this"
				@updateOrderList="updateOrderList"></OrderCard>
			<NoData height="60vh" type="order" v-if="showList.length === 0"></NoData>
		</view>
	</view>
</template>

<script>
	import dayjs from "dayjs";
	// 组件
	import OrderCard from "@/pages-mall/components/order/order-card.vue";

	export default {
		components: {
			OrderCard,
		},
		data() {
			return {
				searchValue: '',
				userInfo: {},
				orderList: [],
				showList: [],
				// 当前tab
				tabIndex: 0,
				tabOps: [{
						name: "待付款",
						value: 0,
					},
					{
						name: "待发货",
						value: 1,
					},
					{
						name: "待收货",
						value: 2,
					},
					{
						name: "待评价",
						value: 3,
					},
					{
						name: "退款/售后",
						value: 4,
					},
					{
						name: "全部",
						value: 5,
					},
				],
			};
		},
		onLoad(ops) {
			this.initUser();
			this.getOrderList(0);
		},
		onShow() {},
		methods: {
			change() {
				if(this.searchValue === '') {
					this.showList = this.goodsList;
				}
			},
			toMine() {
				console.log("返回到主页");
				uni.switchTab({
					url: '/pages/mine-new/index'
				})
			},
			initUser() {
				this.userInfo = uni.getStorageSync("userInfo");
			},
			search() {
				const that = this;
				
				let arr = [];
				this.orderList.map((item, index) => {
					// 订单中物品列表有搜索框文字，加入
					if(item.goods.findIndex(itemm => itemm.title.includes(that.searchValue)) !== -1) {
						arr.push(this.orderList[index]);
					}
				});
				this.showList = arr;
				this.tabIndex = 5;
			},
			clear() {
				this.searchValue = '';
				this.change(5);
			},
			// 更新订单列表
			updateOrderList(index) {
				this.getOrderList(index);
			},
			// 切换tab
			change(index) {
				this.tabIndex = index;
				const tab = this.tabOps[index];
				
				if (index === 5) {
					this.showList = this.orderList;
				} else {
					let arr = [];
					this.orderList.map((item) => {
						// 是该tab状态的订单才加入数组
						if (item.status === tab.value) {
							arr.push(item);
						}
					});
					this.showList = arr;
				}

			},

			// 初始化订单列表
			async getOrderList(selectTabIndex) {
				let orderList = [];
				// orderListDetail = [];
				const that = this;

				// 获取我的订单列表
				const res1 = await this.$u.api.getOrderList();
				res1.map((item) => {
					orderList.push({
						order_id: item.order_id,
					});
				});

				// 根据订单id去获取订单的地址信息
				let task1 = orderList.map((item) => {
					return that.getOrderAddress(item).then();
				});
				task1 = await Promise.all(task1);
				task1 = task1.map((item) => item[0]);
				orderList.map((item, index) => {
					orderList[index] = {
						...orderList[index],
						...task1[index],
					};
				});
				// console.log(orderList);

				// 根据订单id去获取订单的基本信息
				task1 = orderList.map((item) => {
					return that.getOrderMes(item).then();
				});
				task1 = await Promise.all(task1);
				task1 = task1.map((item) => item[0]);
				orderList.map((item, index) => {
					orderList[index] = {
						...orderList[index],
						...task1[index],
						create_time: getApp().globalData.getNowTime(
							dayjs(task1[index].create_time).format()
						),
						end_time: getApp().globalData.getNowTime(
							dayjs(task1[index].end_time).format()
						),
					};
				});
				// console.log(orderList);

				// 根据订单id去获取订单所购买的物品信息
				task1 = orderList.map((item) => {
					return that.getOrderGoodsList(item).then();
				});
				task1 = await Promise.all(task1);
				orderList.map((item, index) => {
					orderList[index] = {
						...orderList[index],
						goods: task1[index],
					};
				});
				console.log(orderList);

				this.orderList = orderList;

				this.change(selectTabIndex);
			},
			// 根据订单id去获取订单的地址信息
			getOrderAddress(item) {
				return this.$u.api.getOrderAddress({
					order_id: item.order_id,
				});
			},
			// 根据订单id去获取订单的基本信息
			async getOrderMes(item) {
				return this.$u.api.getOrderMes({
					order_id: item.order_id,
				});
			},
			// 根据订单id去获取订单的基本信息
			async getOrderGoodsList(item) {
				return this.$u.api.getOrderGoodsList({
					order_id: item.order_id,
				});
			},
		},
	};
</script>

<style lang="scss" scoped>
	.list {
		margin-top: 30rpx;
		padding: 0 30rpx;
	}

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
					content: "";
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
		// padding-left: 50rpx;
		width: 90%;
		margin: 20rpx 0 20rpx 30rpx;
	}
</style>
