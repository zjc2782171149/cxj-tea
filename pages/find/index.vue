<template>
  <view class="container">
    <navbar-common></navbar-common>

    <div class="new">
      <div class="new-up">·喝茶日历·</div>
    </div>

    <div class="calendar">
      <div class="calendar_out">
        <div class="top">
          <div class="date">
            <div class="month">{{ month }}月/</div>
            <div class="year">{{ year }}年</div>
          </div>
          <div class="record" @click="recordDailyTea">点击记录</div>
        </div>
        <div class="bottom">
          <div class="eachday" v-for="(item, index) in calendar" :key="index">
            <div :class="['name', item.isRecord ? 'color-gold' : '']">
              {{ item.day }}
            </div>
            <div
              :class="[
                'number',
                item.isRecord ? 'border-gold color-gold' : 'border-grey',
              ]"
            >
              {{ item.date }}
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="new">
      <div class="new-up">·资讯广场·</div>
    </div>

    <div class="tabs">
      <u-tabs
        :list="opsList"
        :current="type"
        @change="change"
        bar-height="12"
        bar-width="100"
        active-color="#dabd7b"
        gutter="20"
        inactive-color="grey"
        f-size="20"
        fontSize="30"
        letterSpacing="7"
      >
      </u-tabs>
    </div>

    <!-- 回答 -->
    <QuestionList v-if="type === 0" :data="list" :type="type"></QuestionList>

    <!-- 官方科普 -->
    <u-cell-group v-if="type === 1">
      <div class="cell" v-for="(item, index) in list" :key="item.article_id">
        <u-cell-item :arrow="false" @click="turnArticleDetail(item)">
          <div slot="title" class="cell-main">
            <div class="cell-title">{{ item.title }}</div>
            <div class="cell-author">
              <image class="cell-icon" :src="item.icon"></image>

              <div class="cell-nickname">{{ item.nickname }}</div>
              <div class="cell-rank">{{ item.grade }}</div>
            </div>
            <div class="cell-profile">
              <u-parse class="cell-profile" :html="item.desc"></u-parse>
            </div>
          </div>

          <image slot="right-icon" class="cell-image" :src="item.img"></image>
        </u-cell-item>
      </div>
    </u-cell-group>

    <!-- 每日喝茶记录 -->
    <div class="daily-tea" v-if="type === 2">
      <text>您已连续记录喝茶{{ consistentRecordDay }}天啦</text>
    </div>
    <RecordList v-if="type === 2" :data="list" :type="type"></RecordList>

    <!-- 加载更多 -->
    <view class="wrap">
      <u-loadmore :status="status" @loadmore="loadmore" :load-text="loadText" />
    </view>

    <div class="blank"></div>

    <!-- 添加按钮 -->
    <AddBtnFixed @click="add"></AddBtnFixed>
    <!-- 新增弹窗 -->
    <AddPopup ref="AddPopup" @close="closeAddPopup" :ops="addOps"></AddPopup>
  </view>
</template>

<script>
import AddBtnFixed from "@/components/add-btn-fixed.vue";
import AddPopup from "@/components/add-popup.vue";
import QuestionList from "@/pages/find/components/question-list.vue";
import RecordList from "@/pages/find/components/record-list.vue";

import dayjs from "dayjs";
import relativeTime from "dayjs/plugin/relativeTime";
import updateLocale from "dayjs/plugin/updateLocale";
import isLeapYear from "dayjs/plugin/isLeapYear";
import "dayjs/locale/zh-cn";
dayjs.extend(relativeTime);
dayjs.extend(updateLocale);
dayjs.locale("zh-cn");
dayjs.extend(isLeapYear);

export default {
  components: {
    AddBtnFixed,
    AddPopup,
    QuestionList,
    RecordList,
  },
  data() {
    return {
      opsList: [
        {
          name: "问答",
        },
        {
          name: "科普",
        },
        {
          name: "记录",
          // count: 5
        },
      ],
      type: 0,
      articleList: [],
      showArticleList: [], // 应该展示的文章类别
      // 在切换到关注的时候不显示搜索框
      showSearch: true,
      // 新建按钮配置项
      addOps: [
        {
          label: "每日喝茶记录",
          icon: "https://cxj.zhangjiancong.top/images/cxj/find/create-community.png",
          url: "/pages/find/add",
          type: "dailyRecord",
        },
        {
          label: "问题发布",
          icon: "https://cxj.zhangjiancong.top/images/cxj/find/publish-community.png",
          url: "/pages/find/add",
          type: "question",
        },
      ],
      calendar: [],
      dayArr: ["日", "一", "二", "三", "四", "五", "六"],
      monthAr: [
        31,
        31,
        dayjs().isLeapYear() ? 29 : 28,
        31,
        30,
        31,
        30,
        31,
        31,
        30,
        31,
        30,
      ],
      year: "",
      month: "",
      consistentRecordDay: 0,

      status: "loadmore",
      list: [], // 真正要展示的文章列表
      page: 0,
      loadText: {
        loadmore: "上拉或点击我以加载更多",
        loading: "努力加载中",
        nomore: "已经到底啦",
      },
    };
  },
  onLoad() {
    this.initUser();
    this.initCalendar();
  },
  onShow() {
    this.initArticleList();
  },
  onReachBottom() {
    this.loadmore();
  },
  methods: {
    // 加载更多
    loadmore() {
      // console.log(this.page, this.showArticleList.length);

      this.status = "loading";

      setTimeout(() => {
        this.page = ++this.page; // 增加页数

        if (this.page >= Math.ceil(this.showArticleList.length / 10)) {
          this.status = "nomore";
        } else {
          this.list = this.showArticleList.slice(0, (this.page + 1) * 10); // 显示更多列表出来
          this.status = "loadmore";
        }
      }, 1000);
    },
    // 改变日历上的记录
    reloadRecord() {
      let recordArr = [];

      // 自己 发的记录
      this.articleList.map((item) => {
        if (item.type === 2) {
          recordArr.push(item);
        }
      });

      let recordArr2 = recordArr.map((item) => {
        return {
          year: item.publish_time.slice(0, 4) * 1,
          month: item.publish_time.slice(5, 7) * 1,
          date: item.publish_time.slice(8, 10) * 1,
        };
      });

      console.log(recordArr2);

      this.calendar = this.calendar.map((item, index) => {
        // 如果发表的记录中，这一天有记录，那么便进行标记
        if (
          recordArr2.find(
            (record) =>
              record.year === item.year &&
              record.month === item.month &&
              record.date === item.date
          )
        ) {
          return {
            ...item,
            isRecord: true,
          };
        } else {
          return item;
        }
      });
    },
    // 计算连续喝茶天数
    consistentRecord() {
      let recordArr = [];

      // 自己 发的记录
      this.articleList.map((item) => {
        if (item.type === 2) {
          recordArr.push(item);
        }
      });

      let recordArr2 = recordArr.map((item) => {
        return {
          year: item.publish_time.slice(0, 4) * 1,
          month: item.publish_time.slice(5, 7) * 1,
          date: item.publish_time.slice(8, 10) * 1,
        };
      });

      // 喝茶日历进行初始化
      let year = dayjs().year();
      let month = dayjs().month();
      let date = dayjs().date();

      let i;
      for (i = 0; ; i++) {
        if (
          !recordArr2.find(
            (item) =>
              item.year === year &&
              item.month === month + 1 &&
              item.date === date
          )
        ) {
          break;
        }

        if (date - 1 === 0) {
          date = this.monthAr[dayjs().month()];
          // 月份要减一
          if (month - 1 === -1) {
            month = 11;
            // 年份也要减一
            year -= 1;
          } else {
            month -= 1;
          }
        } else {
          date -= 1;
        }
      }

      this.consistentRecordDay = i;
    },
    recordDailyTea() {
      uni.navigateTo({
        url: "/pages/find/add?type=" + "dailyRecord",
      });
    },
    async initArticleList() {
      const that = this;

      const res = await this.$u.api.getArticleAll(); // 文章基本信息
      const task = res.map((item) => {
        return that.$u.api
          .getUserMesSome({
            user_id: item.user_id,
          })
          .then();
      });

      const res2 = await Promise.all(task); // 文章作者部分个人信息

      this.articleList = res.map((item, index) => {
        return {
          ...item,
          ...res2[index],
          publish_time: getApp().globalData.getNowTime(
            dayjs(item.publish_time).format()
          ),
        };
      });
      this.articleList = this.articleList.reverse(); // 反转，最新发的在前面

      this.showArticleList = [];

      this.articleList.map((item) => {
        if (item.type === this.type) {
          that.showArticleList.push({
            ...item,
            publish_time: dayjs(item.publish_time).fromNow(),
          });
        }
      });

      this.list = this.showArticleList.slice(0, (this.page + 1) * 10);
      console.log("要展现的文章", this.list);

      // 修改日历上的喝茶记录
      this.reloadRecord();
      // 计算连续喝茶天数
      this.consistentRecord();
    },
    async initUser() {
      this.userInfo = await this.$u.api.getUserMes();
      uni.setStorageSync("userInfo", this.userInfo);
    },
    turnArticleDetail(item) {
      uni.navigateTo({
        url:
          "/pages/find/answer-detail?id=" +
          item.article_id +
          "&index=" +
          this.type,
      });
    },
    // 喝茶日历进行初始化
    initCalendar() {
      let year = dayjs().year();
      let month = dayjs().month();
      let date = dayjs().date();
      let day = dayjs().day();
      let arr = new Array(7);

      this.year = year;
      this.month = month + 1;

      for (let i = 6; i >= 0; i--) {
        arr[i] = {
          year: year, // 年
          month: month + 1, // 月
          date: date, // 日
          day: this.dayArr[day], // 星期几
          isRecord: false,
        };

        if (date - 1 === 0) {
          date = this.monthAr[dayjs().month()];
          // 月份要减一
          if (month - 1 === -1) {
            month = 11;
            // 年份也要减一
            year -= 1;
          } else {
            month -= 1;
          }
        } else {
          date -= 1;
        }

        if (day - 1 === -1) {
          day = 6;
        } else {
          day -= 1;
        }
      }
      // console.log("数组为", arr);
      this.calendar = arr;
    },
    change(index) {
      this.type = index;

      const that = this;
      this.showArticleList = [];
      this.articleList.map((item) => {
        if (item.type === this.type) {
          that.showArticleList.push({
            ...item,
            publish_time: dayjs(item.publish_time).fromNow(),
          });
        }
      });

      // 重置加载更多
      this.page = 0;
      (this.status = "loadmore"),
        (this.list = this.showArticleList.slice(0, (this.page + 1) * 10));
      // console.log("要展现的文章", this.showArticleList);
    },

    // 添加
    add() {
      this.$refs.AddPopup.open();
    },

    // 关闭
    closeAddPopup() {
      this.$refs.AddPopup.close();
    },
  },
};
</script>

<style lang="scss" scoped>
@import "./index.scss";
</style>
