# vue-ImgCenterMove
vue组件,加载的图片全部都居中并且监听了窗口大小的变化-动态等比例缩放不失真
## 首先还是看一下源码
```
<template lang="pug">
  div.img-center
    img.center-image(:src="thisImgSrc", @load="imgCenterMove", ref="imgSelf")
</template>

<script type="text/javascript">
export default {
  name: 'ImgCenterMove',
  props: {
    imgSrc: String
  },
  data () {
    return {
      thisImgSrc: this.imgSrc,
      changeImgWidth: '',
      changeImgHeight: '',
      updatedOk: false
    }
  },
  mounted () {
    this.$nextTick(() => {
      if (this.updatedOk) {
        window.addEventListener('resize', () => {
          this.timerDebounce(this.imgCenterMove)
        })
      }
    })
  },
  computed: {
    imgHeightWidth () {
      return this.$refs.imgSelf.height / this.$refs.imgSelf.width
    }
  },
  methods: {
    // 当图片加载完毕后才可以调动监听窗口
    imgCenterMove () {
      if (!this.updatedOk) {
        this.updatedOk = true
      }
      this.$nextTick(() => {
        let containerWidth = this.$el.offsetWidth
        let containerHeight = this.$el.offsetHeight
        let containerHeightWidth = containerHeight / containerWidth
        if (containerHeightWidth >= this.imgHeightWidth) {
          this.$refs.imgSelf.height = containerHeight
          this.$refs.imgSelf.width = this.$refs.imgSelf.height / this.imgHeightWidth
          let containerWidthHalf = this.$el.offsetWidth / 2
          let imgWidthHalf = this.$refs.imgSelf.width / 2
          let leftMove = imgWidthHalf - containerWidthHalf
          this.$refs.imgSelf.style.top = 0
          this.$refs.imgSelf.style.left = -leftMove + 'px'
        } else {
          this.$refs.imgSelf.width = containerWidth
          this.$refs.imgSelf.height = this.$refs.imgSelf.width * this.imgHeightWidth
          let containerHeightHalf = this.$el.offsetHeight / 2
          let imgHeightHalf = this.$refs.imgSelf.height / 2
          let upMove = imgHeightHalf - containerHeightHalf
          this.$refs.imgSelf.style.left = 0
          this.$refs.imgSelf.style.top = -upMove + 'px'
        }
      })
    },
    // 函数去抖
    timerDebounce (cont) {
      clearTimeout(cont.tId)
      cont.tId = setTimeout(() => {
        this.imgCenterMove()
      }, 200)
    }
  }
}
</script>

<style lang="scss" scoped>
.img-center {
  position: relative;
  width: 100%;
  height: 100%;
  overflow: hidden;
  img.center-image {
    position: absolute;
  }
}
</style>
```
## 遇到的坑
1. 我做这个的坑主要是开始的时候性能消耗比较大,所以找到了关于函数节流和函数去抖的相关知识看了看,暂时是用的高程3里最基础的去抖,后期再进行优化
2. 这是一个小坑,主要是methods里动态更新时忘记了this.$refs.imgSelf.style.top = 0 和 this.$refs.imgSelf.width = this.$refs.imgSelf.height / this.imgHeightWidth, 就导致有时候当图片相对胖瘦改变的时候上下左右的位移还是之前的没有改变,图片还会变形.
这是暂时遇到的坑
