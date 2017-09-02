<template lang="pug">
  div.swipe-tab
    nav.tab
      ul
        li(v-for="(t, i) in tabs")
          a(@click="tab(i)") {{ t }}
      div.arw-area
        div.arw

    div.inner
      slick(ref="slick" :options="options" @setPosition="setPosition")
        div.view(v-for="c in components")
          div.scrollable
            component(:is="c")
</template>

<script>
import Slick from 'vue-slick'
import 'slick-carousel/slick/slick.css'

export default {
  name: 'swipe-tab',
  props: {
    tabs: {
      type: Array,
      default: []
    },
    components: {
      type: Array,
      default: []
    }
  },
  mounted () {
    let ths = this
    const sl = document.getElementsByClassName('slick-slider')[0]
    this.arw = document.getElementsByClassName('arw')[0]
    this.width = window.innerWidth
    this.arw.style.width = this.width / this.tabs.length + 'px'
    sl.addEventListener('touchstart', function (event) {
      sl.addEventListener('touchmove', function (event) {
        let track = document.getElementsByClassName('slick-track')[0]
        let transform = track.style.transform
        ths.x = parseInt(transform.split('px,')[0].replace('translate3d(', '')) * (-1)
        ths.arw.style.left = (ths.x / ths.tabs.length) + 'px'
      }, false)
    })
  },
  data () {
    return {
      x: null,
      arw: null,
      width: null,
      options: {
        arrows: false,
        infinite: false,
        edgeFriction: 0,
        speed: 200,
        swipeToSlide: true,
        touchThreshold: 2
      }
    }
  },
  methods: {
    setPosition () {
      let slide = this.$refs.slick.currentSlide()
      this.arw.style.left = this.width / this.tabs.length * slide + 'px'
    },
    tab (num) {
      this.$refs.slick.goTo(num)
      this.arw.style.left = this.width / this.tabs.length * num + 'px'
    }
  },
  components: {
    Slick
  }
}
</script>

<style lang="stylus">
.swipe-tab
  position relative
  .tab
    width 100%
    position absolute
    background #fff
    z-index 1
    ul
      margin 0
      padding 0
      list-style none
      display flex
      justify-content space-around
      li
        width 100%
        a
          display block
          width 100%
          text-align center
    .arw-area
      width 100%
      height 2px
      position relative
      .arw
        transition 0.2s
        display block
        width 33.3%
        height 2px
        background #f00
        position absolute
        left 0
        bottom 0
  .view
    width 100%
    height 100vh
    overflow-y scroll
    &:focus
      outline none
    .scrollable
      width 100%
      min-height 90vh
</style>
