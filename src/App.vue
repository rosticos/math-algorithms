<template>
  <div>
    <app-header />
    <div id="app">
      <router-view/>
    </div>
</div>
</template>

<script>
import AppHeader from '@/components/Header'

export default {
  name: 'App',
  components: { AppHeader }
}
</script>

<style lang="sass">
  @import './assets/style/main.sass'
</style>
