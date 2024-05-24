<template>
  <ChatComponent/>
  <ChatWidgetComponent/>
</template>

<script>
import ChatComponent from './components/ChatComponent.vue'
import ChatWidgetComponent from './components/ChatWidgetComponent.vue'

export default {
  name: 'App',
  components: {
    ChatComponent,
    ChatWidgetComponent
  }
}
</script>
