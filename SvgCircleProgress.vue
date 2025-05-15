<template>
  <svg
    :height="realSize"
    :width="realSize"
    x-mlns="http://www.w3.org/200/svg"
    viewBox="0 0 100 100"
    class="circle-progress"
    :class="{ 'circle-progress--rtl': isRtl }"
  >
    <defs>
      <linearGradient :id="refRandomId" x1="0%" y1="0%" x2="0%" y2="100%">
        <stop v-for="(item, index) in backgroundStop" :key="index" :offset="item.key" :stop-color="item.value"></stop>
      </linearGradient>
    </defs>
    <circle
      :r="pathRadius"
      cx="50"
      cy="50"
      stroke="#F2F7FA"
      :stroke-width="pathStrokeWidth"
      :fill="pathBackgroundColor"
    />
    <circle
      v-for="(item, index) in arcArr"
      :key="index"
      :r="hoverRaduis"
      class="circle-progress__hover"
      cx="50"
      cy="50"
      :stroke="item.color"
      :stroke-width="hoverStrokeWidth"
      :stroke-dasharray="item.strokeDasharray"
      :stroke-dashoffset="item.strokeDashoffset"
      fill="none"
      transform="rotate(-90)"
      transform-origin="center"
      stroke-linecap="round"
    >
    </circle>
  </svg>
</template>
<script>
export default {
  name: 'CircleProgress',
  props: {
    progress: { // 当前进度
      type: Number,
      required: true,
    },
    strokeWidth: { // 进度条容器的宽度，px值
      type: Number,
      default: 8
    },
    strokeBorder: { // 进度条内部与容器边框之间的距离（内和外），px值
      type: Number,
      default: 1
    },
    radius: { // 进度条的半径（包括strokeWidth的距离），px值
      type: Number,
      default: 68
    },
    startColor: { // 起始颜色rgb数组
      type: Array,
      default: () => [249, 221, 180]
    },
    endColor: { // 结束颜色rgb数组
      type: Array,
      default: () => [238, 171, 86]
    },
    backgroundColor: { // 进度条内部圆形的背景颜色，可设置渐变
      type: [String, Object],
      default: () => ({
        '0%': '#FFF8F8',
        '100%': '#FFE4CE'
      })
    },
    isRtl: { // 是否反转
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      step: 30, // 进度条渐变色分段数
      refRandomId: Math.random().toString(36).substring(2, 11)
    };
  },
  computed: {
    diameter() { // 进度条容器的直径，px值
      return this.radius * 2;
    },
    pathStrokeWidth() { // 进度条容器的相对宽度（以100为基准）
      return (this.strokeWidth / this.diameter) * 100;
    },
    pathRadius() { // 进度条容器的相对半径（以100为基准）
      return 50 - (this.pathStrokeWidth / 2);
    },
    pathStokeBorder() { // 进度条容器的相对边框距离（以100为基准）
      if (this.strokeBorder === 0) return 0;
      const minBorder = (1 / this.diameter) * 100; // 如果设置了border，则border最少要一个像素点
      const border = (this.strokeBorder / this.diameter) * 100;
      return Math.max(minBorder, border);
    },
    hoverStrokeWidth() { // 进度条内部相对宽度（以100为基准）
      return ((this.strokeWidth / this.diameter) * 100) - this.pathStokeBorder * 2;
    },
    hoverRaduis() { // 进度条内部相对半径（以100为基准）
      return 50 - (this.hoverStrokeWidth / 2) - this.pathStokeBorder;
    },
    pathBackgroundColor() {
      return this.isObject(this.backgroundColor) ? `url(#${this.refRandomId})` : this.backgroundColor;
    },
    backgroundStop() {
      if (!this.isObject(this.backgroundColor)) {
        return [];
      }

      const colorArr = Object.keys(this.backgroundColor).sort((a, b) => parseFloat(a) - parseFloat(b));
      const stopArr = [];
      colorArr.forEach((item) => {
        const obj = {
          key: '',
          value: ''
        };
        obj.key = item;
        obj.value = this.backgroundColor[item];
        stopArr.push(obj);
      });
      return stopArr;
    },
    arcArr() {
      if (this.progress === 0) {
        return [];
      }
      const circleLength = Math.floor(2 * Math.PI * this.hoverRaduis);
      const progressLength = this.progress * circleLength;
      const gradientColor = (startRGB, endRGB, step) => {
        const startR = startRGB[0];
        const startG = startRGB[1];
        const startB = startRGB[2];
        const endR = endRGB[0];
        const endG = endRGB[1];
        const endB = endRGB[2];
        const sR = (endR - startR) / (step - 1); // 总差值
        const sG = (endG - startG) / (step - 1);
        const sB = (endB - startB) / (step - 1);
        const colorArr = [];
        for (let i = 0; i < step; i += 1) {
          const color = `rgb(${sR * i + startR},${sG * i + startG},${sB * i + startB})`;
          colorArr.push(color);
        }
        return colorArr;
      };
      const colorArr = gradientColor(this.startColor, this.endColor, this.step);
      // 计算每个步进中的弧长
      const stepLength = progressLength / this.step;
      return colorArr.map((color, index) => {
        const arcLength = (index + 1) * stepLength;
        return {
          strokeDasharray: `${stepLength},${circleLength}`,
          strokeDashoffset: `-${arcLength - stepLength}`,
          color
        };
      });
    },
    realSize() {
      return this.fixPx(this.diameter);
    }
  },
  methods: {
    // 移动端很多页面都是用rem方案做多设备的适配，样式文件中使用设计稿的px值通常会使用postcss自动转成rem，但props里传入的设计稿里的像素值不会自动转换，
    // 所以可以在这里按比例计算一下设计稿里的像素值在每个页面实际要显示的像素大小（模拟rem转换），当然也可以直接使用props里传入的px值，不进行转换
    fixPx(px) {
      // const scale = (window.innerWidth > 540 ? 540 : window.innerWidth) / 720;
      // return px * scale;
      return px;
    },
    isObject(obj) {
      return obj !== null && typeof obj === 'object';
    }
  }
};
</script>
<style lang='scss' scoped>
.circle-progress {
  position: relative;

  &.circle-progress--rtl {
    transform: scaleX(-1);
  }

  .circle-progress__hover {
    transition:
      stroke-dashoffset 0.3s, stroke-dasharray 0.3s, stroke 0.3s, stroke-width 0.06s, opacity;
  }
}
</style>
