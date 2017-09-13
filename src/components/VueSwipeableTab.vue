<template lang="pug">
  div.swipe-tab(:class="['theme-' + theme]")
    nav.tab
      ul
        li.tabs(v-for="(t, i) in tabs" :class="{active: current == i}")
          a(@click="tab(i)") {{ t }}
      div.arw-area
        div.arw(:style="{width: arwWidth + 'px', left: arwLeft + 'px'}")

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
    }
  },
  data () {
    return {
      elScroll: null,
      elTab: null,
      current: 0,
      tabsInfo: [],
      nowX: 0,
      isTouch: false,
      arwWidth: 0,
      arwLeft: 0
    }
  },
  directives: {
    smartscroll: {
      bind (el, binding, vnode) {
        vnode.context.elScroll = el
        el.addEventListener('touchstart', (event) => {
          vnode.context.isTouch = true
        })
        el.addEventListener('touchend', (event) => {
          console.log('touchend')
          vnode.context.isTouch = false
          // let width = window.innerWidth
          // let num = Math.round(el.scrollLeft / width)
          // let start = el.scrollLeft
          // let end = num * width
          // vnode.context.scroll(el, 0, start, end)
          // vnode.context.current = num
          // vnode.context.arwAnimation(num)
          // el.scrollLeft = window.innerWidth * num
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
          if (vnode.context.isTouch) {
            // let num = vnode.context.tabs.length
            // let originMove = el.scrollLeft
            vnode.context.arwAnimation()
          } else {
            console.log('force' + vnode.context.current)
          }
        })
      }
    }
  },
  mounted () {
    this.elTab = document.getElementsByClassName('tab')[0]
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
    console.log(this.tabsInfo)
    // this.arwAnimation(0)
  },
  methods: {
    scroll (num) {
      this.elScroll.scrollLeft = window.innerWidth * num
      this.current = num
    },
    tab (num) {
      this.scroll(num)
      this.arwWidth = this.tabsInfo[num].width
      this.arwLeft = this.tabsInfo[num].left

      this.elTab.scrollLeft = this.tabsInfo[num].center - (window.innerWidth / 2)
    },
    getDistance () {
      let nowScroll = this.elScroll.scrollLeft
      return (nowScroll - (window.innerWidth * this.current)) / window.innerWidth
    },
    getS (input) {
      let nxt = 1
      let distance = this.getDistance()
      if (distance < 0) nxt = -1
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
        height calc(100vh - 38px)
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
