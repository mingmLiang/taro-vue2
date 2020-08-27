<template>
  <div class="default-page">
    <img :src="src" alt="加载中...">
    <div class="container">
      <p class="font-17 font-c-3 tips">{{ title }}</p>
      <p class="font-13 font-c-9 desc">{{ desc }}</p>
      <div v-if="bt.text"
           :style="{'color':bt.color || '#fff','backgroundColor':bt.bgColor || '#26BD8A'}"
           class="bt"
           @tap="handleClick">{{ bt.text}}
      </div>
    </div>
  </div>
</template>

<script>
import Taro from "@tarojs/taro";
  export default {
    props: {
      src: {
        type: String,
        default: ""
      },
      title: {
        type: String,
        default: ""
      },
      desc: {
        type: String,
        default: ""
      },
      bt: {
        type: Object,
        default: function () {
          return {}
        }
      }
    },
    methods: {
      handleClick() {
        this.bt.routeName &&
        Taro.redirectTo({
          name: this.bt.routeName,
          query: this.bt.query
        });
        this.$emit('fn');
      }
    }
  }
</script>

<style scoped lang="less">
  .default-page {
    position: relative;
    width: 100%;
    img {
      width: 100%;
    }
    .container {
      position: absolute;
      top: 50%;
      left: 0;
      width: 100%;
      text-align: center;
      .tips, .desc {
        font-weight: bold;
        margin-bottom: 20px;
      }
      .bt {
        width: 400px;
        height: 88px;
        line-height: 88px;
        margin: 80px auto 0;
        font-size: 34px;
        border-radius: 44px;
      }
    }
  }
</style>
