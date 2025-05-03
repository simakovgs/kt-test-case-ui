<template>
  <div id="app">
    <h1>Equipment Manager</h1>
    <h4>KT test case</h4>

    <EquipmentCreateForm />
  </div>
</template>

<script>
import EquipmentCreateForm from './components/EquipmentCreateForm.vue'

export default {
  name: 'App',
  components: {
    EquipmentCreateForm
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>