<template>
  <div class="notice">
    <div class="title">操作指南:</div>
    <br />
    可点击下方上传自定义图片，点击预览-> 下载,
    <br />
    图片过大的话生成 gif 需要点时间，请耐心等待几秒~
    <br />
    如果图片下载失败，可能浏览器不支持，可尝试手机自带浏览器或谷歌、火狐浏览器
    <strike>(等开发者修复)</strike>
  </div>
</template>

<script>
export default {
  name: 'Notice',
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="less" scoped>
.notice {
  .title {
    font-size: 16px;
    font-weight: 700;
  }
  text-align: left;
  font-size: 14px;
  padding: 0 20px;
  margin-bottom: 20px;
  line-height: 1.2;
}
</style>
