<template>
  <div>
    <template v-if="!hasLogged">
      <a-input v-model:value="username" placeholder="请输入用户名" />
      <a-button @click="login"> 进入 </a-button>
    </template>
    <template v-else>
      <div class="chat-wrap">
        <div v-for="message in messages" class="message-item">
          {{ message }}
        </div>
        <a-space>
            <a-input v-model:value="toUsername" placeholder="发送给" />
            <a-input v-model:value="textInput" placeholder="请输入消息" />
            <a-button @click="send">发送</a-button>
        </a-space>
      </div>
    </template>
  </div>
</template>

<script setup>
import { reactive, ref } from "vue";

const hasLogged = ref(false);
const username = ref("");
const toUsername = ref("");
const textInput = ref("");
const messages = ref([]);
let websocket
const login = () => {
  console.log("userName", username.value);
  if (!username.value) return;
  websocket = new WebSocket(`ws://localhost:8080/ws?username=${username.value}`);
  websocket.onopen = () => {
    console.log("连接成功");
    hasLogged.value = true;
  };
  websocket.onmessage = (evt) => {
    console.log("onmessage", evt);
    const newMessages = evt.data.split("\n");
    console.log("newMessages", newMessages);
    messages.value = [...messages.value, ...newMessages];
  };
};
const send = () => {
    if(!textInput.value) return
    websocket.send(JSON.stringify({
        to: toUsername.value,
        message: textInput.value
    }))
    textInput.value = ''
}
</script>

<style lang="scss" scoped>
.chat-wrap {
}
</style>
