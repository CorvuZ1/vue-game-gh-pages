<template>
  <div id="app">
    <SimonGame/>
  </div>
</template>
<script>
import "@/reset/reset.sass";
import SimonGame from './components/SimonGame.vue';

export default {
  name: 'App',
  components: {
    SimonGame
  }
}

</script>

<style lang="sass" scoped>
  #app
    width: 100vw
    height: 100vh
    min-width: 1000px
    min-height: 800px
    display: flex
    justify-content: center
    align-items: center
</style>
