<template lang="pug">
  div.swipe-tab(:class="['theme-' + theme]")
    nav.tab
      ul
        li(v-for="(t, i) in tabs" :class="{active: current == i}")
          a(@click="tab(i)") {{ t }}
      div.arw-area
        div.arw

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
    duration: {
      type: Number,
      default: 100
    },
    interval: {
      type: Number,
      default: 10
    }
  },
  directives: {
    smartscroll: {
      bind (el, binding, vnode) {
        vnode.context.elScroll = el
        el.addEventListener('touchstart', (event) => {
          el.addEventListener('touchmove', (event) => {
            let num = vnode.context.tabs.length
            let originMove = el.scrollLeft
            vnode.context.arw.style.left = (originMove / num) + 'px'
          })
        })
        el.addEventListener('touchend', (event) => {
          let width = window.innerWidth
          let num = Math.round(el.scrollLeft / width)
          let start = el.scrollLeft
          let end = num * width
          vnode.context.scroll(el, 0, start, end)
          vnode.context.current = num
          vnode.context.arw.style.left = (width / vnode.context.tabs.length) * num + 'px'
        })
      }
    }
  },
  mounted () {
    this.arw = document.getElementsByClassName('arw')[0]
    this.arw.style.width = window.innerWidth / this.tabs.length + 'px'
  },
  data () {
    return {
      elScroll: null,
      arw: null,
      current: 0
    }
  },
  methods: {
    scroll (el, currentTime, start, end) {
      currentTime += this.interval
      let s = start - end
      s = s / (this.duration / this.interval)
      el.scrollLeft = start - (s * (currentTime / this.interval))
      if (currentTime < this.duration) {
        let ths = this
        setTimeout(() => { ths.scroll(el, currentTime, start, end) }, this.interval)
      }
    },
    tab (num) {
      this.arw.style.left = window.innerWidth / this.tabs.length * num + 'px'
      this.scroll(this.elScroll, 0, this.elScroll.scrollLeft, window.innerWidth * num)
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
