<template lang="pug">
#app
  my-menu
  phylo-tree
</template>

<script>
import PhyloTree from './components/PhyloTree'
import MyMenu from './components/MyMenu'
export default {
  name: 'app',
  components: {
    PhyloTree,
    MyMenu
  },
  mounted () {
    if (!localStorage.getItem('tutorial_completed')) {
      window.introJS().start()
      localStorage.setItem('tutorial_completed', 'true')
    }
  }
}
</script>

<style>
@import './assets/introjs.css'
</style>
