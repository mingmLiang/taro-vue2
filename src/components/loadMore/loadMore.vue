<template>
  <p
    class="showMore"
    v-if="!itemZero"
    @tap="changePageNum">
    {{ !haveUpgrade ? '加载中...' : (canLoadNextPage ? '点击加载更多...' : '没有更多数据') }}
  </p>
</template>

<script>
export default {
  name: 'loadMore',
  props: [
    // 页面数据有没有渲染好
    'haveUpgrade',
    // 是否可以加载下一页
    'canLoadNextPage',
    // 是否监控的数组长度为0
    'itemZero'
  ],
  methods: {
    changePageNum () {
      if (this.canLoadNextPage) {
        this.$emit('ajaxNext')
      }
    }
  }
}
</script>

<style lang="less" scoped>
  .showMore {
    height: 60px;;
    font-size: 15px;
    line-height: 60px;
    text-align: center;
    color: #47a7e4;
    background-color: #fff;
  }
</style>
