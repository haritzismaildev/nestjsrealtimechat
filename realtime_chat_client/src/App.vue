<script setup>
import HelloWorld from './components/HelloWorld.vue'
//import TheWelcome from './components/TheWelcome.vue'

import { io } from 'socket.io-client';
import { onBeforeMount, ref } from 'vue';

  const socket = io('http://localhost:3001');

  const messages = ref([]);
  const messageText = ref('');
  const joined = ref(false);
  const username = ref('');
  const typingDisplay = ref('');

  onBeforeMount(() => {
    socket.emit('findAllMessages', {}, (response) => {
      messages.value = response;
    });

  socket.on('message', (message) => {
    messages.value.push(message);
  });

  socket.on('typing', ({ username, isTyping }) => {
    if(isTyping) {
      typingDisplay.value = `${username} is typing...`;
    } else {
      typingDisplay.value = '';
    }
  });
});

const join = () => {
  socket.emit('join', {username: username.value}, () => {
    joined.value = true;
  })
}

  const sendMessage = () => {
    socket.emit('createMessage', { text: messageText.value}, response => {
      //messages.value.push(response)
      messageText.value = '';
    })
  }

  let timeout;
const emitTyping = () => {
  socket.emit('typing', {isTyping: true });
  timeout = setTimeout(() => {
    socket.emit('typing', { isTyping: false });
  }, 2000);

}
</script>

<template>
  <header>
    <img alt="Vue logo" class="logo" src="./assets/logo.svg" width="125" height="125" />

    <div class="wrapper">
      <HelloWorld msg="Haritz Sample Chat Realtime using Vue - NestJS & Socket IO" />
    </div>
  </header>

  <main>
    <div class="chat">
      <div v-if="!joined">
        <form @submit.prevent="join">
          <label>What's your username : </label>
          <input v-model="username" />
          <button type="submit">Send</button>
        </form>
      </div>
    <div class="chat-container" v-else>
      <div class="messages-container">
        <div v-for="message in messages">
          [{{ message.username }}]: {{ message.text }}
        </div>
      </div>

      <div v-if="typingDisplay">{{ typingDisplay }}</div>

      <hr />

      <div class="message-input">
        <form @submit.prevent="sendMessage">
          <label>Message : </label>
          <input v-model="messageText" @input="emitTyping" />

          <button type="submit">Send</button>
        </form>
      </div>

    </div>
  </div>
  </main>
</template>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  .chat {
    padding: 20px;
    height: 100vh;
  }

  .chat-container {
    display: flex;
    flex-direction: column;
    height: 100%;
  }

  messages-container {
    flex: 1;
  }
}
</style>
