<template>
  <div>
    <el-menu
      router
      class="el-menu-demo"
      mode="horizontal"
      :default-active="$route.path"
      background-color="#545c64"
      text-color="#fff"
      active-text-color="#ffd04b"
    >
      <el-menu-item index="/">Описание</el-menu-item>
      <el-menu-item index="/lagrandj">Метод Лагранджа</el-menu-item>
      <el-menu-item index="/newton">Метод Ньютона</el-menu-item>
    </el-menu>
  </div>
</template>
<script>
export default {
  mounted () {
    this.activeLink = this.$route.path
  },
  watch: {
    $route (newVal, oldVal) {
      this.activeLink = newVal.path
    }
  }
}
</script>
