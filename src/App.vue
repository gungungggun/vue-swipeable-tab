<template lang="pug">
  div#app
    div
      swipeable-tab(:tabs="tabs" :components="components" :theme="2" :is-sequence-tab="true")
    div
      h2 test
      p swipeable out
</template>

<script>
import SwipeableTab from './components/VueSwipeableTab'
import Tab1 from './components/Tab1'
import Tab2 from './components/Tab2'
import Tab3 from './components/Tab3'
import Head1 from './components/Head1'

export default {
  name: 'app',
  data () {
    return {
      tabs: [
        'サッカー',
        '野球',
        'バスケットボール',
        'テニス',
        'バドミントン',
        'ゴルフ'
      ],
      components: [
        {
          header: Head1,
          main: Tab1
        },
        {
          main: Tab2
        },
        {
          header: Head1,
          main: Tab3
        },
        {
          main: Tab1
        },
        {
          main: Tab2
        },
        {
          main: Tab3
        }
      ]
    }
  },
  components: {
    SwipeableTab
  }
}
</script>

<style lang="stylus">
body, html
  padding 0
  margin 0
  width 100%
#app
  width 100%
  height 100vh
  overflow-x hidden
  overflow-y scroll
  background #f0f0f0
*::-webkit-scrollbar
  display none
</style>
