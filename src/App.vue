<template lang="pug">
  div#app
    div
      swipeable-tab(:tabs="tabs" :components="components" :theme="1" :flex="false")
    div
      h2 test
      p swipeable out
</template>

<script>
import SwipeableTab from './components/VueSwipeableTab'
import Tab1 from './components/Tab1'
import Tab2 from './components/Tab2'
import Tab3 from './components/Tab3'

export default {
  name: 'app',
  data () {
    return {
      tabs: [
        'タブ１',
        'タブ２',
        'タブ３',
        'タブ４',
        'タブ５',
        'タブ６'
      ],
      components: [
        Tab1,
        Tab2,
        Tab3,
        Tab1,
        Tab2,
        Tab3
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
</style>
