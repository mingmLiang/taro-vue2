<template>
  <view id="myHouses">
    <scroll>
      <view class="myHouses-list">
        <view
          v-for="(item, index) in data"
          :key="index"
          class="myHouses-item"
          @tap="itemClick(item)"
        >
          <view class="font-12 font-c-9">最后更新：{{ item.update_time }}</view>
          <view class="myHouses-box">
            <view class="myHouses-status" v-if="item.status==1">审核中</view>
            <view :class="['list-bgc-' + ( index % 6 + 1 )]" class="myHouses-box-header">
              <view class="font-20 font-b font-c-f">{{ item.name }}</view>
              <view
                class="myHouses-detail font-14 font-b font-c-f"
              >{{ item.build}} {{item.cell}} {{item.room }}</view>
            </view>
            <view class="myHouses-box-info">
              <view class="flex-item-grow-1 item">
                <view>
                  <view
                    class="d-inline-block font-20 font-bb font-c-3"
                  >{{ item.EstimatedProgresslink }}</view>
                  <view class="d-inline-block font-13 font-bb font-c-0">%</view>
                </view>
                <view class="font-13 font-c-6">预计进度</view>
              </view>
              <view class="flex-item-grow-1 item">
                <view>
                  <view
                    class="d-inline-block font-20 font-bb font-c-3"
                  >{{ item.ActualProgresslink }}</view>
                  <view class="d-inline-block font-13 font-bb font-c-0">%</view>
                </view>
                <view class="font-13 font-c-6">实际进度</view>
              </view>
              <view class="flex-item-grow-1 item">
                <view>
                  <view class="d-inline-block font-20 font-bb font-c-3">{{ item.article_num }}</view>
                </view>
                <view class="font-13 font-c-6">实景数</view>
              </view>
            </view>
          </view>
        </view>
      </view>

      <default-page
        src="/static/3/dep_nrm@3x.png"
        title="暂无房间"
        desc="当前没有加入房间喔~"
        :bt="{text:'找房间'}"
        @fn="routerLink('roomfindRoomIndex')"
        v-if="isData && data.length === 0"
      ></default-page>
      <view
        class="d-inline-block iconfont icon-custom-add find-house-icon"
        :class="themeClass.font"
        @tap="routerLink('roomfindRoomIndex')"
      ></view>
    </scroll>
    <headerList></headerList>
    <navFooter></navFooter>
  </view>
</template>

<script>
import { getTheme } from "@/utils/utils";
import { getStorageSync, setStorageSync } from "@/utils/tools";
import loadMore from "@components/loadMore/loadMore";
import service from "@/apiModules/service";
import { mapActions, mapMutations } from "vuex";
import { isArray } from "@utils/utils";
import { toPercent } from "@utils/getRoomRule";
import navFooter from "@components/footer/navFooter";
import headerList from "./components/header";
import Scroll from "@components/scroll/scroll";
import { Taro, ScrollView } from "@tarojs/taro";
export default {
  components: {
    loadMore,
    navFooter,
    headerList,
    Scroll
  },
  data() {
    return {
      platform: process.env.TARO_ENV,
      themeClass: {},
      // 滑动列表实例
      bscrollList: null,
      // 页码
      page: 1,
      // 列表是否加载中标识符
      isListLoading: false,
      // 源数据
      data: [],
      isTopLoading: false,
      // 分页参数
      current_page: 0,
      last_page: 1,
      per_page: 0,
      total: 0,
      // 用于新的分页
      isData: "",
      pullDownRefresh: true,
      pullDownRefreshThreshold: 90,
      pullDownRefreshStop: 40,
      pullUpLoad: true,
      pullUpLoadThreshold: 0,
      pullUpLoadMoreTxt: "房间已更新",
      pullUpLoadNoMoreTxt: "没有更多数据了",
      startY: 0,
      scrollToX: 0,
      scrollToY: -200,
      scrollToTime: 700,
      scrollToEasing: "bounce",
      // items: [],
      itemIndex: 0,
    };
  },
  watch: {
    pullDownRefreshObj: {
      handler(val) {
        const scroll = this.$refs.scroll.scroll;
        if (val) {
          scroll.openPullDown();
        } else {
          scroll.closePullDown();
        }
      },
      deep: true,
    },
    pullUpLoadObj: {
      handler(val) {
        const scroll = this.$refs.scroll.scroll;
        if (val) {
          scroll.openPullUp();
        } else {
          scroll.closePullUp();
        }
      },
      deep: true,
    },
    startY() {
      this.rebuildScroll();
    },
  },
  computed: {
    pullDownRefreshObj: function () {
      return this.pullDownRefresh
        ? {
            threshold: parseInt(this.pullDownRefreshThreshold),
            stop: parseInt(this.pullDownRefreshStop),
          }
        : false;
    },
    pullUpLoadObj: function () {
      return this.pullUpLoad
        ? {
            threshold: parseInt(this.pullUpLoadThreshold),
            txt: {
              more: this.pullUpLoadMoreTxt,
              noMore: this.pullUpLoadNoMoreTxt,
            },
          }
        : false;
    },
    // 是否是最后一页标识符
    isLastPage: {
      get: function () {
        if (this.last_page != this.current_page) {
          return false;
        } else {
          return true;
        }
      },
      set: function () {},
    },
  },
  methods: {
    ...mapMutations(["setRoomVersionId", "updateLoadingStatus"]),
    touchFn() {
      console.log("touchFn");
    },
    routerLink(e) {
      // 跳转
      Taro.redirectTo({
        name: e,
      });
    },
    // 列表click处理
    itemClick(item) {
      // 存储 room_version_id
      this.$store.commit("setRoomVersionId", item.house.room_version_id);
      Taro.redirectTo({
        name: "roomDynamic",
        query: {
          room_version_id: item.house.room_version_id,
        },
      });
    },
    // 获取数据
    getData() {
      return new Promise((reslove, reject) => {
        if (this.isLastPage && this.isData != "") {
          reslove([]);
        } else {
          service(this.$store.getters.API_ROUTER_CONFIG.room_version_list, {
            object: getStorageSync("union_type") ? "owner" : "service_personal",
            p: this.page,
          }).then((res) => {
            if (res.status === 200) {
              if (res.data_list.data && isArray(res.data_list.data)) {
                this.current_page = res.data_list.current_page;
                this.last_page = !res.data_list.last_page
                  ? 1
                  : res.data_list.last_page;
                this.per_page = res.data_list.per_page;
                this.total = res.data_list.total;
                if (this.page === 1) {
                  this.data = res.data_list.data;
                  this.data.forEach((res) => {
                    res.build = res.house.build;
                    res.cell = res.house.cell;
                    res.name = res.house.name;
                    res.room = res.house.room;
                    res.EstimatedProgresslink = toPercent(
                      res.EstimatedProgress.progress
                    );
                    res.ActualProgresslink = toPercent(
                      res.ActualProgress.progress
                    );
                  });
                } else {
                  this.data = this.data.concat(res.data_list.data);
                  this.data.forEach((res) => {
                    res.build = res.house.build;
                    res.cell = res.house.cell;
                    res.name = res.house.name;
                    res.room = res.house.room;
                    res.EstimatedProgresslink = toPercent(
                      res.EstimatedProgress.progress
                    );
                    res.ActualProgresslink = toPercent(
                      res.ActualProgress.progress
                    );
                  });
                }
              }
              this.isData = "ing";
              reslove(res.data_list.data);
            } else {
              reslove({});
            }
          });
        }
      });

      // 设置数据
      // 存储房间数据
    },
    // 初始化
    init() {
      this.page = 1;
      this.getData().then(() => {
        this.$store.commit("updateLoadingStatus", { isPageLoading: false });
      });
    },
    // 新的分页数据
    onPullingDown() {
      // 模拟更新数据
      if (this._isDestroyed) {
        return;
      }
      this.page = 1;
      this.getData().then((res) => {
        if (res.length > 0) {
        } else {
          this.$refs.scroll.forceUpdate();
        }
      });
    },
    onPullingUp() {
      // 更新数据
      if (this._isDestroyed) {
        return;
      }
      this.page += 1;

      this.getData().then((res) => {
        if (res.length == 0) {
          this.$refs.scroll.forceUpdate();
        }
      });
    },
    clickItem() {
      this.$router.back();
    },
    rebuildScroll() {
      Vue.nextTick(() => {
        this.$refs.scroll.destroy();
        this.$refs.scroll.initScroll();
      });
    },
  },
  created() {
    getTheme().then((res) => {
      this.themeClass = res;
    });
  },
  mounted() {
    this.init();
    setStorageSync("union_type", "a");
  },
};
</script>

<style lang="less">
.myHouses-box-header {
  padding: 40px 60px;
}
#myHouses {
  width: 100%;
  position: relative;
  .myHouses-box {
    margin-top: 20px;
    border-radius: 20px;
    box-shadow: 0 2px 20px 0 rgba(137, 122, 122, 0.25);
    overflow: hidden;
    margin-bottom: 40px;
    position: relative;
  }
  .myHouses-status {
    position: absolute;
    right: -50px;
    top: 30px;
    transform: rotateZ(45deg);
    background-color: #fff;
    width: 200px;
    text-align: center;
    display: block;
  }
  .myHouses-list {
    padding: 40px 40px 0 40px;
  }

  .myHouses-item {
    &:not(:first-child) {
      margin-top: 0.6rem;
    }
  }

  .myHouses-detail {
    margin-top: 0.12rem;
  }

  .myHouses-box-info {
    display: flex;
    height: 200px;
    padding: 40px 0 63px;

    > .item {
      text-align: center;
    }
  }
  .myHouses-none {
    position: absolute;
    top: 200px;
    left: 0;
    width: 100%;
    .myHouses-none-title {
      margin-bottom: 40px;
      margin-top: 216px;
      text-align: center;
    }
  }
}

.myHouses-btn {
  width: 630px;
  height: 88px;
  margin: 0 auto;
}

.find-house-icon {
  position: fixed;
  right: 30px;
  bottom: 120px;
  font-size: 88px;
  border-radius: 50%;
}
</style>
