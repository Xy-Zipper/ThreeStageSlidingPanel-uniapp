<template>
  <view class="content">
    <view class="map">
    <amapTrace
        ref="map"
        amap-key="" 
        :list="list"
        @load-data="loadData"
        @update-trace-index="getTraceIndex"
      ></amapTrace>
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
import amapTrace from "@/components/amap-trace/index";
export default {
  components: {
    threeStageSlidingPanel,
    amapTrace
  },
  data() {
    return {
      list: [],
      traceIndex: -1
    };
  },
  onLoad() {},
  methods: { 
    loadData() {
      this.list = [
        {
          lng: "119.133338",
          lat: "25.391288"
        },
        {
          lng: "119.136493",
          lat: "25.389075"
        },
        {
          lng: "119.140767",
          lat: "25.387481"
        },
        {
          lng: "119.149414",
          lat: "25.386302"
        },
        {
          lng: "119.162543",
          lat: "25.390648"
        },
        {
          lng: "119.172423",
          lat: "25.409414"
        },
        {
          lng: "119.169637",
          lat: "25.425889"
        },
        {
          lng: "119.163303",
          lat: "25.442133"
        }
      ];

      this.$refs.map.play();
    },
    getTraceIndex(index) {
      this.traceIndex = index;
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

.map {
  width: 100%;
  height: 100%;
}

image {
  width: 100%;
  height: 1000px;
}
</style>
