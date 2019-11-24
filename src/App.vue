<template>
  <div>
    <div class="nav">
      <div class="nav__item">
        <router-link to="/" tag="span" class="btn">Описание проекта</router-link>
      </div>
      <div class="nav__item">
        <router-link to="/lagrandj" tag="span" class="btn">Интерполяционный многочлен Лагранджа</router-link>
      </div>
      <div class="nav__item">
        <router-link to="/newton" tag="span" class="btn">Метод "Ньютона"</router-link>
      </div>
    </div>
    <div id="app">
      <router-view/>
    </div>
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
