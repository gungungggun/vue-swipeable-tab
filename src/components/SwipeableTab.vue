<template lang="pug">
  div.swipe-tab(:class="['theme-' + theme]")
    nav.tab
      ul
        li(v-for="(t, i) in tabs" :class="{active: current == i}")
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
    },
    theme: {
      type: Number,
      default: 1
    },
    speed: {
      type: Number,
      default: 100
    },
    touchThreshold: {
      type: Number,
      default: 4
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
        speed: this.speed,
        swipeToSlide: true,
        touchThreshold: this.touchThreshold
      },
      current: 0
    }
  },
  methods: {
    setPosition () {
      try {
        let slide = this.$refs.slick.currentSlide()
        this.arw.style.left = this.width / this.tabs.length * slide + 'px'
        this.current = slide
      } catch (e) {
        console.log(e)
      }
    },
    tab (num) {
      this.$refs.slick.goTo(num)
      this.arw.style.left = this.width / this.tabs.length * num + 'px'
      this.current = num
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
        transition 0.1s
        display block
        width 33.3%
        height 2px
        background #f00
        position absolute
        left 0
        bottom 0
  .view
    width 100%
    height calc(100vh - 46px)
    padding-top 46px
    overflow-y scroll
    &:focus
      outline none
    .scrollable
      width 100%
      min-height 90vh
  &.theme-2
    .tab
      ul
        li
          padding 10px 2px
          margin 0
          a
            color #313140
            &:active, &:hover
              text-decoration none
          &.active
            a
              color #fff
      .arw-area
        z-index -1
        top -4px
        .arw
          border-radius 5px
          background #00a2fd
          height 38px
</style>
