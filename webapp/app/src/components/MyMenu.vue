<template lang="pug">
div
  .floating-menu
    .ui.primary.button(v-on:click="tutorial") Tutorial
    .ui.basic.button(href="#" v-scroll-to="'#attribution'") About
    .color-scale-container
      .scale-label High (1.0)
      .scale-bar
      .scale-label Low (0.0)
  .ui.container#welcome-container(v-if="showWelcome" data-intro="Welcome to our web application. Here you can explore how body size impacts urban tolerance across the tree of life.")
    h1#masthead 
</template>

<script>
export default {
  name: 'my-menu',
  data () {
    return {
      showWelcome: false
    }
  },
  methods: {
    tutorial () {
      this.showWelcome = true
      this.$nextTick(() => {
        window.introJS().start().onexit(() => {
          this.showWelcome = false
        })
      })
    }
  },
  mounted () {
    document.documentElement.setAttribute('data-theme', 'light')
  }
}
</script>

<style scoped>
.floating-menu {
  position: fixed;
  top: 1em;
  right: 1em;
  z-index: 1001;
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 0.5em;
}
.color-scale-container {
  margin-top: 1em;
  display: flex;
  flex-direction: column;
  align-items: center;
  background: rgba(255, 255, 255, 0.9);
  padding: 10px;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  border: 1px solid #ddd;
}
.scale-bar {
  width: 20px;
  height: 150px;
  background: linear-gradient(to bottom, #d73027, #f46d43, #fdae61, #fee090, #ffffbf, #e0f3f8, #abd9e9, #74add1, #4575b4);
  margin: 5px 0;
  border-radius: 2px;
}
.scale-label {
  font-size: 10px;
  font-weight: bold;
  color: #333;
  text-align: center;
}
#masthead {
  text-align: center;
  font-weight: 200;
  letter-spacing: 3px;
  text-transform: lowercase;
}
.container {
  padding: 1em 0;
}
</style>
