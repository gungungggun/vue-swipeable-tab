<template lang="pug">
  div.swipe-tab(:class="['theme-' + theme]")
    nav.tab
      ul(:style="{width: tabWidth + 'px'}")
        li.tabs(v-for="(t, i) in tabs" :class="{active: current == i}")
          a(@click="tab(i)") {{ t }}
      div.arw-area
        div.arw(:style="{width: arwWidth + 'px', transform: 'translate3d(' + arwLeft + 'px,0,0)'}")

    div.inner(v-smartscroll="")
      div.scrollable(:style="{width: components.length * 100 + 'vw'}")
        div.view(v-for="(c, i) in components")
          nav.header(v-show="c.header != null" :class="isHeaderShows[i] ? '' : 'swipe-tab-header-close'")
            //- 何故かこれを入れないとviewsScrollTop[i]がバインドされない
            p(v-show="false") {{ prebind }}
            component(:is="c.header" v-headerscroll="")
          div.main(:id="'view' + i" :class="{lock: isLock.y, withtop: c.header != null}" v-mainscroll="")
            component(:is="c.main")
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
    frame: {
      type: Number,
      default: 5
    },
    showHeaderHeight: {
      type: Number,
      default: 100
    },
    isHeaderLock: {
      type: Boolean,
      default: true
    },
    isSequenceTab: {
      type: Boolean,
      default: false
    },
    swipeBoost: {
      type: Number,
      default: 1
    }
  },
  data () {
    return {
      current: 0,
      prebind: '',
      elScroll: null,
      elTab: null,
      elMains: [],
      tabsInfo: [],
      viewsScrollTop: [],
      isHeaderShows: [],
      oldX: 0,
      oldY: 0,
      swipeX: 0,
      swipeY: 0,
      isSwipeLock: false,
      arwWidth: 0,
      arwLeft: 0,
      isLock: {
        x: false,
        y: false
      },
      target: null,
      d: null,
      tabWidth: 5000,
      margin: [0, 4],
      isOverScroll: false,
      overScrollOldY: null
    }
  },
  directives: {
    smartscroll: {
      bind (el, binding, vnode) {
        let c = vnode.context
        c.elScroll = el
        el.addEventListener('touchstart', (event) => {
          c.oldX = event.touches[0].clientX
          c.oldY = event.touches[0].clientY
        })
        el.addEventListener('touchend', (event) => {
          let plus = 0
          if (c.getDistance() > 0.5) {
            plus = 1
          } else if (c.getDistance() < -0.5) {
            plus = -1
          } else {
            if (!c.isSwipeLock) {
              if (c.swipeX > 30 && c.swipeX > c.swipeY) {
                plus = 1
              } else if (c.swipeX < -30 && c.swipeX > c.swipeY) {
                plus = -1
              }
            }
          }
          if (c.current + plus < 0 || c.current + plus >= c.components.length) plus = 0
          c.tab(c.current + plus)
          c.isLock.y = false
          c.isSwipeLock = false
        })
        el.addEventListener('scroll', (event) => {
          c.arwAnimation()
        })
        el.addEventListener('touchmove', (event) => {
          if (!c.isLock.x) {
            let oldX = event.touches[0].clientX
            let oldY = event.touches[0].clientY
            c.swipeX = c.oldX - oldX
            c.swipeY = c.oldY - oldY
            if (Math.abs(c.swipeX) > 5 && Math.abs(c.swipeX) > Math.abs(c.swipeY)) {
              el.scrollLeft += c.swipeX * c.swipeBoost
              if (c.isSequenceTab) {
                if (c.getDistance() > 0.5) {
                  c.current++
                  c.isSwipeLock = true
                } else if (c.getDistance() < -0.5) {
                  c.current--
                  c.isSwipeLock = true
                }
              }
              c.oldX = oldX
              c.isLock.y = true
            }
          }
        })
      }
    },
    mainscroll: {
      bind (el, binding, vnode) {
        let c = vnode.context
        c.viewsScrollTop.push(0)
        c.elMains.push(el)
        el.addEventListener('touchmove', (event) => {
          let i = el.id.replace('view', '')
          c.prebind = c.viewsScrollTop[i]
          if (el.scrollTop !== c.viewsScrollTop[i]) {
            c.isLock.x = true
            c.viewsScrollTop[i] = el.scrollTop
            if (c.viewsScrollTop[i] > c.showHeaderHeight) {
              c.isHeaderShows[i] = false
            } else {
              c.isHeaderShows[i] = true
            }
          }
          if (el.scrollTop === 0 && !c.isLock.y) {
            if (c.isOverScroll) {
              if (c.overScrollOldY !== null) {
                let diff = c.overScrollOldY - event.touches[0].clientY
                c.$emit('overScroll', diff)
              }
              c.overScrollOldY = event.touches[0].clientY
            }
            c.isOverScroll = true
          } else {
            c.isOverScroll = false
          }
        })
        el.addEventListener('touchend', (event) => {
          c.isLock.x = false
          c.isOverScroll = false
          c.overScrollOldY = null
          c.$emit('overScrollCancel')
        })
      }
    },
    headerscroll: {
      bind (el, binding, vnode) {
        let c = vnode.context
        if (c.isHeaderLock) {
          el.addEventListener('touchmove', (event) => {
            c.isLock.x = true
            c.isLock.y = true
          })
          el.addEventListener('touchend', (event) => {
            c.isLock.x = false
            c.isLock.y = false
          })
        }
      }
    }
  },
  mounted () {
    this.elTab = document.getElementsByClassName('tab')[0]
    let tabs = document.getElementsByClassName('tabs')
    let elms = []
    let width = 0
    let left = 0
    let right = 0
    let tabWidth = this.margin[this.theme - 1] * 3
    elms.forEach.call(tabs, (e) => {
      this.isHeaderShows.push(true)
      width = e.clientWidth
      right += width
      let center = left + (width / 2)
      this.tabsInfo.push({
        width: width,
        left: left,
        right: right,
        center: center
      })
      left += width
      tabWidth += width
    })
    this.tabWidth = tabWidth
    this.arwWidth = this.tabsInfo[0].width
    this.arwLeft = this.tabsInfo[0].left

    this.tab(this.current)
  },
  methods: {
    scroll (num) {
      this.target = num * window.innerWidth
      this.d = (this.target - this.elScroll.scrollLeft) / this.frame
      this.scrollAnimation()
      this.current = num
    },
    scrollAnimation () {
      let pm = this.d > 0 ? 1 : -1
      if (this.target * pm > this.elScroll.scrollLeft * pm) {
        if ((this.elScroll.scrollLeft + this.d) * pm > this.target * pm) {
          this.elScroll.scrollLeft = this.target
        } else {
          this.elScroll.scrollLeft += this.d
        }
        requestAnimationFrame(this.scrollAnimation)
      }
    },
    tab (num) {
      this.arwWidth = this.tabsInfo[num].width
      this.arwLeft = this.tabsInfo[num].left
      this.elTab.scrollLeft = this.tabsInfo[num].center - (window.innerWidth / 2)
      this.scroll(num)
    },
    getDistance () {
      return (this.elScroll.scrollLeft - (window.innerWidth * this.current)) / window.innerWidth
    },
    getS (input) {
      let nxt = 1
      let distance = this.getDistance()
      if (distance < 0) nxt = -1
      if (this.current + nxt === this.components.length) {
        return this.tabsInfo[this.current][input]
      }
      return this.tabsInfo[this.current][input] + (this.tabsInfo[this.current + nxt][input] - this.tabsInfo[this.current][input]) * distance * nxt
    },
    arwAnimation () {
      this.arwLeft = this.getS('left')
      this.arwWidth = this.getS('width')
      this.elTab.scrollLeft = this.getS('center') - (window.innerWidth / 2)
    }
  },
  ready () {
    requestAnimationFrame(this.scrollAnimation)
  }
}
</script>

<style lang="stylus">
.swipe-tab
  position relative
  .tab
    width 100%
    position absolute
    z-index 1
    width 100vw
    overflow-x scroll
    ul
      margin 0
      padding 0
      list-style none
      width 1000%
      &:after
        content ""
        display block
        overflow hidden
        clear both
      li
        float left
        a
          transition .2s
          padding 7px 15px
          box-sizing border-box
          display block
          width 100%
          text-align center
        &.active
          a
            text-decoration none
    .arw-area
      width 100%
      height 2px
      position relative
      .arw
        transition 0.1s
        display block
        height 2px
        background #f00
        position absolute
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
        .main
          height 100%
          overflow-y scroll
          -webkit-overflow-scrolling touch
          &.withtop
            padding-top 65px
          &.lock
            overflow-y hidden
      .header
        transition .3s
        position absolute
        top 0
        z-index 1
        transform translate3d(0, 0, 0)
        &.swipe-tab-header-close
          transform translate3d(0, -80px, 0)
          opacity 0
  &.theme-1
    .tab
      background #000
      ul
        li
          a
            background #000
            color #999
          &.active
            a
              color #fff
  &.theme-2
    .tab
      padding 4px
      ul
        li
          display flex
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
        top -2px
        .arw
          border-radius 5px
          background #00a2fd
          height 38px
    .inner
      height calc(100vh - 46px)
      padding-top 46px
</style>
