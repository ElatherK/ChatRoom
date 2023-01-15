<template>
  <div id="container">
    <button @click="handleConnect">连接服务</button>
    <button @click="handleClose">断开连接</button>
    <input type="text" placeholder="用户名" v-model="userName" />
    <div class="left">
      <div class="roomName">
        <span>聊天室名称</span>
      </div>
      <div class="content" ref="content">
      </div>
      <div class="textBox">
        <div class="toolBox"></div>
        <div class="textArea">
          <a-textarea
            v-model="textValue"
            @pressEnter="handlePressEnter"
          ></a-textarea>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import { Input } from "ant-design-vue";
import Vue from "vue";
Vue.use(Input);
export default {
  data() {
    return {
      textValue: "",
      ws: null,
      userName: "",
    };
  },
  methods: {
    handlePressEnter() {
      const message = this.textValue
      if (this.ws) {
        this.ws.send(message + "/**/" + this.userName);
        this.textValue = ''
      }else{
        this.$message.error("未连接到服务器,请先连接!")
      }
    },
    handleConnect() {
      if(!this.userName){
        this.$message.error('用户名不能为空!')
        return
      }
      this.ws = new WebSocket(`ws://localhost:5000/${this.userName}`);
      this.ws.onopen = () => {
        this.$message.info("连接成功");
      };
      this.ws.onmessage = (event) => {
        let name = event.data.split("/**/")[1];
        let message = event.data.split("/**/")[0];
        if (name === this.userName) {
          this.$refs.content.innerHTML += `
          <div class="me">
            <div class="meMessageBox">
              <span class="meName">${name}</span>
              <span class="message">${message}</span>
            </div>
          </div>
          `;
        }else{
          this.$refs.content.innerHTML += `
          <div class="other">
            <div class="otherMessageBox">
              <span class="otherName">${name}</span>
              <span class="message">${message}</span>
            </div>
          </div>
          `;
        }
        this.$refs.content.scrollTop = this.$refs.content.scrollHeight;
      };
      this.ws.onclose = () => {
        this.$refs.content.innerHTML += "<div>----------你已断开连接----------</div>";
      };
    },
    handleClose() {
      this.$message.info("连接断开");
      if(this.ws){
        this.ws.send("close Connction" + "/" + this.userName);
        this.ws.close();
      }
      this.ws = null
    },
  },
};
</script>
<style lang="less">
#container {
  margin: 0 auto;
  padding: 0;
  width: 660px;
  background-color: rgba(15, 34, 67, 0.05);
  .left {
    .roomName {
      display: flex;
      align-items: center;
      height: 55px;
      border-bottom: 1px solid #706767;
      padding-left: 25px;
      span {
        color: black;
      }
    }
    .content {
      height: 425px;
      border-bottom: 1px solid #706767;
      overflow: scroll;
      padding: 10px;
      position: relative;
      .other {
        overflow: visible;
        display: flex;
        justify-content: flex-start;
        .otherMessageBox {
          display: flex;
          flex-direction: column;
          align-items: flex-start;
          .message {
            background: skyblue;
            padding: 10px;
            border-radius: 10px;
          }
        }
      }
      .me {
        overflow: visible;
        display: flex;
        justify-content: flex-end;
        .meMessageBox {
          display: flex;
          flex-direction: column;
          align-items: flex-end;
          .message {
            background: skyblue;
            padding: 10px;
            border-radius: 10px;
          }
        }
      }
    }
    .textBox {
      height: 125px;
      .textArea {
        .ant-input {
          font-size: 25px;
          padding: 0;
          height: 125px;
          width: 100%;
          border: 0;
          background-color: rgba(15, 34, 67, 0.05);
        }
      }
    }
  }
}
</style>