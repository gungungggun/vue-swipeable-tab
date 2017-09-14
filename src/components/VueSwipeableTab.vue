<template lang="pug">
  div.swipe-tab(:class="['theme-' + theme]")
    nav.tab
      ul
        li.tabs(v-for="(t, i) in tabs" :class="{active: current == i}")
          a(@click="tab(i)") {{ t }}
      div.arw-area
        div.arw(:style="{width: arwWidth + 'px', left: arwLeft + 'px'}")

    div.inner(v-smartscroll="")
      div.scrollable(:style="{width: scrollableWidth + 'vw', transform: 'translateX(-' + scrollableX + 'vw)'}")
        div.background(:style="{width: 100 * components.length + 'vw'}")
          div.view(v-for="(c, i) in components" :id="'view' + i" :class="{lock: isLock.y}" v-viewscroll="")
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
    }
  },
  data () {
    return {
      elScroll: null,
      elTab: null,
      current: 0,
      tabsInfo: [],
      viewsScrollTop: [],
      oldX: 0,
      arwWidth: 0,
      arwLeft: 0,
      scrollableWidth: 0,
      scrollableX: 0,
      isLock: {
        x: false,
        y: false
      }
    }
  },
  directives: {
    smartscroll: {
      bind (el, binding, vnode) {
        let c = vnode.context
        c.elScroll = el
        el.addEventListener('touchstart', (event) => {
          c.oldX = event.touches[0].clientX
        })
        el.addEventListener('touchend', (event) => {
          let time = 100
          setTimeout(() => {
            if (c.getDistance() > 0.5) {
              c.tab(c.current + 1)
            } else if (c.getDistance() < -0.5) {
              c.tab(c.current - 1)
            } else {
              c.tab(c.current)
            }
            c.isLock.y = false
          }, time)
        })
        el.addEventListener('scroll', (event) => {
          c.arwAnimation()
        })
        el.addEventListener('touchmove', (event) => {
          if (!c.isLock.x) {
            let oldX = event.touches[0].clientX
            if (Math.abs(oldX - c.oldX) > 5) {
              el.scrollLeft += (c.oldX - oldX)
              c.oldX = oldX
              c.isLock.y = true
            }
          }
        })
      }
    },
    viewscroll: {
      bind (el, binding, vnode) {
        let c = vnode.context
        c.viewsScrollTop.push(0)
        el.addEventListener('touchmove', (event) => {
          let i = el.id.replace('view', '')
          if (el.scrollTop !== c.viewsScrollTop[i]) {
            c.isLock.x = true
            c.viewsScrollTop[i] = el.scrollTop
          }
        })
        el.addEventListener('touchend', (event) => {
          c.isLock.x = false
        })
      }
    }
  },
  mounted () {
    this.elTab = document.getElementsByClassName('tab')[0]
    this.scrollableWidth = 200
    let tabs = document.getElementsByClassName('tabs')
    let elms = []
    let left = 0
    let right = 0
    elms.forEach.call(tabs, (e) => {
      right += e.clientWidth
      let center = left + (e.clientWidth / 2)
      this.tabsInfo.push({
        width: e.clientWidth,
        left: left,
        right: right,
        center: center
      })
      left += e.clientWidth
    })
    this.arwWidth = this.tabsInfo[0].width
    this.arwLeft = this.tabsInfo[0].left
  },
  methods: {
    scroll (num) {
      let scrollLeft = 0
      if (num > 0) {
        scrollLeft = window.innerWidth
      }
      this.smooth(num, scrollLeft)
    },
    smooth (num, target, d) {
      d = d || (target - this.elScroll.scrollLeft) / 10
      setTimeout(() => {
        if (target !== this.elScroll.scrollLeft) {
          this.elScroll.scrollLeft += d
          this.smooth(num, target, d)
        } else {
          if (num !== this.components.length - 1) {
            this.scrollableWidth = 100 * (num + 2)
          } else {
            this.scrollableWidth = 100 * (num + 1)
          }
          if (num > 1) {
            this.scrollableX = 100 * (num - 1)
          } else {
            this.scrollableX = 0
          }
          this.current = num
        }
      }, 10)
    },
    tab (num) {
      this.arwWidth = this.tabsInfo[num].width
      this.arwLeft = this.tabsInfo[num].left
      this.elTab.scrollLeft = this.tabsInfo[num].center - (window.innerWidth / 2)
      this.scroll(num)
    },
    getDistance () {
      let nowScroll = this.elScroll.scrollLeft
      let w = 0
      if (this.scrollableWidth > 200) {
        w = window.innerWidth
      }
      return (nowScroll - w) / window.innerWidth
    },
    getS (input) {
      let nxt = 1
      let distance = this.getDistance()
      if (distance < 0) nxt = -1
      if (this.current + nxt === this.components.length) return this.tabsInfo[this.current][input]
      return this.tabsInfo[this.current][input] + (this.tabsInfo[this.current + nxt][input] - this.tabsInfo[this.current][input]) * distance * nxt
    },
    arwAnimation () {
      this.arwLeft = this.getS('left')
      this.arwWidth = this.getS('width')
      this.elTab.scrollLeft = this.getS('center') - (window.innerWidth / 2)
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
    background #000
    z-index 1
    width 100vw
    overflow-x scroll
    &.flex
      ul
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
          padding 7px 15px
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
      top 36px
      .arw
        transition 0.1s
        display block
        height 2px
        background #f00
        position absolute
        left 0
        bottom 0
  .inner
    width 100vw
    height calc(100vh - 38px)
    padding-top 38px
    overflow-y hidden
    overflow-x hidden
    .scrollable
      width 100vw
      overflow hidden
      &:after
        content: "";
        display: block;
        height: 0;
        clear: both;
        visibility:hidden;
      .view
        width 100vw
        float left
        height calc(100vh - 38px)
        overflow-y scroll
        &.lock
          overflow-y hidden
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
