<template>
  <div class="input-form_middle">
    <h2>Наша команда</h2>
    <div class="container">
      <div style="display: grid; grid-template-columns: 300px 300px 300px;">
        <div v-for="(unit, index) in team" :key="unit.name" style="text-align: center">
          <img :src="`img/${index + 1}.png`" class="avatar" alt="photo">
          <div class="description">
            <p class="name">{{ unit.name }}</p>
            <p class="role">{{ unit.role }}</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Team from './team.json'

export default {
  data () {
    return {
      team: []
    }
  },
  created () {
    this.team = JSON.parse(JSON.stringify(Team))
  }
}
</script>

<style>

</style>
