<template>
  <div v-if="isShow" :class="b()">
    <div v-if="ops.modal" :class="b('mask')" @click="close"></div>
    <span class="el-image-viewer__btn el-image-viewer__close" @click="close">
      <i class="el-icon-circle-close"></i>
    </span>
    <span v-if="isRrrow" class="el-image-viewer__btn el-image-viewer__prev" @click="handlePrev()">
      <i class="el-icon-arrow-left"></i>
    </span>
    <span v-if="isRrrow" class="el-image-viewer__btn el-image-viewer__next" @click="handleNext()">
      <i class="el-icon-arrow-right"></i>
    </span>
    <div ref="box" :class="b('box')">
      <component
        :is="carouselName"
        ref="carousel"
        :show-indicators="false"
        :initial-index="index"
        :initial-swipe="index"
        :interval="0"
        arrow="never"
        indicator-position="none"
        :height="height"
        @change="handleChange"
      >
        <component
          :is="carouselItemName"
          v-for="(item, indexs) in datas"
          :key="indexs"
          @click.native.self="ops.closeOnClickModal ? close() : ''"
        >
          <!--          <div v-else-->
          <!--               :class="b('file')">-->
          <!--            <a :href="item.url"-->
          <!--               target="_blank">-->
          <!--              <i class="el-icon-document"></i>-->
          <!--              <p>{{item.name}}</p>-->
          <!--            </a>-->
          <!--          </div>-->
        </component>
      </component>
    </div>
    <div class="el-image-viewer__btn el-image-viewer__actions">
      <div class="el-image-viewer__actions__inner">
        <i class="el-icon-zoom-out" @click="subScale"></i>
        <i class="el-icon-zoom-in" @click="addScale"></i>
        <!-- <i class="el-image-viewer__actions__divider"></i> -->
        <!-- <i class="el-icon-full-screen"></i> -->
        <!-- <i class="el-image-viewer__actions__divider"></i> -->
        <i class="el-icon-refresh-left" @click="rotate = rotate - 90"></i>
        <i class="el-icon-refresh-right" @click="rotate = rotate + 90"></i>
      </div>
    </div>
  </div>
</template>
<script>
import create from 'core/create'
export default create({
  name: 'image-preview',
  data() {
    return {
      left: 0,
      top: 0,
      scale: 1,
      datas: [],
      rotate: 0,
      isShow: false,
      index: 0,
      onClose: null,
    }
  },
  computed: {
    carouselName() {
      if (this.$isVan) return `${this.$GRID.ui.type}Swipe`
      return `${this.$GRID.ui.type}Carousel`
    },
    carouselItemName() {
      if (this.$isVan) return `${this.$GRID.ui.type}SwipeItem`
      return `${this.$GRID.ui.type}CarouselItem`
    },
    styleBoxName() {
      return {
        marginLeft: this.setPx(this.left),
        marginTop: this.setPx(this.top),
      }
    },
    styleName() {
      return {
        transform: `scale(${this.scale}) rotate(${this.rotate}deg)`,
        maxWidth: '100%',
        maxHeight: '100%',
      }
    },
    isRrrow() {
      return this.imgLen != 1
    },
    imgLen() {
      return this.imgList.length
    },
    imgList() {
      return this.datas.map((ele) => ele.url)
    },
  },
  methods: {
    handlePrev() {
      this.$refs.carousel.prev()
      this.index = this.$refs.carousel.activeIndex
      this.stopItem()
    },
    handleNext() {
      this.$refs.carousel.next()
      this.index = this.$refs.carousel.activeIndex
      this.stopItem()
    },
    stopItem() {
      this.left = 0
      this.top = 0
      this.$refs.item.forEach((ele) => {
        ele.pause && ele.pause()
      })
    },
    getIsVideo(item) {
      if (this.$typeList.video.test(item.url) || item.type == 'video') {
        return { is: 'video' }
      }
      return {}
    },
    subScale() {
      if (this.scale != 0.2) {
        this.scale = parseFloat((this.scale - 0.2).toFixed(2))
      }
    },
    addScale() {
      this.scale = parseFloat((this.scale + 0.2).toFixed(2))
    },
    handleChange() {
      this.scale = 1
      this.rotate = 0
    },
    move(e) {
      //获取目标元素s
      //算出鼠标相对元素的位置
      let disX = e.clientX
      let disY = e.clientY
      let scale = 2
      document.onmousemove = (e) => {
        //鼠标按下并移动的事件
        //用鼠标的位置减去鼠标相对元素的位置，得到元素的位置
        let left = e.clientX - disX
        let top = e.clientY - disY
        disX = e.clientX
        disY = e.clientY
        //移动当前元素
        this.left = this.left + left * scale
        this.top = this.top + top * scale
      }
      document.onmouseup = (e) => {
        document.onmousemove = null
        document.onmouseup = null
      }
    },
    close() {
      this.isShow = false
      if (typeof this.ops.beforeClose == 'function') {
        this.ops.beforeClose(this.datas, this.index)
      }
      if (typeof this.onClose === 'function') {
        this.onClose(this)
      }
    },
  },
})
</script>
