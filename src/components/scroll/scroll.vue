<template>
  <view class="content">
    <scroll-view
      :scroll-y="true"
      onscroll="OnScroll"
      scrollWithAnimation
      style="height: calc(100vh - 100px);"
    >
      <slot></slot>
    </scroll-view>
  </view>
</template>

<script type="text/ecmascript-6">
export default {
  name: "scroll",
  props: {},
  data() {
    return {
      dargStyle: {
        top: "0px",
        transition: `all 3s`,
      },
    };
  },
  created() {},
  methods: {
    OnScroll() {
      dsfds
      console.log("OnScroll2");
    },
  },
  components: {},
};
</script>

<style lang="less">
.content {
  padding: 120px 0 80px 0;
}
</style>
