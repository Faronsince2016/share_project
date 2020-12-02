<template>
  <div class="feature-img-container">
    <img
      ref="img"
      class="img"
      :src="src"
      style="width: inherit;height: inherit;max-height: 750px;vertical-align: middle"
      @click.alt.exact.stop="altClick"
      @load="onImgLoad"
      @error="onError"
      alt="" />
    <div class="face-area" ref="faceArea" :style="{}"></div>
  </div>
</template>
<script>
// import $ from 'jQuery'
import { ResizeObserver } from 'vue-resize'

export default {
  name: 'FeatureImg',
  components: { ResizeObserver },
  data () {
    return {
      init: false,
      observer: null
    }
  },
  props: {
    src: {
      type: String,
      default: '',
      required: true
    },
    // eslint-disable-next-line vue/require-default-prop
    cover: {
      required: false
    },
    position: {
      required: false
    }
  },
  activated () {
    window.requestIdleCallback(this.onImgLoad)
  },
  methods: {
    handleResize () {
      if (!this.init) return
      if (this.idleHandle) window.cancelIdleCallback(this.idleHandle)
      this.idleHandle = window.requestIdleCallback(() => {
        this.onImgLoad()
        this.idleHandle = null
      })
    },
    onImgLoad (e) {
      if (!this.src) return
      if (e) {
        this.$emit('loaded', e)
        this.init = true
      }
      const cover = this.cover
      const $imgWrap = window.$(this.$refs.imgWrap)
      const { clientWidth, clientHeight } = this.$el
      if (!clientWidth && !clientHeight) return
      const { naturalWidth, naturalHeight } = this.$refs.img
      const containerRate = clientWidth / clientHeight
      const faceObj = this.Position || null
      try {
        const hh = clientHeight / naturalHeight
        const ww = clientWidth / naturalWidth
        const imgRate = naturalWidth / naturalHeight
        if (containerRate > imgRate) {
          $imgWrap.css({ transform: `translate(-50%, -50%) scale(${cover ? ww : hh})` })
          if (faceObj && cover) {
            const faceAreaH = (faceObj.Y + faceObj.Height) * ww // 图片方框底边高度
            const imgH = naturalHeight * ww // 显示的图片高度
            const hb = (clientHeight + imgH) / 2
            const ht = (imgH - clientHeight) / 2
            if (faceAreaH > hb - 2) {
              // $imgWrap.css({ top: `${naturalHeight * 0.5 * (ww - 1) - (faceAreaH - clientHeight + 2)}px`, left: "50%" })
              $imgWrap.css({
                top: `calc(50% + ${(imgH - clientHeight) * 0.5 - (faceAreaH - clientHeight + 2)}px)`,
                left: '50%'
              })
            } else if (faceObj.Y * ww < ht) {
              $imgWrap.css({
                top: `calc(50% + ${(imgH - clientHeight) * 0.5 - (faceObj.Y * ww - 1)}px)`,
                left: '50%'
              })
            }
          }
        } else {
          $imgWrap.css({ transform: `translate(-50%, -50%) scale(${cover ? hh : ww}) ` })
          if (faceObj && cover) {
            const faceAreaW = (faceObj.X + faceObj.Width) * hh
            const imgW = hh * naturalWidth
            const wr = (clientWidth + imgW) / 2
            const wl = (imgW - clientWidth) / 2
            if (faceAreaW > wr) {
              $imgWrap.css({
                left: `calc(50% + ${(imgW - clientWidth) * 0.5 - (faceAreaW - clientWidth + 1)}px)`,
                top: '50%'
              })
            } else if (faceObj.X * hh < wl) {
              $imgWrap.css({
                left: `calc(50% + ${(imgW - clientWidth) * 0.5 - (faceObj.X * hh - 1)}px)`,
                top: '50%'
              })
            }
          }
        }
        $imgWrap.show()
        if (faceObj) {
          const $area = window.$(this.$refs.faceArea)
          $area.css({ width: faceObj.Width, height: faceObj.Height, top: faceObj.Y, left: faceObj.X })
          $area.fadeIn(200, () => {
            setTimeout(() => {
              $area.fadeOut(200)
            }, 800)
          })
        }
      } catch (error) {
        console.log(error)
        console.log(this.src)
      }
    },
    onError (e) {
      this.$emit('error', e)
    },
    altClick (e) {
      const img = e.target
      const src = img.src
      if (!src) return
      const evObj = document.createEvent('MouseEvents')
      const nameStartIndex = src.lastIndexOf('/')
      const downloadName = src.slice(nameStartIndex + 1) + '.jpg'
      const $a = document.createElement('a')
      $a.setAttribute('href', src)
      $a.setAttribute('download', downloadName)
      evObj.initMouseEvent('click', true, true, window, 0, 0, 0, 0, 0, false, false, true, false, 0, null)
      $a.dispatchEvent(evObj)
    }
  },
  watch: {
    src () {
      window.$(this.$refs.faceArea).hide()
      window.$(this.$refs.imgWrap).css({ top: '50%', left: '50%' })
    }
  }
}
</script>
<style lang="less">
.feature-img-container {
  position: relative;
  overflow: hidden;
  width: 100%;
  height: 100%;

  .feature-img-wrap {
    display: none;
    position: absolute;
    transition: all;
    user-select: none;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);

    .face-area {
      display: none;
      position: absolute;
      box-shadow: 0 0 0 1px red;
    }

    .img {
      height: auto;
      width: auto;
      vertical-align: middle;

      &.default-src {
        display: none;
      }
    }
  }
}
</style>
