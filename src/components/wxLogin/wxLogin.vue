<template>
  <div>
    <button open-type="getUserInfo" @getuserinfo="bindGetUserInfo">授权登录</button>
  </div>
</template>

<script>
import JSEncrypt from "@/login/utils/jsencrypt";
import Base64 from "js-base64";
import {
  getAuthorityLogin,
  getUserInfo,
  getApplicationTheme,
  wxOauthAppletMessage,
} from "@/login/loginApi";
export default {
  data() {
    return {};
  },
  components: {},
  methods: {
    bindGetUserInfo: function (e) {
      const that = this;
      if (!wx.getStorageSync("loginResCode")) {
        wx.login({
          success: function (res) {
            if (res.code) {
              wx.setStorageSync("loginResCode", res.code);
              that.getOpenid(that);
            }
          },
          error: function () {
            that.toast("小程序官方登陆失败，wx.login");
          },
        });
      }
      const detail = e.mp.detail;
      if (detail) {
        wx.setStorageSync("encryptedData", detail.encryptedData);
        wx.setStorageSync("iv", detail.iv);
        this.getOpenid(this);
      } else {
        //用户按了拒绝按钮
        wx.showModal({
          title: "警告",
          content: "您点击了拒绝授权，将无法进入小程序，请授权之后再进入!!!",
          showCancel: false,
          confirmText: "返回授权",
          success: function (res) {
            if (res.confirm) {
              console.log("用户点击了“返回授权”");
            }
          },
        });
      }
    },
    getOpenid(that) {
      const loginResCode = wx.getStorageSync("loginResCode");
      const encryptedData = wx.getStorageSync("encryptedData");
      const iv = wx.getStorageSync("iv");
      if (!loginResCode || !encryptedData || !iv) {
        return;
      }
      const accountInfo = wx.getAccountInfoSync();
      console.log("wxOauthAppletMessage", wxOauthAppletMessage);
      wxOauthAppletMessage({
        app_id: accountInfo.miniProgram.appId,
        code: loginResCode,
        encryptedData: encryptedData,
        iv: iv,
      }).then((res) => {
        console.log("返回", res);
        if (res && res.status === 200) {
          const data = res.data_list;
          wx.setStorageSync("openid", data.openid);
          wx.setStorageSync("user_id", data.user_id);
          that.toLogin(that);
        } else {
          that.toast(`Openid获取失败${res.msg}`);
        }
      });
    },
    toLogin(that) {
      const openid = wx.getStorageSync("openid");
      if (!openid) {
        return;
      }
      let js_encrypt = new JSEncrypt();
      js_encrypt.setPublicKey(`
      -----BEGIN PUBLIC KEY-----
      MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC3RRmkj4VV/wqEAmLbfHMOzUYT
      qFMsqPLae3W/RKGElVbhkOSEhrjiliCm+xvLrQDSYq0Xy2lQ6E4NE+lYjV1GWKU7
      NVZdehyDXoUTqrLc1WgTq4Pk0V6UdeD6I4RIvKKAJF1thmFaLydtPDMf9O87DuMb
      SArH6hh2Go0mlcANEQIDAQAB
      -----END PUBLIC KEY-----`);
      let enData = Base64.Base64.encode(js_encrypt.encrypt(openid));
      let _data = {
        home_account: "b00ceeacb94d036010dc0d9502aada8c",
        openid: enData,
      };
      // 前台登录
      getAuthorityLogin(_data).then((res) => {
        if (res.status === 200) {
          wx.setStorageSync("token", res.data_list.access_token);
          wx.setStorageSync("tokenStart", new Date().valueOf());
          wx.setStorageSync("refToken", res.data_list.refresh_token);
          wx.setStorageSync("refTokenStart", new Date().valueOf());
          //从数据库获取用户信息
          that.getUserInfo();
          that.back();
        } else {
          that.toast("系统内部登录失败，请联系客服");
        }
      });
    },
  },
};
</script>

<style>
.card {
  padding: 10px;
  margin: 0;
}
</style>
