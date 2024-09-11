<!-- 轨迹回放私有组件/设备详情-轨迹回放 -->
<template>
  <view
    class="trace-play"
    :mapId="mapId"
    :traceList="traceList"
    :isPlay="isPlay"
    :speedMultiple="speedMultiple"
    :progress="progress"
    :change:mapId="traceModule.getMapId"
    :change:traceList="traceModule.loadTrack"
    :change:isPlay="traceModule.toggleRenderPlay"
    :change:speedMultiple="traceModule.changeRenderSpeed"
    :change:progress="traceModule.changeProgress"
  >
    <view class="trace-map" :id="mapId"></view>
    <!-- 底部播放模块 -->
  <!--   <view class="map-tools">
      <view class="map-tools-inner">
        <uni-datetime-picker
          type="datetimerange"
          start="2019-01-01"
          :end="today"
          :model-value="range"
          :border="false"
          :clear-icon="false"
          hide-second
          @change="rangeChange"
        />
        <view class="between-center trace-player">
          <view class="both-center player-btn" @tap="togglePlay">
            <view class="play" v-show="!isPlay"></view>
            <view class="stop first-stop" v-show="isPlay"></view>
            <view class="stop" v-show="isPlay"></view>
          </view>
          <progress
            class="play-progress"
            :id="'progress-' + mapId"
            active
            active-mode="forwards"
            active-color="#28ADFE"
            background-color="#E8EEF7"
            :percent="progress.value"
            duration="5"
            @tap="handleTapProgress"
          />
          <view class="both-center player-btn speed-btn" @tap="changeSpeed"
            >x{{ speedMultiple }}</view
          >
        </view>
      </view>
    </view> -->
  </view>
</template>

<script>
// 日期格式化
const formatDate = ({
  date = new Date(),
  format = "yyyy/MM/dd hh:mm:ss"
} = {}) => {
  if (date !== "Invalid Date") {
    const o = {
      "M+": date.getMonth() + 1, // month
      "d+": date.getDate(), // day
      "h+": date.getHours(), // hour
      "m+": date.getMinutes(), // minute
      "s+": date.getSeconds(), // second
      "q+": Math.floor((date.getMonth() + 3) / 3), // quarter
      S: date.getMilliseconds() // millisecond
    };
    if (/(y+)/.test(format)) {
      format = format.replace(
        RegExp.$1,
        (date.getFullYear() + "").substr(4 - RegExp.$1.length)
      );
    }
    for (let k in o) {
      if (new RegExp("(" + k + ")").test(format)) {
        format = format.replace(
          RegExp.$1,
          RegExp.$1.length == 1
            ? o[k]
            : ("00" + o[k]).substr(("" + o[k]).length)
        );
      }
    }
    return format;
  }
  return "";
};

export default {
  name: "lk-amap-trace",
  data() {
    return {
      today: Date.now(),
      mapId: Math.random().toString(16).substring(2, 15) + Date.now(),
      listQuery: {
        start: formatDate({ format: "yyyy-MM-dd 00:00:00" }),
        end: formatDate({ format: "yyyy-MM-dd hh:mm:ss" })
      },
      isPlay: false, // 是否播放
      progress: {
        // 播放进度百分比
        value: 0,
        isClick: false // 用于解决手动改变进度条后播放进入死循环的问题
      },
      speedIndex: 0, // 当前播放倍速下标
      speedOptions: [1, 2, 3] // 播放倍速列表
    };
  },
  props: {
    // 轨迹数据
    list: {
      type: Array,
      default: null
    },
    // 轨迹数据经度字段
    lngKey: {
      type: String,
      default: "lng"
    },
    // 轨迹数据纬度字段
    latKey: {
      type: String,
      default: "lat"
    },
    // 轨迹数据时间字段
    timeKey: {
      type: String,
      default: "time"
    },
    // 是否启用高德安全密钥jscode
    useJscode: {
      type: Boolean,
      default: true
    },
    // 代理服务器域名或地址
    serviceHost: {
      type: String,
      default: ""
    },
    // 申请的高德地图Web端的key值
    amapKey: {
      type: String,
      required: true
    }
  },
  computed: {
    traceList({
      list,
      lngKey,
      latKey,
      timeKey,
      useJscode,
      serviceHost,
      amapKey
    }) {
      return {
        list,
        lngKey,
        latKey,
        timeKey,
        useJscode,
        serviceHost,
        amapKey
      };
    },
    // 当前播放速度倍数
    speedMultiple({ speedOptions, speedIndex }) {
      return speedOptions[speedIndex];
    },
    range() {
      return [this.listQuery.start, this.listQuery.end];
    }
  },
  mounted() {
    const { windowWidth } = uni.getSystemInfoSync(); // 可使用窗口宽度
    this.windowWidth = windowWidth;
    this.getProgressWidth();
  },
  methods: {
    // 获取播放进度条宽度
    getProgressWidth() {
      uni
        .createSelectorQuery()
        .select(`#progress-${this.mapId}`)
        .fields(
          {
            size: true
          },
          data => {
            if (data.width) {
              this.progressWidth = data.width;
            } else {
              this.$nextTick(this.getProgressWidth);
            }
          }
        )
        .exec();
    },
    mapReady() {
      this.$emit("load-data", this.listQuery);
    },
    rangeChange(data) {
      this.listQuery.start = data[0] + ":00";
      this.listQuery.end = data[1] + ":59";
      this.progress = {
        value: 0,
        isClick: false
      };
      this.mapReady();
    },
    // 播放/暂停
    togglePlay() {
      if (!this.list || !this.list.length) {
        return;
      }
      this.isPlay = !this.isPlay;
    },
    pausePlay() {
      this.isPlay = false;
    },
    play() {
      this.isPlay = true;
    },
    // 切换播放速度
    changeSpeed() {
      if (this.speedIndex === this.speedOptions.length - 1) {
        this.speedIndex = 0;
      } else {
        this.speedIndex += 1;
      }
    },
    updateProgress(progress) {
      if (!progress.isClick) {
        this.progress = progress;
        this.getProgressIndex(progress.value / 100);
      }
    },
    // 获取当前播放进度对应的节点下标
    getProgressIndex(decimal) {
      const index = parseInt(decimal * this.list.length);
      this.$emit(
        "update-trace-index",
        index >= this.list.length ? this.list.length - 1 : index
      );
    },
    // 点击进度条
    handleTapProgress(e) {
      if (!this.list || !this.list.length) {
        return;
      }
      const isPlay = this.isPlay;
      if (isPlay) {
        this.isPlay = false;
      }
      const pointX = +e.detail.x; // 点击点距离屏幕左侧距离
      const progressWidth = this.progressWidth; // 进度条总宽度
      const leftDistance = (this.windowWidth - progressWidth) / 2; // 进度条距离屏幕左侧距离（进度条位于屏幕水平中间）
      const progressX = pointX - leftDistance; // 点击点距离进度条左侧距离
      const decimal = (progressX / progressWidth).toFixed(2);
      this.progress = {
        value: decimal * 100,
        isClick: true
      };
      // 获取当前进度节点对应的经纬度数据
      this.getProgressIndex(decimal);
      this.isPlay = isPlay;
    }
  }
};
</script>

<script module="traceModule" lang="renderjs">
export default {
  data() {
    return {
      speed: 1000 // 巡航速度，单位千米/小时
    };
  },
  methods: {
    getMapId(id) {
      this.mapId = id
    },
    // 引入高德地图
    loadAmap(useJscode, serviceHost, amapKey) {
      if (typeof window.AMap === 'function') {
        this.initMap()
      } else {
        if (!amapKey) {
          this.$ownerInstance.callMethod('getError', '缺少高德地图Web端的key值')
          return
        }
        if (useJscode) {
          // 搭配代理服务器并携带安全密钥转发
          window._AMapSecurityConfig = {
            serviceHost: serviceHost + '/_AMapService' // 例如 ：serviceHost:'http://1.1.1.1:80/_AMapService'
          }
        }
        // 动态引入较大类库避免影响页面展示
        const script = document.createElement('script')
        script.src = 'https://webapi.amap.com/maps?v=2.0&key=' + amapKey
        script.onload = this.initMap
        document.head.appendChild(script)
      }
    },
    // 初始化地图
    initMap() {
      if (!this.map && window.AMap) {
        this.map = new AMap.Map(this.mapId, {
          resizeEnable: true,
          zoom: 4
        })
        this.map.on('complete', () => this.center = this.map.getCenter())
      }
      if (window.AMap && !window.AMapUI) {
        const uiScript = document.createElement('script')
        uiScript.src = 'https://webapi.amap.com/ui/1.1/main.js?v=1.1.1'
        uiScript.onload = () => this.$ownerInstance.callMethod('mapReady')
        document.head.appendChild(uiScript)
      }
    },
    // 加载轨迹播放器
    initPlayer({list, lngKey, latKey, timeKey, useJscode, serviceHost, amapKey}) {
      AMapUI.loadUI(['overlay/SvgMarker'], SvgMarker => {
        if (!SvgMarker.supportSvg) { // 当前环境并不支持SVG，此时SvgMarker会回退到父类，即SimpleMarker
          console.error('当前环境不支持SVG')
        }
        // 创建结束点图标
        const shape = new SvgMarker.Shape['Circle']({
          height: 20, // 尺寸
          strokeWidth: 10, // 光环大小
          strokeColor: 'rgba(62,155,248,0.5)',
          fillColor: '#3e9bf8'
        })
        const labelCenter = shape.getCenter()
        new SvgMarker(
          shape,
          {
            map: this.map,
            position: new AMap.LngLat(list[0][lngKey], list[0][latKey]),
            containerClassNames: 'shape-Circle',
            showPositionPoint: false
          }
        )
      })
      AMapUI.load(['ui/misc/PathSimplifier'], PathSimplifier => {
        if (!PathSimplifier.supportCanvas) {
          console.error('当前环境不支持 Canvas！')
          return
        }
        this.pathSimplifierIns = new PathSimplifier({
          zIndex: 300,
          autoSetFitView: false, // 设置为true有时导致轨迹绘制失败
          map: this.map, // 所属的地图实例
          getPath: pathData => pathData.path,
          renderOptions: {
            renderAllPointsIfNumberBelow: -1, // 绘制路线节点，如不需要可设置为-1
            pathLineStyle: {
              dirArrowStyle: true
            },
            getPathStyle: (pathItem, zoom) => {
              const lineWidth = Math.round(2 * Math.pow(1.1, zoom - 3))
              return {
                pathLineStyle: {
                  strokeStyle: '#44d7b6',
                  lineWidth
                },
                pathLineSelectedStyle: {
                  lineWidth: lineWidth + 2
                },
                pathNavigatorStyle: {
                  fillStyle: '#3e99fb'
                }
              }
            }
          }
        })
        // 设置数据
        this.pathSimplifierIns.setData([{
          name: '轨迹',
          path: list.map(item => [item[lngKey], item[latKey]])
        }])
        // 对第一条线路（即索引 0）创建一个巡航器
        this.trackObj = this.pathSimplifierIns.createPathNavigator(0, {
          loop: false, // 循环播放
          speed: this.speed * this.speedTimes
        })
        const trackObj = this.trackObj
        // 监听轨迹播放
        trackObj.on('move', () => {
          const cursor = trackObj.getCursor()
          const index = cursor.idx
          const totalIdx = index + cursor.tail
          let progress = totalIdx * 100 / (this.traceData.length || 1)
          if (totalIdx === this.traceData.length - 1 && progress < 100) { // 解决轨迹动画播放完成时进度条没有滑动到终点的问题
            progress = 100
          }
          this.$ownerInstance.callMethod('updateProgress', {value: progress, isClick: this.progressChanged})
        })
        // 监听暂停播放
        trackObj.on('pause', () => {
          // #ifdef H5
          this.isPlay = false
          // #endif
          // #ifdef APP-PLUS
          this.$ownerInstance.callMethod('pausePlay')
          // #endif
        })
        this.map.setZoomAndCenter(18, [list[list.length - 1][lngKey], list[list.length - 1][latKey]])
        this.traceData = list
        trackObj.moveToPoint(this.traceData.length - 1)
      });
    },
    // 加载轨迹
    loadTrack({list, lngKey, latKey, timeKey, useJscode, serviceHost, amapKey}) {
      if (list.length) {
        if (this.pathSimplifierIns) { // 重新绘制
          this.map.clearMap()
          this.pathSimplifierIns.setData({
            name: '轨迹',
            path: []
          })
          this.trackObj.destroy()
        }
        if (list.length) {
          this.initPlayer({list, lngKey, latKey, timeKey})
        } else {
          this.map.setZoomAndCenter(4, this.center)
        }
      } else {
        this.loadAmap(useJscode, serviceHost, amapKey)
      }
    },
    // 播放/暂停
    toggleRenderPlay(isPlay) {
      if (!this.trackObj) {
        return
      }
      this.progressChanged = false
      const trackObj = this.trackObj
      this.isPlay = isPlay
      if (isPlay) {
        if (trackObj.isCursorAtPathStart() || trackObj.isCursorAtPathEnd()) {
          trackObj.start()
        } else {
          trackObj.resume()
        }
      } else {
        trackObj.pause()
      }
    },
    // 切换速度播放
    changeRenderSpeed(speedMultiple) {
      this.speedTimes = speedMultiple
      if (this.trackObj) {
        this.trackObj.setSpeed(this.speed * speedMultiple)
      }
    },
    // 手动改变播放进度
    changeProgress({value, isClick}) {
      this.progressChanged = isClick
      if (!this.trackObj || !isClick) {
        return
      }
      const trackObj = this.trackObj
      if (trackObj.isCursorAtPathStart() || trackObj.isCursorAtPathEnd()) {
        trackObj.start()
      } else {
        trackObj.resume()
      }
      if (value === 0) {
        trackObj.moveToPoint(0);
      } else if (value === 100) {
        trackObj.moveToPoint(this.traceData.length - 1)
      } else {
        const indexNum = value / 100 * this.traceData.length - 1
        const index = indexNum >= 0 ? Math.floor(indexNum) : 0
        const decimal = String(indexNum).split('.')[1] || 0
        trackObj.moveToPoint(index, +('0.' + decimal))
      }
      if (this.isPlay) {
        this.progressChanged = false // 解决在播放状态时手动改变进度条后进度条不再更新的问题
      } else { // 解决在停止状态时手动改变进度条后导航器会自动暂停播放的问题
        // #ifdef H5
        this.isPlay = false
        // #endif
        // #ifdef APP-PLUS
        this.$ownerInstance.callMethod('pausePlay')
        // #endif
        trackObj.pause()
      }
      this.pathSimplifierIns.renderLater()
    }
  }
}
</script>

<style lang="scss">
.trace-play {
  position: relative;
  width: 100%;
  height: 100%;
  .both-center {
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .between-center {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  .trace-map {
    width: 100%;
    height: 100%;
  }
  .map-tools {
    position: absolute;
    bottom: 0;
    z-index: 10;
    width: 100%;
    padding: 16rpx;
    .map-tools-inner {
      border-radius: 4px;
      padding: 0 32rpx;
      background: $uni-bg-color;
      box-shadow: 0 0 8px 0 rgba(80, 111, 140, 0.2);
      overflow: hidden;
      :deep(.uni-date-x) {
        border-radius: 0;
        padding: 16rpx 0;
        .icon-calendar {
          display: none;
        }
      }
      .trace-player {
        border-top: 1px solid #ebf2fb;
        height: 126rpx;
        .player-btn {
          border-radius: 4px;
          width: 64rpx;
          height: 64rpx;
          border: 1px solid #d2dded;
          .play {
            border-bottom: 18rpx solid transparent;
            border-top: 18rpx solid transparent;
            border-left: 24rpx solid $uni-color-success;
            border-right: 0 solid transparent;
            width: 0;
            height: 0;
            margin-left: 4rpx;
          }
          .stop {
            width: 6rpx;
            height: 36rpx;
            background-color: $uni-text-color-grey;
            &.first-stop {
              margin-right: 10rpx;
            }
          }
        }
        .play-progress {
          width: 430rpx;
        }
      }
    }
    .track_icon {
      width: 2vw;
      height: 2vw;
      margin-right: 12rpx;
    }
    .map-play-slider {
      flex: 1;
      margin-left: 0.5208vw;
    }
  }
}
</style>
