<template lang="pug">
  div#app
    div
      swipe-tab(:tabs="tabs" :components="components")
</template>

<script>
import SwipeTab from './components/SwipeTab'
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
    SwipeTab
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
