<template>
  <view
    class="panel"
    @touchmove.stop="touchmoveHandle"
    @touchstart.stop="touchstartHandle"
    @touchend.stop="touchendHandle"
    ref="panel"
    :style="{
      height: `${height}px`,
      transform: `translateY(${currentY})`,
      '--translate-end': animationStyles['--translate-end'],
      '--translate-start': animationStyles['--translate-start']
    }"
    :class="[{ 'box-radius': !isScroll }, { translate: animationType }]"
  >
    <view class="panel-content"> 
      <scroll-view
        ref="scroll"
        id="scroll"
        :scroll-y="isScroll"
        style="height: 100vh"
      >
        <slot></slot>
      </scroll-view>
    </view>
  </view>
</template>

<script>
export default {
  name: "OneExpansionPanel",
  props: {
    levelArray: {
      type: Array,
      default: () => ["0%", "20%", "70%"]
    },
    firstLevel: {
      type: String,
      default: "70%"
    }
  },
  data() {
    const fastMoveDelay = 200;
    return {
      fastMoveDelay,
      animationStyles: {},
      animationType: "",
      maxHeight: 1000,
      height: 0,
      currentY: this.firstLevel,
      lastY: 0,
      startY: 0,
      startTime: 0
    };
  },
  computed: {
    isScroll() {
      const maxLevel = this.levelArray[0];

      return (
        Number(this.currentY.replace("%", "")) <= maxLevel.replace("%", "")
      );
    }
  },
  created() {
    const info = uni.getSystemInfoSync();
    this.maxHeight = info.screenHeight - info.windowTop;
    this.height = this.maxHeight;
  },
  methods: {
    foldPanel() {
      const styles = {
        "--translate-start": this.currentY,
        "--translate-end": this.firstLevel
      };

      this.animationStyles = styles;
      this.animationType = "start";
      setTimeout(() => {
        this.currentY = styles["--translate-end"];
      }, 300);
    },
    touchstartHandle(event) {
      this.startTime = Date.now();
      this.animationType = "";
      this.lastY = event.touches[0].pageY;
      this.startY = this.currentY;
    },
    touchmoveHandle(event) {
      const { lastY, currentY, maxHeight, isScroll } = this;
      const { pageY } = event.touches[0];
      const { lastScrollTop = 0 } = this.$refs.scroll || {};
      // 如果是滚动状态，则不执行
      if (isScroll && lastScrollTop) return;
      const curPos = Number(currentY.replace("%", ""));
      let curNum = curPos - ((lastY - pageY) / maxHeight) * 100;
      if (curNum > 100) curNum = 100; // 限制最大值
      if (curNum < 0) curNum = 0; // 限制最小值
      this.currentY = curNum + "%";

      this.lastY = pageY;
    },
    touchendHandle(e) {
      const { fastMoveDelay, startTime, startY, isScroll } = this;
      const { lastScrollTop = 0 } = this.$refs.scroll || {};

      if (isScroll && lastScrollTop) return;

      const styles = {
        "--translate-start": this.currentY,
        "--translate-end": null
      };
      const percent = Number(this.currentY.replace("%", ""));
      //   判断是否快速上下滑动
      const now = Date.now();

      const levelArrayNumber = this.levelArray.map(o =>
        Number(o.replace("%", ""))
      );
      if (now - startTime <= fastMoveDelay) {
        const startPercent = Number(startY.replace("%", ""));
        if (percent == startPercent) {
          return;
        }
        const levelIndex = levelArrayNumber.findIndex(o => o == startPercent);
        // 快速滑动 上升或降低一个等级
        // 上滑
        if (startPercent - percent >= 0) {
          if (levelIndex !== 0) {
            styles["--translate-end"] = this.levelArray[levelIndex - 1];
          } else {
            styles["--translate-end"] = this.levelArray[0];
          }
        } else {
          // 下滑
          if (levelIndex !== this.levelArray.length - 1) {
            styles["--translate-end"] = this.levelArray[levelIndex + 1];
          } else {
            styles["--translate-end"] =
              this.levelArray[this.levelArray.length - 1];
          }
        }
      } else {
        if (percent <= levelArrayNumber[0]) {
          styles["--translate-end"] = this.levelArray[0];
        } else if (percent >= levelArrayNumber[levelArrayNumber.length - 1]) {
          styles["--translate-end"] =
            this.levelArray[this.levelArray.length - 1];
        } else {
          levelArrayNumber.reduce((pre, cur) => {
            if (percent > pre && percent < cur) {
              styles["--translate-end"] =
                (Math.abs(pre - percent) > Math.abs(cur - percent)
                  ? cur
                  : pre) + "%";
            }
            return cur;
          }, levelArrayNumber[0]);
        }
      }

      this.animationType = "start";
      this.animationStyles = styles;
      setTimeout(() => {
        this.currentY = styles["--translate-end"];
      }, 300);
    }
  }
};
</script>

<style scoped>
.translate {
  animation: translate 0.3s ease-in-out forwards;
}
@keyframes translate {
  0% {
    transform: translateY(var(--translate-start));
  }
  100% {
    transform: translateY(var(--translate-end));
  }
}
.panel {
  z-index: 999;
  position: absolute;
  bottom: 0;
  top: 0;
  width: 100vw;
  background-color: #ffffff;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  height: 100vh;
}
.panel-content {
  width: 100%;
  height: 100%;
}
.box-radius {
  border-radius: 20rpx 20rpx 0 0;
  box-shadow: 0 -8rpx 30rpx rgba(0, 0, 0, 0.1);
}
</style>
