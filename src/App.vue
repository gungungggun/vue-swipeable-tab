<template lang="pug">
  div#app
    div
      swipeable-tab(:tabs="tabs" :components="components")
    div
      h2 test
      p swipeable out
</template>

<script>
import SwipeableTab from './components/VueSwipeableTab'
import Tab1 from './components/Tab1'

export default {
  name: 'app',
  data () {
    return {
      tabs: [
        'タブ１',
        'タブ２',
        'タブ３'
      ],
      components: [
        Tab1,
        Tab1,
        Tab1
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
  background #f0f0f0
</style>
