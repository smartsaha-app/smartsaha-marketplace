<script setup>
import { ref, onMounted } from 'vue'
const messages = ref([])

onMounted(() => {
  const Pusher = require('pusher-js')
  const pusher = new Pusher('2661ae59074a15a5cf76', { cluster: 'mt1' })

  const channel = pusher.subscribe('notifications-channel')
  channel.bind('new-notification', (data) => {
    messages.value.push(data.message)
  })
})
</script>

<template>
  <div class="notifications-dropdown">
    <h2>Notifications</h2>
    <ul>
      <li v-for="(msg, index) in messages" :key="index">{{ msg }}</li>
    </ul>
  </div>
</template>

<style scoped>
.notifications-dropdown {
  position: relative;
  padding: 1rem;
  background: white;
  border-radius: 0.5rem;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
  max-width: 300px;
}
</style>
