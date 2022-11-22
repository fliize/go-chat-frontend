<template>
  <div>
    <template v-if="!token">
      <a-input v-model:value="username" placeholder="请输入用户名" />
      <a-input v-model:value="password" placeholder="请输入密码" />
      <a-button @click="login"> 进入 </a-button>
    </template>
    <template v-else>
      <div class="chat-wrap">
        <div class="friend-list">
          <div v-for="friend in friendList" class="friend-item" :class="{ active: activeUser === friend.Username }" @click="activeUser = friend.Username">
            {{friend.Username}}
          </div>
        </div>
        <div class="right-panel">
          <div class="message-list">
            <div class="message-item">1</div>
            <div class="message-item">1</div>
           <div class="message-item">1</div>
            <div class="message-item">1</div>
          </div>
          <div class="send-area">
            <div class="send-wrap">
              <a-textarea v-model:value="textInput" />
              <a-button @click="send">发送</a-button>
            </div>
          </div>
        </div>
      </div>
    </template>
  </div>
</template>

<script setup>
import { reactive, ref, watch } from "vue";

const username = ref("");
const password = ref("");
const textInput = ref("");
const messages = ref([]);
const activeUser = ref('')
const token = ref('')
const friendList = ref([])
let websocket
const login = () => {
  return fetch("http://localhost:8081/login", {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json; charset=utf-8'
    },
    body: JSON.stringify({
      username: username.value,
      password: password.value
    })
  }).then(data => data.json()).then(data => {
      console.log('login', data)
      token.value = data.message
        console.log("userName", username.value);
  if (!username.value) return;
  websocket = new WebSocket(`ws://localhost:8080/ws?token=${token.value}&username=${username.value}`);
  websocket.onopen = () => {
    console.log("连接成功");
  };
  websocket.onmessage = (evt) => {
    console.log("onmessage", evt);
    const newMessages = evt.data.split("\n");
    console.log("newMessages", newMessages);
    messages.value = [...messages.value, ...newMessages];
  };
  websocket.onclose = (evt) => {
    console.log('websocket关闭', evt)
  }
    })
};
const send = () => {
    if(!textInput.value) return
    websocket.send(JSON.stringify({
        to: activeUser.value,
        from: username.value,
        message: JSON.stringify({
          msgType: 1,
          content: textInput.value
        }),
    }))
    textInput.value = ''
}

const getFriendList = () => {
  fetch('http://localhost:8081/getFriendList', {
    method: 'GET',
    headers: {
      token: token.value
    },
  }).then(data => data.json()).then(data => {
      console.log('login', data)
      friendList.value = data.message
    })
}

watch(() => token.value, () => {
  getFriendList()
})
</script>

<style lang="scss" scoped>
.chat-wrap {
  display: flex;
  width: 600px;
  height: 400px;
  border: 1px solid red;
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  margin: auto;
  .friend-list{
    width: 100px;
    border: 1px solid red;
    .friend-item{
      height: 24px;
      line-height: 24px;
      cursor: pointer;
      &.active {
        background-color: blue;
        color: #ffffff;
      }
    }
  }
  .right-panel{
    flex: 1;
    border: 1px solid red;
    display: flex;
    flex-direction: column;
    .message-list{
      flex: 1;
      overflow: auto;
    }
    .send-area{
      height: 100px;
      border: 1px solid red;
      .send-wrap{
        display: flex;
        align-items: center;
        gap: 10px;
        padding-top: 20px;
      }
    }
  }
}
</style>
