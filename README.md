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
  // 当组件更新完毕后才监听窗口变化
  updated () {
    this.updatedOk = true
  },
  computed: {
    imgHeightWidth () {
      return this.$refs.imgSelf.height / this.$refs.imgSelf.width
    }
  },
  methods: {
    imgCenterMove () {
      this.$nextTick(() => {
        let containerWidth = this.$el.offsetWidth
        let containerHeight = this.$el.offsetHeight
        let containerHeightWidth = containerHeight / containerWidth
        if (containerHeightWidth >= this.imgHeightWidth) {
          this.$refs.imgSelf.height = containerHeight
          let containerWidthHalf = this.$el.offsetWidth / 2
          let imgWidthHalf = this.$refs.imgSelf.width / 2
          let leftMove = imgWidthHalf - containerWidthHalf
          this.$refs.imgSelf.style.top = 0
          this.$refs.imgSelf.style.left = -leftMove + 'px'
        } else {
          this.$refs.imgSelf.width = containerWidth
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
2. 然后就是如果页面如果刚加载的时候就进行缩放页面就会出现找不到width,height等问题,但是页面还是正常,图片还是能动态更改,用了好长时间才发现是生命周期的问题counted函数是页面载入后就立即触发了,而本组件内的图片这时候尺寸更改后组件相对就会更新,所以可能取不到值,这时候就用了updated函数,当组件也更新完毕时监听窗口才能够触发,这时候就不会再出现那种情况了
3. 这是一个小坑,主要是methods里动态更新时忘记了this.$refs.imgSelf.style.top = 0, 就导致有时候当图片相对胖瘦改变的时候上下左右的位移还是之前的没有改变
这是暂时遇到的坑
