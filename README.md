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
    let newImg = new Image()
    newImg.onload = () => {
      this.updatedOk = true
    }
    newImg.onerror = (err) => {
      console.log(err)
    }
    newImg.src = this.thisImgSrc
    this.$nextTick(() => {
      window.addEventListener('resize', this.eventListeners)
    })
  },
  destroyed () {
    window.removeEventListener('resize', this.eventListeners)
  },
  computed: {
    imgHeightWidth () {
      return this.$refs.imgSelf.height / this.$refs.imgSelf.width
    }
  },
  methods: {
    // 监听的回调函数
    eventListeners () {
      if (this.updatedOk) {
        this.timerDebounce(this.imgCenterMove)
      }
    },
    // 最基本的函数去抖,(后续会优化...)
    timerDebounce (cont) {
      clearTimeout(cont.tId)
      cont.tId = setTimeout(() => {
        this.imgCenterMove()
      }, 200)
    },
    imgCenterMove () {
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
## 使用方法(非常简单)
```
html
<ImgCenterMove :img-src="...">  //直接传入图片绑定路径到子组件
<ImgCenterMove img-src="...">  //这里路径为直接传入的非绑定

<script>
import ImgCenterMove from '../ImgCenterMove'  //先导入组件
export default {
  name: '#',
  components: {
    ImgCenterMove   //注册组件
  }
}
</script>
```
## 遇到的坑
1. 我做这个的坑主要是开始的时候性能消耗比较大,所以找到了关于函数节流和函数去抖的相关知识看了看,暂时是用的高程3里最基础的去抖,后期再进行优化
2. 这是一个小坑,主要是methods里动态更新时忘记了this.$refs.imgSelf.style.top = 0 和 this.$refs.imgSelf.width = this.$refs.imgSelf.height / this.imgHeightWidth, 就导致有时候当图片相对胖瘦改变的时候上下左右的位移还是之前的没有改变,图片还会变形.
3. 开始这个坑我没有发现,就是当页面加载完毕后,如果跳转了路由,再去缩放页面大小,虽然页面还是正常的,但是控制台就会报错,告诉说不能找到width属性什么的,后来才发现,是因为切换路由过后,上个页面的这个组件里的监听页面大小变化还没有关闭掉,但是数据已经替换掉了,所以就导致找不到属性的问题,因此我添加了destroyed钩子函数,用来当组件被销毁时将组件中的监听也一并销毁,就不会出现报错问题了.
## 如果你发现还有什么问题,还请及时反馈给我.
