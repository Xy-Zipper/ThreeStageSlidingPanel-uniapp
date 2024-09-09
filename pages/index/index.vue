<template>
  <view class="content">
    <view class="map">
      <button @click="onClick">最小高度</button>
    </view>
    <threeStageSlidingPanel ref="panel">
      <view class="news" v-for="item in 10">
        <image src="/static/content.png" mode="aspectFill"></image>
      </view>
    </threeStageSlidingPanel>
  </view>
</template>

<script>
import threeStageSlidingPanel from "@/components/xycoder-threeStageSlidingPanel/index";
export default {
  components: {
    threeStageSlidingPanel
  },
  data() {
    return {
      title: "Hello"
    };
  },
  onLoad() {},
  methods: {
    onClick() {
      this.$refs.panel.foldPanel();
    }
  }
};
</script>

<style>
page {
  height: 100%;
  display: flex;
}

.content {
  flex: 1;
  display: flex;
  flex-direction: column;
  width: 100vw;
  overflow: hidden;
  background-color: #e8ecf3;
  position: relative;
}
.news {
  width: 100%;
}
image {
  width: 100%;
  height: 1000px;
}
</style>
