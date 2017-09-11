<template lang="pug">
  div.swipe-tab(:class="['theme-' + theme]")
    nav.tab
      ul
        li(v-for="(t, i) in tabs" :class="{active: current == i}")
          a(@click="tab(i)") {{ t }}
      div.arw-area
        div.arw(@move-arw="moveArw()")

    div.inner(v-smartscroll="")
      div.scrollable(:style="{width: 100 * components.length + 'vw'}")
        div.view(v-for="c in components")
          component(:is="c")
</template>

<script>
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
    }
  },
  directives: {
    smartscroll: {
      bind (el, binding, vnode) {
        el.addEventListener('touchstart', (event) => {
          el.addEventListener('touchmove', (event) => {
          })
        })
        el.addEventListener('touchend', (event) => {
          let width = window.innerWidth
          let num = Math.round(el.scrollLeft / width)
          let start = el.scrollLeft
          let end = num * width
          let currentTime = 0
          let duration = 100
          let interval = 10
          let scroll = () => {
            currentTime += interval
            let s = start - end
            s = s / (duration / interval)
            el.scrollLeft = start - (s * (currentTime / interval))
            if (currentTime < duration) {
              setTimeout(scroll, interval)
            }
          }
          scroll()
        })
      }
    }
  },
  mounted () {
    this.arw = document.getElementsByClassName('arw')[0]
    this.width = window.innerWidth
    this.arw.style.width = this.width / this.tabs.length + 'px'
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
    moveArw () {
      console.log('OK')
    },
    tab (num) {
      this.$refs.slick.goTo(num)
      this.arw.style.left = this.width / this.tabs.length * num + 'px'
      this.current = num
    }
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
  .inner
    width 100vw
    height calc(100vh - 46px)
    padding-top 46px
    overflow-y hidden
    overflow-x scroll
    .scrollable
      width 100vw
      &:after
        content: "";
        display: block;
        height: 0;
        clear: both;
        visibility:hidden;
      .view
        width 100vw
        float left
        height calc(100vh - 46px)
        overflow-y scroll
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
