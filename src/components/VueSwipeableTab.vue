<template lang="pug">
  div.swipe-tab(:class="['theme-' + theme]")
    nav.tab(:class="{flex: flex}")
      ul
        li.tabs(v-for="(t, i) in tabs" :class="{active: current == i}")
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
    flex: {
      type: Boolean,
      dafault: true
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
        /*
        el.addEventListener('touchstart', (event) => {
          el.addEventListener('touchmove', (event) => {
            let num = vnode.context.tabs.length
            let originMove = el.scrollLeft
            vnode.context.arwAnimation(num, originMove)
          })
        })
        */
        el.addEventListener('touchstart', (event) => {
          vnode.context.isTouch = true
        })
        el.addEventListener('touchend', (event) => {
          console.log('touchend')
          vnode.context.isTouch = false
          let width = window.innerWidth
          let num = Math.round(el.scrollLeft / width)
          let start = el.scrollLeft
          let end = num * width
          vnode.context.scroll(el, 0, start, end)
          vnode.context.current = num
          vnode.context.arwAnimation(num)
        })
        el.addEventListener('scroll', (event) => {
          let oldX = vnode.context.nowX
          let nowX = el.scrollLeft
          if (oldX !== nowX) {
            vnode.context.nowX = nowX
            el.dispatchEvent(new Event('scroll-x'))
          }
        })
        el.addEventListener('scroll-x', (event) => {
          console.log('scroll-x')
          if (vnode.context.isTouch) {
            let num = vnode.context.tabs.length
            let originMove = el.scrollLeft
            vnode.context.arwAnimation(num, originMove)
          }
        })
      }
    }
  },
  mounted () {
    this.elArw = document.getElementsByClassName('arw')[0]
    this.elTab = document.getElementsByClassName('tab')[0]
    if (this.flex) {
      this.elArw.style.width = window.innerWidth / this.tabs.length + 'px'
    } else {
      let tabs = document.getElementsByClassName('tabs')
      let elms = []
      let ths = this
      let point = 0
      elms.forEach.call(tabs, (e) => {
        point += e.clientWidth
        if (point < window.innerWidth / 2) {
          this.middlePoint++
        }
        ths.tabsInfo.push({
          width: e.clientWidth,
          breakPoint: point - e.clientWidth / 2
        })
      })
      this.arwAnimationNotFlex(0)
    }
  },
  data () {
    return {
      elScroll: null,
      elArw: null,
      elTab: null,
      current: 0,
      tabsInfo: [],
      middlePoint: 0,
      nowX: 0,
      isTouch: false
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
      this.arwAnimation(num)
      this.scroll(this.elScroll, 0, this.elScroll.scrollLeft, window.innerWidth * num)
      this.current = num
    },
    arwAnimation (num, originMove) {
      if (this.flex) {
        this.arwAnimationFlex(num, originMove)
      } else {
        this.arwAnimationNotFlex(num, originMove)
      }
    },
    arwAnimationFlex (num, originMove) {
      if (typeof originMove !== 'undefined') {
        this.elArw.style.left = (originMove / num) + 'px'
      } else {
        this.elArw.style.left = window.innerWidth / this.tabs.length * num + 'px'
      }
    },
    arwAnimationNotFlex (num, originMove) {
      if (typeof originMove !== 'undefined') {
        let x = ((originMove - window.innerWidth * this.current) / window.innerWidth) * this.tabsInfo[this.current].width
        if (this.current < this.middlePoint) {
          this.elArw.style.left = x + 'px'
        } else {
          console.log(originMove / num)
          this.elTab.scrollLeft = 100
        }
      } else {
        this.elArw.style.width = this.tabsInfo[num].width + 'px'
        let left = 0
        for (let i = 0; i < num; i++) {
          left += this.tabsInfo[i].width
        }
        if (num >= this.middlePoint) {
          this.elTab.scrollLeft = this.tabsInfo[num].breakPoint - (window.innerWidth / 2)
        } else {
          this.elTab.scrollLeft = 0
        }
        this.elArw.style.left = left + 'px'
      }
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
    width 100vw
    overflow-x scroll
    &.flex
      ul
        width 100%
        float none
        padding 0
        display flex
        justify-content space-around
        li
          width 100%
      .arw-area
        top auto
    ul
      margin 0
      padding 0
      list-style none
      width 1000%
      li
        float left
        a
          padding 5px 15px
          box-sizing border-box
          background #000
          color #999
          display block
          width 100%
          text-align center
        &.active
          a
            color #fff
            text-decoration none

    .arw-area
      width 100%
      height 2px
      position relative
      top 32px
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
