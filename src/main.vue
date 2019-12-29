<template>
  <div class="klk-cropper">
    <img :id="id" :src="src" />
  </div>
</template>
<script>
import Cropper from "cropperjs";
import "cropperjs/dist/cropper.css";
import _ from "lodash";

export default {
  name: "Cropper",
  props: {
    // 用于选择图片
    id: {
      type: String,
      default: () => _.uniqueId("klk_cropper_")
    },
    // 图片src 支持 base64, BlobURL, 路径
    src: {
      type: String,
      default: ""
    },
    /**
     * 0: 无限制
     * 1: 限制裁剪框不超过画布的大小
     * 2: 按照长或宽, 将最小画布大小限制为适合容器
     * 3: 限制最小画布大小以填充容器
     * 如果画布和容器的比例相同，则2和3之间没有差异
     */
    viewMode: {
      type: Number,
      default: 0
    },
    /**
     * 指定鼠标拖拽动作
     * crop: 裁剪
     * move: 移动图片
     */
    dragMode: {
      type: String,
      default: "crop"
    },
    /**
     * 初始的裁剪框比例, 不传时, 默认和图片的横纵比一样
     */
    initialAspectRatio: {
      type: Number,
      default: undefined
    },
    /**
     * 裁剪框比例, 不传则可以自由拖拽改变裁剪框比例, 定义之后裁剪框比例将固定
     */
    aspectRatio: {
      type: Number,
      default: undefined
    },
    /**
     * 裁剪框位置数据, x,y,w,h
     */
    data: {
      type: Object,
      default: null
    },
    preview: {
      type: [Object, String],
      default: ""
    },
    /**
     * 调整窗口大小时重新渲染
     */
    responsive: {
      type: Boolean,
      default: true
    },
    // 调整窗口大小后恢复裁剪区域
    restore: {
      type: Boolean,
      default: true
    },
    // 是否跨域
    checkCrossOrigin: {
      type: Boolean,
      default: true
    },
    checkOrientation: {
      type: Boolean,
      default: true
    },
    // 是否显示黑色遮罩
    modal: {
      type: Boolean,
      default: true
    },
    // 是否显示虚线框
    guides: {
      type: Boolean,
      default: true
    },
    // 是否显示中心点
    center: {
      type: Boolean,
      default: true
    },
    // 是否突出显示裁剪框, 即裁剪内容高亮
    highlight: {
      type: Boolean,
      default: true
    },
    // 是否显示网格背景
    background: {
      type: Boolean,
      default: true
    },
    // 初始化时是否自动裁剪
    autoCrop: {
      type: Boolean,
      default: true
    },
    // 自动裁剪的比例大小 介于0和1之间的数字, 定义自动裁剪区域大小（百分比）
    autoCropArea: {
      type: Number,
      default: 0.8
    },
    // 是否允许移动图片
    movable: {
      type: Boolean,
      default: true
    },
    // 是否可旋转
    rotatable: {
      type: Boolean,
      default: true
    },
    // 是否可对称转换
    scalable: {
      type: Boolean,
      default: true
    },
    // 是否可缩放
    zoomable: {
      type: Boolean,
      default: true
    },
    // 触摸缩放
    zoomOnTouch: {
      type: Boolean,
      default: true
    },
    // 鼠标滚轮缩放
    zoomOnWheel: {
      type: Boolean,
      default: true
    },
    // 鼠标滚轮缩放比例
    wheelZoomRatio: {
      type: Number,
      default: 0.1
    },
    // 裁剪框是否可移动
    cropBoxMovable: {
      type: Boolean,
      default: true
    },
    // 裁剪框是否可缩放
    cropBoxResizable: {
      type: Boolean,
      default: true
    },
    // 双击切换当前模式 移动图片 or 裁剪框
    toggleDragModeOnDblclick: {
      type: Boolean,
      default: true
    },

    // 尺寸控制 ---------------------------
    // 最小图片宽度
    minCanvasWidth: {
      type: Number,
      default: 0
    },
    // 最小图片高度
    minCanvasHeight: {
      type: Number,
      default: 0
    },
    // 最小裁剪框宽度
    minCropBoxWidth: {
      type: Number,
      default: 0
    },
    // 最小裁剪框高度
    minCropBoxHeight: {
      type: Number,
      default: 0
    },
    // 最小容器宽度
    minContainerWidth: {
      type: Number,
      default: 0
    },
    // 最小容器高度
    minContainerHeight: {
      type: Number,
      default: 0
    },

    // 事件回调 ---------------------------
    // 加载完成事件
    ready: {
      type: Function,
      default: () => {}
    },
    // 开始裁剪
    cropstart: {
      type: Function,
      default: () => {}
    },
    // 拖放裁剪
    cropmove: {
      type: Function,
      default: () => {}
    },
    // 裁剪结束
    cropend: {
      type: Function,
      default: () => {}
    },
    // 裁剪
    crop: {
      type: Function,
      default: () => {}
    },
    // 缩放事件
    zoom: {
      type: Function,
      default: () => {}
    }
  },
  computed: {
    // 图片缩放比
    scalingRatio() {
      let imgData = this.cropper.getImageData();
      return imgData.width / imgData.naturalWidth;
    }
  },
  watch: {
    // 图片src改变时, 重新渲染
    src(next, old) {
      next !== old && this.replace(next);
    }
  },
  mounted() {
    this.init();
  },
  methods: {
    // 初始化
    init() {
      const image = document.getElementById(this.id);
      this.cropper = new Cropper(image, this.$props);
    },
    // 重置为初始状态
    reset() {
      return this.cropper.reset();
    },

    // 去除裁剪框
    clear() {
      return this.cropper.clear();
    },

    // 裁剪
    cropp() {
      return this.cropper.crop();
    },

    /**
     * 更新图片src
     * @param {string} src - 新的图片路径
     * @param {boolean} [hasSameSize] - true标识尺寸是否一样, 此时不需要重新渲染cropper
     * @returns {Object} cropper对象
     */
    replace(src, hasSameSize = false) {
      return this.cropper.replace(src, hasSameSize);
    },

    // 允许裁剪
    enable() {
      return this.cropper.enable();
    },

    // 禁止裁剪 disable
    disable() {
      return this.cropper.disable();
    },

    // 销毁
    destroy() {
      return this.cropper.destroy();
    },

    /**
     * 移动裁剪框
     * @param {number} offsetX x偏移量
     * @param {number} offsetY y偏移量
     * @returns {Object} cropper对象
     */
    move(offsetX, offsetY) {
      return this.cropper.move(offsetX, offsetY);
    },

    /**
     * 移动裁剪框到指定位置
     * @param {number} x x位置
     * @param {number} [y=x] y位置
     * @returns {Object} cropper对象
     */
    moveTo(x, y = x) {
      return this.cropper.moveTo(x, y);
    },

    /**
     * 缩放图片(相对)
     * @param {number} ratio 缩放比例(相对当前)
     * @param {Event} _originalEvent 事件对象
     * @returns {Object} cropper对象
     */
    relativeZoom(ratio, _originalEvent) {
      return this.cropper.zoom(ratio, _originalEvent);
    },

    /**
     * 缩放图片
     * @param {number} ratio - 缩放比例(相对原始比例)
     * @param {Event} _originalEvent 事件对象
     * @returns {Object} cropper对象
     */
    zoomTo(ratio, _originalEvent) {
      return this.cropper.zoomTo(ratio, _originalEvent);
    },

    /**
     * 旋转(相对)
     * @param {number} degree 旋转角度(相对当前)
     * @returns {Object} cropper对象
     */
    rotate(degree) {
      return this.cropper.rotate(degree);
    },

    /**
     * 旋转
     * @param {number} degree - 旋转角度(相对原始)
     * @returns {Object} cropper对象
     */
    rotateTo(degree) {
      return this.cropper.rotateTo(degree);
    },

    /**
     * 对称变换 X
     * @param {number} scaleX
     * @returns {Object} cropper对象
     */
    scaleX(scaleX) {
      return this.cropper.scaleX(scaleX);
    },

    /**
     * 对称变换 Y
     * @param {number} scaleY
     * @returns {Object} cropper对象
     */
    scaleY(scaleY) {
      return this.cropper.scaleY(scaleY);
    },

    /**
     * 变换
     * @param {number} scaleX
     * @param {number} [scaleY=scaleX]
     * @returns {Object} cropper对象
     */
    scale(scaleX, scaleY = scaleX) {
      return this.cropper.scale(scaleX, scaleY);
    },

    /**
     * 获取裁剪框的位置信息和裁剪的尺寸信息
     * @param {boolean} [rounded=false] - 是否四舍五入
     * @returns {Object} 位置信息对象
     * x: 裁剪框相对于图片左侧偏移量
     * y: 裁剪框相对于图片上侧偏移量
     * width: 裁剪宽度像素
     * height: 裁剪高度像素
     * rotate: 图片旋转的角度
     */
    getData(rounded = false) {
      return this.cropper.getData(rounded);
    },

    /**
     * 设置裁剪框的位置信息和裁剪的尺寸信息
     * @param {Object} data - 格式同getData获取的位置信息对象
     * @returns {Object} cropper对象
     */
    setData(data) {
      return this.cropper.setData(data);
    },

    /**
     * 获取容器数据
     * @returns {Object}
     * width: 容器宽度
     * height: 容器高度
     */
    getContainerData() {
      return this.cropper.getContainerData();
    },

    getImageData() {
      return this.cropper.getImageData();
    },

    /**
     * canvas 画布信息
     * @returns {Object}
     */
    getCanvasData() {
      return this.cropper.getCanvasData();
    },

    /**
     * Set the canvas position and size with new data.
     * @param {Object} data - The new canvas data.
     * @returns {Object} cropper对象
     */
    setCanvasData(data) {
      return this.cropper.setCanvasData(data);
    },

    /**
     * 获取裁剪框位置大小信息
     * @returns {Object}
     * left: the offset left of the crop box
     * top: the offset top of the crop box
     * width: the width of the crop box
     * height: the height of the crop box
     */
    getCropBoxData() {
      return this.cropper.getCropBoxData();
    },

    /**
     * Set the crop box position and size with new data.
     * @param {Object} data - The new crop box data.
     * @returns {Object} cropper对象
     */
    setCropBoxData(data) {
      return this.cropper.setCropBoxData(data);
    },

    /**
     * 获取裁剪的canvas
     * @param {Object} [options={}]
     */
    getCroppedCanvas(options = {}) {
      return this.cropper.getCroppedCanvas(options);
    },

    /**
     * 改变裁剪框比例
     * @param {number} aspectRatio
     * @returns {Object} cropper对象
     */
    setAspectRatio(aspectRatio) {
      return this.cropper.setAspectRatio(aspectRatio);
    },

    /**
     * 改变为拖拽模式or裁剪模式
     * @param {string} mode - move | drag
     * @returns {Object} cropper对象
     */
    setDragMode(mode) {
      return this.cropper.setDragMode(mode);
    },
    /**
     * 获取 base64 编码String
     * @param {string} mime - 图片格式 | image/webp | image/jpg | image/jpeg | image/png
     * @returns {string} base64
     */
    getDataURI(mime, quality = 1) {
      let cvs = this.getCroppedCanvas();
      return cvs.toDataURL(mime, quality);
    },
    /**
     * ?  获取 Blob ? 需不需要要转为 URL ?
     * @param {string} mime - 图片格式 | image/webp | image/jpg | image/jpeg | image/png
     * @returns {string} Blob or URL
     */
    async getBlobURL(mime, quality = 1) {
      let cvs = this.getCroppedCanvas();
      let objURL = await new Promise(resolve => {
        cvs.toBlob(
          blob => {
            resolve(URL.createObjectURL(blob));
          },
          mime,
          quality
        );
      });
      return objURL;
    }
  }
};
</script>
<style lang="scss">
.klk-cropper {
  width: 100%;
  height: 100%;
  > img {
    display: block;
    max-width: 100%;
  }
  .cropper-line,
  .cropper-point {
    background-color: transparent;
  }
  .cropper-modal {
    opacity: 0.7;
  }
}
</style>
