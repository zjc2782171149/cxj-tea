<template>
	<view class="container">
		<navbar-common></navbar-common>

		<div class="swiper">
<!-- 						<swiper class="banner-swiper" circular indicator-dots="true" indicator-color="rgb(244, 246, 252)" indicator-active-color="rgb(228, 197, 152)" 
			autoplay :interval="2000" :duration="500">
				<swiper-item class="banner-swiper-item" v-for="(item, index) in swipers" :key="item.act_id"
				@click="$u.route({ url: '/pages/actDetail/index', params: { url: item.url } })">
					<u-image width="100%" height="100%" :src="item.pics" :fade="true" duration="450">
						<view slot="error" style="font-size: 24rpx;">加载失败</view>
					</u-image>
					<div class="text-swiper">{{item.title}}</div>
					<div class="grey"></div>
				</swiper-item>
			</swiper> -->
			<u-swiper :list="list" class="swiper-i" title="true"  border-radius="14" img-mode="aspectFill" indicator-pos="bottomRight" 
			 bg-color="rgb(191, 191, 191)"></u-swiper>
		</div>



		<div class="new">
			<div class="new-up">·新品上市·</div>
		</div>

		<div class="new-items">
			<div class="new-item" v-for="(item, index) in new_up" :key="item.good_id"
				@click="$u.route({ url: '/pages-mall/pages/goods/detail', params: { id: item.good_id } })">
				<u-image width="100%" height="100%" :src="item.pics" :fade="true" duration="450">
					<view slot="error" style="font-size: 24rpx;">加载失败</view>
				</u-image>
				<div class="text">{{ item.title.slice(0, 9) }}</div>
				<div class="grey"></div>
			</div>

			<NoData height="25vh" type="good" v-if="new_up.length === 0"></NoData>

		</div>

		<div class="new like">
			<div class="new-up">·活动推荐·</div>
		</div>

		<!-- 横向滚动列表 -->
		<colmun-act :list="horizontalScrollNavList"></colmun-act>

		<div class="blank"></div>

	</view>
</template>

<script>
	import {
		RandomNum
	} from '@/utils/random.js';
	import ColmunAct from '@/components/colmun-act.vue';

	export default {
		components: {
			ColmunAct
		},

		data() {
			return {
				swipers: [],
				new_up: [],

				// 横向滚动列表
				horizontalScrollNavList: [],
				list: [{
						image: 'https://cxj.zhangjiancong.top/images/cxj/swiper1.jpg',
						title: '蒹葭苍苍，白露为霜。所谓伊人，在水一方'
					},
					{
						image: 'https://cxj.zhangjiancong.top/images/cxj/swiper2.jpg',
						title: '溯洄从之，道阻且长。溯游从之，宛在水中央'
					},
					{
						image: 'https://cxj.zhangjiancong.top/images/cxj/swiper3.jpg',
						title: '蒹葭萋萋，白露未晞。所谓伊人，在水之湄'
					}
				],
				swiperTitle: {
					height: "2px",
					fontSize: '13px',
					fontWeight: '600',
					backgroundColor: 'rgb(191, 191, 191)',
					opacity: '0.741',
					fontFamily: "SourceHanSansCN",
					color: 'rgb(255, 255, 255)',
					zIndex: '1000'
				}
			}
		},
		onLoad() {

		},
		onShow() {
			this.initUser();
			this.initActList();
		},
		methods: {
			async initUser() {
				this.userInfo = await this.$u.api.getUserMes();
				uni.setStorageSync("userInfo", this.userInfo);
			},
			// 初始化
			async initActList() {
				this.swipers = [];
				this.new_up = [];
				this.horizontalScrollNavList = [];

				const actList = await this.$u.api.getActsAll(); // 活动列表
				const swiperList = await this.$u.api.getSwipersAll(); // 轮播图列表
				const newUpList = await this.$u.api.getGoodsAll(); // 新品上市列表
				// 从活动列表中随机选3个到活动推荐
				let sum = actList.length >= 3 ? 3 : actList.length,
					randomArr = [];
				while (true) {
					if (sum === 0)
						break;

					let i = RandomNum(0, actList.length);
					if (randomArr.findIndex(item => item === i) === -1) {
						randomArr.push(i);
						this.horizontalScrollNavList.push(actList[i]);
						sum--;
					}
				}

				// 从活动列表中随机选4个到轮播图列表
				sum = swiperList.length >= 4 ? 4 : swiperList.length, randomArr = [];
				while (true) {
					if (sum === 0)
						break;

					let i = RandomNum(0, swiperList.length);
					if (randomArr.findIndex(item => item === i) === -1) {
						randomArr.push(i);
						this.swipers.push(swiperList[i]);
						sum--;
					}
				}

				this.new_up = newUpList.reverse().slice(0, 5);

			},
			// 去购物粗
			goCommodityDetail() {
				uni.navigateTo({
					url: '/pages-mall/pages/goods/detail'
				});
			},

		}
	}
</script>

<style lang="scss" scoped>
	@import './index.scss';
</style>
