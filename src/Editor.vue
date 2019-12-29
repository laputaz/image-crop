<template>
  <div class="klk-pdf">
    <div class="klk-pdf-toolbar">
      <span class="toolbar__title">截图工具</span>
      <span v-if="imageUrl && mode === 'edit'" class="toolbar__icons">
        <el-tooltip effect="dark" content="截取" placement="top">
          <i v-show="!isCropping" class="el-icon-plus" @click="editImage"></i>
        </el-tooltip>
        <el-tooltip effect="dark" content="确认" placement="top">
          <i v-show="isCropping" class="el-icon-check" @click="confirmCrop"></i>
        </el-tooltip>
        <el-tooltip effect="dark" content="取消" placement="top">
          <i v-show="isCropping" class="el-icon-close" @click="cancelCrop"></i>
        </el-tooltip>
      </span>
      <span class="toolbar__input">
        <span>模板相似度&nbsp;</span>
        <el-input
          v-model="similarity"
          style="width: 90px;"
          size="small"
          placeholder
          type="number"
          min="0"
          max="100"
          :disabled="mode === 'view'"
        />%
      </span>
    </div>
    <div class="klk-pdf-main">
      <div class="klk-pdf-main__editor">
        <p class="klk-pdf__title">
          <!-- 模板 -->
          <!-- <span> -->
          <slot name="reupload"></slot>
          <!-- </span> -->
        </p>
        <div class="klk-pdf-main__editor__content">
          <slot v-if="!imageUrl">No Image Here.</slot>
          <transition name="fade">
            <Cropper
              v-if="imageUrl && showCrop"
              ref="cropper"
              :src="imageUrl"
              :background="false"
              :view-mode="2"
              :zoomable="true"
              :toggle-drag-mode-on-dblclick="false"
              :ready="onCropperReady"
            />
          </transition>
        </div>
      </div>
      <div class="klk-pdf-main__aside">
        <p class="klk-pdf__title"></p>
        <div class="klk-pdf-main__aside__content">
          <template v-if="imageUrl">
            <transition-group name="list">
              <template v-for="(op, i) in previewList">
                <div
                  v-if="isInCurrentPage(op)"
                  :key="op.title"
                  class="klk-pdf-main__aside__item"
                  :class="{ 'is-current': isCurrent(op) }"
                  @click="setCurrent(op)"
                >
                  <p class="item__title">
                    {{ op.title }}
                    <el-button
                      v-if="mode === 'edit'"
                      size="mini"
                      :plain="true"
                      type="danger"
                      @click.stop.prevent="deleteItem(op, i)"
                      >delete</el-button
                    >
                  </p>
                  <div class="item__coordinate">
                    <p style="margin: 0;">
                      {{ `start:${op.area_left_up_x} x ${op.area_left_up_y}` }}
                    </p>
                    <p>
                      {{
                        `end:${op.area_right_down_x} x ${op.area_right_down_y}`
                      }}
                    </p>
                  </div>
                  <div
                    class="item__preview"
                    :style="{ 'background-image': `url(${op.content})` }"
                  />
                  <p class="item__variables" @click.stop.prevent>
                    <span>变量名称</span>
                    <el-select
                      slot="prepend"
                      v-model="op.var_token"
                      size="small"
                      placeholder="请选择"
                      :disabled="mode === 'view'"
                    >
                      <el-option
                        v-for="vars in selectOptions"
                        :key="vars.key"
                        :label="vars.label"
                        :value="vars.key"
                      />
                    </el-select>
                    <el-input
                      v-model="op.var_name"
                      size="small"
                      placeholder="请输入内容"
                      :disabled="mode === 'view' || op.var_token !== ''"
                      clearable
                    />
                  </p>
                  <p class="item_similar">
                    <el-checkbox
                      v-model="op.need_check"
                      size="small"
                      :disabled="mode === 'view'"
                      >精确</el-checkbox
                    >
                    <el-input
                      v-model="op.check_value"
                      size="small"
                      :disabled="!op.need_check || mode === 'view'"
                    />
                  </p>
                </div>
              </template>
            </transition-group>
          </template>
          <p
            v-if="!imageUrl || !previewList.length"
            class="klk-pdf-main__aside__empty"
          >
            没有变量
          </p>
        </div>
      </div>
    </div>
    <div class="klk-pdf-pagination">
      <el-pagination
        small
        :page-size="1"
        :current-page="currentPage"
        layout="prev, pager, next"
        :total="total"
        @current-change="pageChange"
      />
    </div>
  </div>
</template>

<script>
// 裁剪组件
import Cropper from "./main";
const fontStyle = "bold 30px Helvetica";
const inactiveColor = "rgba(180,24,34,0.6)";
const activeColor = "rgba(55,202,59,0.6)";

export default {
  components: { Cropper },
  props: {
    // 预览 or 编辑  edit / view
    mode: {
      type: String,
      default: "edit"
    },
    // 预览 or 编辑模式需要传
    imageData: {
      type: Object,
      default: () => {}
    },
    // 相似度
    similar: {
      type: [Number, String],
      default: 100
    },
    // 变量
    list: {
      type: Array,
      default: () => []
    },
    // total
    total: {
      type: Number,
      default: 1
    },
    // 变量下拉框
    options: {
      type: Object,
      default: () => {}
    },
    // 当前页
    currentPage: {
      type: Number,
      default: 1
    }
  },
  data() {
    return {
      // 裁剪模式
      isCropping: false,
      // currentPage: 1,
      previewList: [],
      // 裁剪组件实例
      cropperInstance: { tag: true },
      currentEdit: "",
      showCrop: true
    };
  },
  computed: {
    // 相似度
    similarity: {
      get() {
        return this.similar;
      },
      set(val) {
        this.$emit("update:similar", val);
      }
    },
    // 图片URL
    imageUrl() {
      return this.imageData.content;
    },
    // 图片宽度
    naturalWidth() {
      return this.imageData.width;
    },
    // 图片高度
    naturalHeight() {
      return this.imageData.height;
    },
    // 下拉框选项
    selectOptions() {
      let options = this.options;
      let arr = this.enums.map(v => ({
        key: v,
        label: options[v]
      }));
      arr.unshift({
        key: "",
        label: "自定义变量"
      });
      return arr;
    },
    // 当前emus值
    enums() {
      return Object.keys(this.options) || [];
    }
  },
  watch: {
    imageUrl(next, old) {
      if (!this.cropperInstance.tag && next != old) {
        this.reload();
      }
    },
    list: {
      immediate: true,
      handler(next, old) {
        let list = [...this.list];
        list.forEach(v => {
          v.title = `变量区域${v.id}`;
          v.content = "";
          v.need_check = Number(v.need_check) === 1;
          v.area_left_up_x = Number(v.area_left_up_x);
          v.area_left_up_y = Number(v.area_left_up_y);
          v.area_right_down_x = Number(v.area_right_down_x);
          v.area_right_down_y = Number(v.area_right_down_y);
        });
        this.previewList = list;
        !this.cropperInstance.tag &&
          this.cropperInstance.replace(this.imageUrl, false);
      }
    }
  },
  methods: {
    editImage() {
      this.isCropping = true;
      this.cropperInstance.replace(this.currentImage, true);
      this.cropperInstance.setDragMode("crop");
    },
    // 当前选中
    isCurrent(op) {
      return op.title === this.currentEdit;
    },
    // 选择
    setCurrent(op) {
      if (this.isCurrent(op) || this.isCropping) return;
      this.currentEdit = op.title;
      this.generateMasks();
    },
    // 删除
    deleteItem(op) {
      let index = -1;
      let item = this.previewList.find((v, i) => {
        if (v.title === op.title) {
          index = i;
          return v;
        }
      });
      // 有id标识是原有, 不删除, 只是加delete标记, 否则删除
      if (item.hasOwnProperty("id")) {
        item.is_deleted = 1;
        this.previewList.splice(index, 1, item);
      } else {
        this.previewList.splice(index, 1);
      }
      this.generateMasks();
    },
    // 确认裁剪
    confirmCrop() {
      let data = this.cropperInstance.getData();
      if (!(data.x || data.y || data.height || data.width)) {
        this.setMoveStatus();
        return;
      }
      this.cropperInstance.replace(this.imageUrl, true);
      this.cropperInstance.setData(data);
      this.cropperInstance.$nextTick(_ => {
        let item = {
          title: `变量区域${this.previewList.length}`,
          var_name: "",
          area_left_up_x: data.x,
          area_left_up_y: data.y,
          area_right_down_x: data.x + data.width,
          area_right_down_y: data.y + data.height,
          content: this.cropperInstance.getDataURI("image/png", 1),
          cropData: data,
          var_page_no: this.currentPage,
          check_value: "",
          need_check: 0,
          var_token: undefined
        };
        this.setMoveStatus();
        this.setCurrent(item);
        this.previewList.push(item);
        this.generateMasks();
      });
    },
    // 取消
    cancelCrop() {
      this.setMoveStatus();
      this.generateMasks();
    },
    // 去除裁剪框
    setMoveStatus() {
      this.isCropping = false;
      this.cropperInstance.clear();
      this.cropperInstance.setDragMode("move");
    },
    // 生成遮罩
    async generateMasks(data, datauri) {
      let canvas = document.createElement("canvas");
      let context = canvas.getContext("2d", { alpha: false });
      let image = new Image();
      let src = this.imageUrl;
      canvas.width = this.naturalWidth;
      canvas.height = this.naturalHeight;
      await new Promise(resolve => {
        image.onload = resolve();
        image.src = this.imageUrl;
      });
      context.drawImage(image, 0, 0, this.naturalWidth, this.naturalHeight);
      this.previewList.forEach((v, i) => {
        if (!this.isInCurrentPage(v)) return;
        let data = {
          x: v.area_left_up_x,
          y: v.area_left_up_y,
          width: v.area_right_down_x - v.area_left_up_x,
          height: v.area_right_down_y - v.area_left_up_y
        };
        context.fillStyle = this.isCurrent(v) ? activeColor : inactiveColor;
        context.fillRect(data.x, data.y, data.width, data.height);
        context.font = fontStyle;
        context.fillStyle = "#fff";
        context.fillText(v.title, data.x + 10, data.y + 50);
      });
      src = canvas.toDataURL("image/png", 1);
      this.cropperInstance.replace(src, true);
      this.currentImage = src;
    },
    // Cropper渲染完毕
    onCropperReady() {
      // 记录crop实例
      this.cropperInstance = this.$refs.cropper;
      // 生成右侧previewList
      this.generatePreviewList();
    },
    // 是否在当前页
    isInCurrentPage(op) {
      return op.var_page_no === this.currentPage && !op.is_deleted;
    },
    // 获取裁剪的图片
    generatePreviewList() {
      let list = this.previewList.map(v => {
        if (!this.isInCurrentPage(v)) {
          return { ...v };
        }
        let data = {
          x: +v.area_left_up_x,
          y: +v.area_left_up_y,
          width: +v.area_right_down_x - +v.area_left_up_x,
          height: +v.area_right_down_y - +v.area_left_up_y
        };
        this.cropperInstance.setData(data);
        // 记录cropper需要的数据
        v.cropData = data;
        // 获取截图
        v.content = this.cropperInstance.getDataURI("image/png", 1);
        return { ...v };
      });
      this.previewList.forEach((v, i) => {
        this.$set(this.previewList, i, list[i]);
      });
      this.generateMasks();
      this.setMoveStatus();
    },
    // 页码改变
    pageChange(currentPage) {
      // this.currentPage = currentPage;
      this.$emit("current-page:update", currentPage);
      this.$emit("page-change", currentPage);
    },
    // 重加载裁剪框
    reload() {
      if (!this.cropperInstance) return;
      this.showCrop = false;
      this.cropperInstance.$nextTick(_ => {
        this.showCrop = true;
      });
    },
    // 获取数据
    getData() {
      if (!this.imageUrl) return [];
      let list = _.cloneDeep(this.previewList);
      return list.map(v => {
        let obj = { ...v };
        delete obj.content;
        delete obj.cropData;
        obj.need_check = +obj.need_check;
        obj.area_left_up_x = String(obj.area_left_up_x);
        obj.area_left_up_y = String(obj.area_left_up_y);
        obj.area_right_down_x = String(obj.area_right_down_x);
        obj.area_right_down_y = String(obj.area_right_down_y);
        return obj;
      });
    }
  }
};
</script>

<style lang="scss">
$toolbarBackground: #edf0f6;
$toolbarHeight: 50px;
$paginationHeight: 30px;
$editorWidth: 60%;
$mainBackgroundColor: #fff;
$titleHeight: 30px;

.klk-pdf {
  height: 100%;
  border: 1px solid darken($toolbarBackground, 5%);
  border-radius: 3px;
  // 状态栏
  .klk-pdf-toolbar {
    position: relative;
    height: $toolbarHeight;
    line-height: $toolbarHeight;
    background-color: $toolbarBackground;
    padding-left: 20px;
    border-bottom: 1px solid darken($toolbarBackground, 5%);
    .toolbar__title {
      font-size: 15px;
      font-weight: 500;
    }
    .toolbar__icons {
      text-indent: 10px;
      .el-icon-close {
        color: rgb(240, 84, 84);
        &:hover {
          color: rgb(240, 84, 84);
        }
      }
      .el-icon-check {
        color: rgb(39, 207, 151);
        &:hover {
          color: rgb(39, 207, 151);
        }
      }
      & > i {
        cursor: pointer;
        color: rgb(133, 133, 133);
        transition: 0.1s;
        font-size: 16px;
        &:hover {
          color: rgb(111, 152, 206);
        }
      }
    }
    .toolbar__input {
      position: absolute;
      right: 20px;
    }
  }
  // 主内容
  .klk-pdf-main {
    height: calc(100% - #{$toolbarHeight} - #{$paginationHeight});
    width: 100%;
    display: flex;
    .klk-pdf__title {
      font-size: 15px;
      font-weight: 600;
      margin: 0;
      line-height: $titleHeight;
    }
    // 编辑
    &__editor {
      width: $editorWidth;
      height: 100%;
      padding: 10px;
      background-color: $mainBackgroundColor;
      &__content {
        overflow: hidden;
        display: flex;
        height: calc(100% - #{$titleHeight});
        width: 100%;
        justify-content: center;
        align-items: center;
      }
      .cropper-modal {
        background-color: rgba(0, 0, 0, 0.05);
      }
    }
    // 侧栏
    &__aside {
      width: calc(100% - #{$editorWidth});
      height: 100%;
      background-color: #fbfafa;
      border-radius: 4px;
      border-left: 1px solid #eee;
      padding: 10px;
      padding-right: 0;
      &__content {
        overflow: auto;
        overflow-x: hidden;
        // height: calc(100% - #{$titleHeight});
        height: 100%;
      }
      &__empty {
        height: calc(100% - 30px);
        display: flex;
        align-items: center;
        justify-content: center;
      }
      &__item {
        position: relative;
        width: calc(100% - 10px);
        padding: 15px;
        border: 1px solid #eaeaea;
        border-radius: 3px;
        max-height: 500px;
        margin: 5px;
        background-color: #fff;
        transition: all 0.2s;
        &.is-current {
          background-color: rgb(251, 253, 254);
          border: 1px solid rgba(33, 150, 243, 0.33);
          box-shadow: 0 0 6px 0px #c2c2c247;
        }
        &:hover {
          cursor: pointer;
          box-shadow: 0 0 6px 0px #c2c2c247;
          background-color: rgba(171, 205, 231, 0.05);
        }
        .item__title {
          height: 20px;
          line-height: 20px;
          font-size: 15px;
          font-weight: 500;
          .el-button,
          span {
            float: right;
            cursor: pointer;
          }
        }
        .item__coordinate {
          height: 20px;
          line-height: 20px;
        }
        .item__preview {
          min-height: 150px;
          margin: 20px 0;
          background-size: contain;
          background-position: center;
          background-repeat: no-repeat;
        }
        .item__variables {
          display: grid;
          grid-template-columns: 80px 30% 40%;
          height: 30px;
          line-height: 30px;
        }
        .item_similar {
          display: grid;
          grid-template-columns: 80px 70%;
          height: 30px;
          line-height: 30px;
        }
      }
    }
  }
  // 分页
  .klk-pdf-pagination {
    // padding: 10px 0;
    height: $paginationHeight;
    display: flex;
    background-color: #fff;
    justify-content: center;
    align-items: center;
  }
  // animation
  .list-enter-active {
    transition: all 0.3s;
  }
  /** 移除过程 **/
  .list-leave-active {
    transition: all 0.3s;
  }
  /*** 开始插入、移除结束的位置变化 ***/
  .list-enter,
  .list-leave-to {
    opacity: 0;
    margin: 0 5px;
    max-height: 0;
    padding: 0 10px;
    transform: translateY(30px);
  }
}
</style>
