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
    eventListeners () {
      if (this.updatedOk) {
        this.timerDebounce(this.imgCenterMove)
      }
    },
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
