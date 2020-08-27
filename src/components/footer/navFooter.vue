<template>
  <view class="nav-box footer">
    <!-- 业主端 -->
    <view v-if="project == 'a'" class="floor">
      <view
        v-for="(item, index) in navData"
        :key="index"
        class="nav-item"
        :class="[
               index === activedNavIndex && index === 1?'nav-box-inner':''
                ]"
        @tap="navClickHandle(item, index)"
      >
        <view class="nav-mask">
          <text
            class="icon iconfont"
            :class="index === activedNavIndex?item.className+'-active '+themeClass.font:item.className+' font-c-70'"
          ></text>
          <view :class="['nav-word', index === activedNavIndex?'font-main-2':'']">{{ item.text }}</view>
        </view>
      </view>
    </view>
    <!-- 服务端 -->
    <view v-if="project == 'b'" class="floor">
      <view
        v-for="(item, index) in navData"
        :key="index"
        :class="{
               'nav-item':true 
            }"
        @tap="navClickHandleCompany(item, index)"
      >
        <view>
          <text
            class="icon iconfont"
            :class="index === activedNavIndex?item.className+'-active '+themeClass.font:item.className+' font-c-70'"
          ></text>
          <view :class="['nav-word', index === activedNavIndex?themeClass.font:'']">{{ item.text }}</view>
        </view>
      </view>
    </view>
    <text
      v-if="project == 'b'"
      class="iconfont icon-custom-add find-house-icon"
      :class="themeClass.font"
      @tap="handleGoPage()"
    ></text>
    <view class="iframe-page" v-if="flag">
      <iframe :src="requestUrl" frameborder="0" @load="load()" scrolling="auto"></iframe>
    </view>
  </view>
</template>

<script>
import Taro from "@tarojs/taro";
import { mapActions } from "vuex";
import CryptoJS from "crypto-js";
import { getStorageSync } from "@/utils/tools";
import { getRouteName } from "@/utils/tools";
import service from "@/apiModules/service";
import { getTheme } from "@/utils/utils";

export default {
  data() {
    return {
      themeClass: {},
      // 当前激活的导航索引
      activedNavIndex: null,
      // 导航数据
      navData: [],
      userInfoStr: "", // 加密后的信息
      userInfo: {},
      project: getStorageSync("union_type"),
      requestUrl: "",
      requestBaseUrl: "",
      flag: false,
    };
  },
  watch: {
    $route() {
      this.updatedStatus();
    },
  },
  methods: {
    ...mapActions(["actGetUserInfo", "actGetReportSsoUrl"]),
    encrypt(data) {
      let text = JSON.stringify(data);
      text = CryptoJS.enc.Base64.stringify(
        CryptoJS.enc.Utf8.parse(text)
      ).toString();
      let key = CryptoJS.enc.Utf8.parse("2343465472318902"); //为了避免补位，直接用16位的秘钥
      let iv = CryptoJS.enc.Utf8.parse("2343465472318902"); //16位初始向量
      let encrypted = CryptoJS.AES.encrypt(text, key, {
        iv: iv,
        mode: CryptoJS.mode.CBC,
        padding: CryptoJS.pad.ZeroPadding,
      }).toString();
      return CryptoJS.enc.Base64.stringify(
        CryptoJS.enc.Utf8.parse(encrypted)
      ).toString();
    },
    // 服务人员端导航
    navClickHandleCompany(item, index) {
      if (item && item.routeName) {
        if (item.routeName == "report") {
          this.requestBaseUrl =
            "https://erp.ijuzhong.com/WebReport/ReportServer?op=fs";
          this.actGetReportSsoUrl({}).then((res) => {
            if (res.status == 200 && res.data_list) {
              if (res.data_list.has_member) {
                this.requestUrl = res.data_list.url;
                this.flag = true;
              } else {
                location.href = this.requestBaseUrl;
              }
            }
          });
        } else {
          this.activedNavIndex = index;
          Taro.redirectTo({
            name: item.routeName,
          });
        }
      }
    },
    load() {
      location.href = this.requestBaseUrl;
      setTimeout(() => {
        this.flag = false;
      }, 100);
    },
    handleGoPage() {
      var routeName = "";
      const currentName = getRouteName();
      switch (currentName) {
        case "myBusiness":
          // 我的单源
          routeName = "addOrderBase";
          break;
        case "company":
          //  加入公司
          routeName = "joinCompany";
          break;
        case "myCustomer":
          routeName = "roomfindRoomIndex";
          // 公司
          break;
        case "me":
          routeName = "roomfindRoomIndex";
          // 我的
          break;
      }
      Taro.redirectTo({
        name: routeName,
      });
    },
    // 导航click处理
    navClickHandle(item, index) {
      if (item && item.routeName) {
        if (item.routeName == "mall2") {
          window.location = `https://mall2.ijuzhong.com/mobile?crm=${this.userInfoStr}`;
        } else {
          Taro.redirectTo({
            name: item.routeName,
          });
        }
      }
    },
    // 更新状态
    updatedStatus() {
      if (this.project == "a") {
        this.navData = [
          {
            text: "房间",
            routeName: "myHouses",
            routeNameL: "modelRooms",
            className: "icon-custom-room",
          },
          {
            text: "装修",
            routeName: "design",
            routeNameL: "construction",
            routeNameR: "steward",
            className: "icon-custom-decorate",
          },
          {
            text: "商城",
            routeName: "mall2",
            routeNameL: "mall2",
            className: "icon-custom-mall",
          },
          {
            text: "我的",
            routeName: "me",
            className: "icon-custom-my",
          },
        ];
      } else {
        this.navData = [
          {
            text: "工地",
            routeName: "myCustomer",
            className: "icon-custom-client",
          },
          {
            text: "单源",
            routeName: "myBusiness",
            routeNameL: "businessFrom",
            className: "icon-custom-business",
          },
          {
            text: "公司",
            routeName: "company",
            className: "icon-custom-room",
          },
          {
            text: "报表",
            routeName: "report",
            // routeName: "customerReport",
            // routeNameL: "businessReport",
            className: "icon-custom-chart",
          },
          {
            text: "我的",
            routeName: "me",
            className: "icon-custom-my",
          },
        ];
      }
      const currentName = getRouteName();
      for (let i = 0; i < this.navData.length; i++) {
        console.log(
          "init",
          this.project,
          this.navData[i].routeName,
          this.navData[i].routeNameL,
          this.navData[i].routeNameR,
          currentName
        );
        if (
          currentName == this.navData[i].routeName ||
          currentName == this.navData[i].routeNameL ||
          currentName == this.navData[i].routeNameR
        ) {
          this.activedNavIndex = i;
          // break
        }
      }
    },
    // 初始化
    init() {
      this.updatedStatus();
    },
  },
  mounted() {
    this.init();
    getTheme().then((res) => {
      this.themeClass = res;
      console.log(this.themeClass, "this.themeClass");
    });
    this.$nextTick(() => {
      service(
        this.$store.getters.API_ROUTER_CONFIG.get_user_info,
        {},
        "get"
      ).then((res) => {
        this.userInfo = res.data_list;
        const obj = {
          user_id: this.userInfo && this.userInfo.id,
          nick_name: this.userInfo && this.userInfo.nick_name,
        };
        this.userInfoStr = this.encrypt(obj);
      });
    });
  },
  updated() {},
};
</script>
<style lang="less">
.find-house-icon {
  position: fixed;
  right: 30px;
  bottom: 120px;
  font-size: 88px;
  border-radius: 50%;
}
.nav-mask {
  img {
    height: 80%;
    width: 80%;
  }
}
.floor {
  display: flex;
  width: 100vw;
}
.nav-box-inner {
  display: flex;
  .nav-mask {
    // background-color: #ea6767;
    img {
      height: 80%;
      width: 80%;
    }
  }
}
.logo-img {
  height: 48px;
  width: 51px;
  display: block;
  margin: 0 auto;
}
.nav-box {
  height: 98px;
  position: fixed;
  left: 0;
  bottom: 0;
  width: 100%;
  background-color: #fff;
  // height: 51px;
  display: flex;
  width: 100%;
  padding-top: 8px;
  box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.35);
}

.nav-item {
  flex: 1;
  height: 100%;
  text-align: center;
  .icon {
    font-size: 30px;
  }

  &.findHouse {
    position: relative;
    .nav-mask {
      display: flex;
      align-items: center;
      justify-content: center;
      flex-direction: column;
      position: absolute;
      left: 50%;
      transform: translate(-50%, 0);
      width: 114px;
      height: 114px;
      bottom: 8px;
      border-radius: 50%;
    }
    .nav-word {
      color: #fff;
    }
    .nav-focus {
      margin-top: 6px;
      color: #fff;
      font-size: 20px;
    }
  }
}
.iframe-page {
  display: none;
}
.nav-word {
  font-size: 20px;
  color: #707070;
  opacity: 0.7;
}
</style>
