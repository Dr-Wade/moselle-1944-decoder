<template>
  <div id="app">
    <div class="selector">
      <button class="option" :style="{ opacity: component == 'chiffrement' ? '1' : '0.5' }" type="button" @click="component = 'chiffrement'">Chiffrement</button>
      <button class="option" :style="{ opacity: component == 'dechiffrement' ? '1' : '0.5' }" type="button" @click="component = 'dechiffrement'">Déchiffrement</button>
    </div>
    <component :is="component" />
  </div>
</template>

<script>
import Chiffrement from './components/Chiffrement.vue'
import Dechiffrement from './components/Dechiffrement.vue'

export default {
  name: 'app',
  components: {
    Chiffrement,
    Dechiffrement
  },
  data: () => ({
    component: 'chiffrement'
  })
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
  padding: 0 20px;
  text-align: center;
}

.selector {
  width: 100%;
  display: flex;
  gap: 5px;
  justify-content: center;
  margin-bottom: 10px;
}

.option {
  background: #285436;
  color: white;
  font-size: 18px;
  padding: 5px 10px;
  border-radius: 5px;
  border: none;
  outline: none;
}

.process-button {
  background: #285436;
  color: white;
  font-size: 18px;
  padding: 5px 10px;
  border-radius: 5px;
  border: none;
  outline: none;
  margin: 20px ;
}

.result {
  margin-top: 20px;
  width: 100%;
  height: auto;
  min-height: 20vh;
  border: solid 1px black;
}
</style>
