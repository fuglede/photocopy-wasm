<template>
  <Photocopy/>
</template>

<script>
import Photocopy from './components/Photocopy.vue'

export default {
  name: 'App',
  components: {
    Photocopy
  }
}
</script>

<style>
#app {
  margin: 30px;
}

footer {
  position: fixed;
  font-size: 100%;
  left: 10px;
  bottom: 5px;
  right: 10px;
  width: 95%;
  text-align: center;
}
</style>
