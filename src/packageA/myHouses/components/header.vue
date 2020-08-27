<template>
  <view class="pages-myHouses-header">
    <view class="flex box">
      <view
        class="myHousesBase-item font-13 font-c-9"
        :class="[
           index === activedNavIndex?themeClass.font:''
        ]"
        v-for="(item, index) in navData"
        :key="index"
        @tap="routerTo(item, index)"
      >
        {{ item.text }}
        <view v-show="index === activedNavIndex" :class="themeClass.bg" class="liner"></view>
      </view>
    </view>
  </view>
</template>

<script>
import Taro from "@tarojs/taro";
import { getRouteName } from "@/utils/tools";
import { getTheme } from "@/utils/utils";
export default {
  components: {},
  props: {},
  data() {
    return {
      themeClass: {},
      // 导航数据
      navData: [
        {
          text: "我的房间",
          routeName: "myHouses",
        },
        {
          text: "样板房",
          routeName: "modelRooms",
        },
      ],
    };
  },
  watch: {},
  computed: {
    // 被激活导航索引
    activedNavIndex() {
      const routeName = getRouteName();
      if (routeName === "myHouses") {
        return 0;
      } else if (routeName === "modelRooms") {
        return 1;
      }
    },
  },
  methods: {
    routerTo(item, index) {
      Taro.redirectTo({
        name: item.routeName,
      });
    },
  },
  created() {
    getTheme().then((res) => {
      this.themeClass = res;
    });
  },
  mounted() {},
};
</script>

<style lang="less">
.pages-myHouses-header {
  position: relative;
  height: 100px;
  background-color: #fff;
  z-index: 99;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;

  .box {
    display: flex;
    width: 100%;
    line-height: 100px;
    background-color: #fff;
    box-shadow: 0 2px 6px 0 rgba(137, 122, 122, 0.25);
  }

  .myHousesBase-item {
    position: relative;
    display: block;
    height: 100%;
    flex: 1;
    text-align: center;

    .liner {
      position: absolute;
      bottom: 0;
      left: 50%;
      width: 20px;
      height: 8px;
      transform: translateX(-50%);
    }
  }
}
</style>
