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
      :key="`${arcArr.length}_${index}`"
      :r="hoverRaduis"
      class="circle-progress__hover"
      :class="{ 'hover-transition': setTransition }"
      cx="50"
      cy="50"
      :stroke="item.color"
      :stroke-width="hoverStrokeWidth"
      :stroke-dasharray="item.strokeDasharray"
      :stroke-dashoffset="item.strokeDashoffset"
      :opacity="item.opacity"
      :style="{ transition: item.transition }"
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
      required: true
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
    startColor: { // 起始颜色
      type: String,
      default: '#FFC53D'
    },
    endColor: { // 结束颜色
      type: String,
      default: '#FF6C1E'
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
    },
    minStep: {
      type: Number,
      default: 10
    },
    maxStep: { // 进度条渐变色分段的最大数量
      type: Number,
      default: 100
    },
    setTransition: {
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      refRandomId: Math.random().toString(36).substring(2, 11)
    };
  },
  computed: {
    startColorRGB() {
      return this.extractRgbArray(this.startColor);
    },
    endColorRGB() {
      return this.extractRgbArray(this.endColor);
    },
    diameter() { // 进度条容器的直径，px值
      return this.radius * 2;
    },
    realSize() {
      return this.fixPx(this.diameter);
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
    diffColorRgb() {
      const startR = this.startColorRGB[0];
      const startG = this.startColorRGB[1];
      const startB = this.startColorRGB[2];
      const endR = this.endColorRGB[0];
      const endG = this.endColorRGB[1];
      const endB = this.endColorRGB[2];
      const diffR = endR - startR;
      const diffG = endG - startG;
      const diffB = endB - startB;
      return [diffR, diffG, diffB];
    },
    step() {
      const circleLength = Math.floor(2 * Math.PI * this.hoverRaduis);
      const [diffR, diffG, diffB] = this.diffColorRgb;
      const maxDiff = Math.max(Math.abs(diffR), Math.abs(diffG), Math.abs(diffB)) + 1;
      let step = maxDiff / 2.56;
      const stepRealSize = ((circleLength / step) * this.hoverStrokeWidth) * (this.realSize / 100 ** 2);
      const baseSize = 40;
      step = Math.ceil(step * Math.max((stepRealSize / baseSize), 1));
      step = Math.max(Math.min(step, this.maxStep), this.minStep);
      return step;
    },
    arcArr() {
      return this.getArcArr(this.step);
    }
  },
  methods: {
    getArcArr(step) {
      const circleLength = Math.floor(2 * Math.PI * this.hoverRaduis);
      const progressLength = this.progress * circleLength;
      const [diffR, diffG, diffB] = this.diffColorRgb;
      const sR = diffR / (step - 1); // 总差值
      const sG = diffG / (step - 1);
      const sB = diffB / (step - 1);
      const colorArr = [];
      for (let i = 0; i < step; i += 1) {
        const color = `rgb(${sR * i + this.startColorRGB[0]},${sG * i + this.startColorRGB[1]},${sB * i + this.startColorRGB[2]})`;
        colorArr.push(color);
      }
      // 计算每个步进中的弧长
      const stepLength = progressLength / step;
      return colorArr.map((color, index) => {
        const arcLength = (index + 1) * stepLength;
        return {
          strokeDasharray: `${stepLength},${circleLength}`,
          strokeDashoffset: `-${arcLength - stepLength}`,
          color,
          opacity: this.progress === 0 ? 0 : 1,
          transition: this.setTransition ? (this.progress === 0 ? 'stroke-dashoffset 0.3s, stroke-dasharray 0.3s, stroke 0.3s, opacity 0.4s' : '') : ''
        };
      });
    },
    // 移动端页面是用rem方案做多设备的适配，样式文件中使用设计稿的px值通常会使用postcss自动转成rem，但props里传入的设计稿里的像素值不会自动转换，
    // 所以可以在这里按比例计算一下设计稿里的像素值在每个页面实际要显示的像素大小（模拟rem转换），当然如果没使用到的话可以直接使用props里传入的px值，不进行转换
    fixPx(px) {
      const scale = (window.innerWidth > 540 ? 540 : window.innerWidth) / 720;
      return px * scale;
    },
    isObject(obj) {
      return obj !== null && typeof obj === 'object';
    },
    extractRgbArray(colorString) {
      // 移除所有空格
      const cleanColor = colorString.replace(/\s+/g, '');

      // 检查是否为HEX格式（支持3/4/6/8位）
      const hexMatch = cleanColor.match(/^#?([0-9A-F]{3,8})$/i);
      if (hexMatch) {
        let hex = hexMatch[1];
        // 处理3位和4位简写格式
        if (hex.length === 3 || hex.length === 4) {
          hex = hex.split('').map(c => c + c).join('');
        }
        // 提取RGB值（忽略Alpha通道）
        return [
          parseInt(hex.substring(0, 2), 16),
          parseInt(hex.substring(2, 4), 16),
          parseInt(hex.substring(4, 6), 16)
        ];
      }
      // 检查是否为RGB格式
      const rgbMatch = cleanColor.match(/^rgb\((\d+),(\d+),(\d+)\)$/);
      if (rgbMatch) {
        return [
          parseInt(rgbMatch[1], 10),
          parseInt(rgbMatch[2], 10),
          parseInt(rgbMatch[3], 10)
        ];
      }
      // 检查是否为RGBA格式（忽略alpha值）
      const rgbaMatch = cleanColor.match(/^rgba\((\d+),(\d+),(\d+),[^)]+\)$/);
      if (rgbaMatch) {
        return [
          parseInt(rgbaMatch[1], 10),
          parseInt(rgbaMatch[2], 10),
          parseInt(rgbaMatch[3], 10)
        ];
      }
      // 无效格式返回null
      return undefined;
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
    &.hover-transition {
      transition: stroke-dashoffset 0.3s, stroke-dasharray 0.3s, stroke 0.3s; 
    }
  }
}
</style>
