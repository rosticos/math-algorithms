<template>
  <div id="app">
    <div class="nav">
      <div class="nav__item">
        <router-link to="/" tag="span" class="btn">Интерполяционный многочлен Лагранджа</router-link>
      </div>
      <div class="nav__item">
        <router-link to="/about" tag="span" class="btn">Метод "Ньютона"</router-link>
      </div>
    </div>
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style lang="sass">
  @import './assets/style/main.sass'
</style>
