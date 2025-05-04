<template>
  <div id="app">
    <h1>Equipment Manager</h1>
    <h4>KT test case</h4>

    <div class="components-container">
      <EquipmentCreateForm id="create-form" class="left-component"/>
      <EquipmentInfo id="form-info" class="right-component"/>
    </div>
  </div>
</template>

<script>
import EquipmentCreateForm from './components/EquipmentCreateForm.vue'
import EquipmentInfo from './components/EquipmentInfo.vue'

export default {
  name: 'App',
  components: {
    EquipmentCreateForm,
    EquipmentInfo
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

.components-container {
  display: flex;
  justify-content: space-between;
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
}

.left-component {
  flex: 1;
  margin-right: 10px;
  text-align: center;
}

.right-component {
  flex: 1;
  margin-left: 10px;
  text-align: center;
}

#create-form, #form-info {
  margin: 5px;
}
</style>