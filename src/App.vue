<template>
  <div id="app">
    <chat-room></chat-room>
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import ChatRoom from './components/ChatRoom.vue'
export default {
  name: 'App',
  components: {
    ChatRoom
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>

