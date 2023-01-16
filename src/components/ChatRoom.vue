<template>
  <div id="container">
    <div class="connect-box">
      <button @click="handleConnect">连接服务</button>
      <button @click="handleClose">断开连接</button>
      <input type="text" placeholder="用户名" v-model="userName" />
    </div>
    <div class="room">
      <div class="right"></div>
      <div class="left">
        <div class="roomName">
          <span>聊天室名称</span>
          <span class="tool" @click="openGroup">...</span>
        </div>
        <div :class="['group',isOpen?'showBox':'hideBox']">
          <ul>
            <li v-for="el,index in userList" :key="index">
              <div class="user" v-if="el.status===0">
                <a-avatar>{{el.name}}</a-avatar>
                <div>{{el.name}}</div>
              </div>
              <div class="user" v-else>
                <a-avatar style="backgroundColor:#3399ff">{{el.name}}</a-avatar>
                <div>{{el.name}}</div>
              </div>
            </li>
          </ul>

        </div>
        <div class="content" ref="content"></div>
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
  </div>
</template>
<script>
import { Input,Avatar } from "ant-design-vue";
import Vue from "vue";
Vue.use(Input);
Vue.use(Avatar)
export default {
  data() {
    return {
      textValue: "",
      ws: null,
      userName: "",
      isOpen: false,
      userList:[]
    };
  },
  methods: {
    handlePressEnter() {
      const message = this.textValue;
      if (this.ws) {
        this.ws.send(message + "/**/" + this.userName);
      }
      this.textValue = "";
    },
    handleConnect() {
      if (!this.userName) {
        this.$message.error("用户名不能为空!");
        return;
      }
      this.ws = new WebSocket(`ws://localhost:5000/${this.userName}`);
      this.ws.onopen = () => {
        this.$message.info("连接成功");
      };
      this.ws.onmessage = (event) => {
        let name = event.data.split("/**/")[1];
        let message = event.data.split("/**/")[0];
        if( name === undefined ){
          this.userList = JSON.parse(event.data)
          console.log('有人断开了');
          return
        }
        if (name === this.userName) {
          this.$refs.content.innerHTML += `
          <div class="me">
            <div class="meMessageBox">
              <span class="meName">${name}</span>
              <span class="message">${message}</span>
            </div>
          </div>
          `;
        } else {
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
        this.$refs.content.innerHTML +=
          "<div class='unconnect'>----------你已断开连接----------</div>";
      };
    },
    handleClose() {
      this.$message.info("连接断开");
      if (this.ws) {
        this.ws.send("close Connction" + "/" + this.userName);
        this.ws.close();
      }
      this.ws = null;
      this.userList.forEach(element => {
        if(element.name === this.userName){
          element.status = 0
          return
        }
      });
    },
    openGroup() {
      this.isOpen = !this.isOpen;
      // if (this.isOpen) {
      //   this.$refs.group.style.width = 260 + "px";
      //   console.log(this.$refs.group.style);
      // } else {
      //   this.$refs.group.style.width = 0 + "px";
      //   console.log(this.$refs.group.style);
      // }
    },
  },
};
</script>
<style lang="less">
#container {
  margin: 0 auto;
  margin-top: 50px;
  padding: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  .connect-box {
    display: flex;
    gap: 8px;
  }
  .room {
    width: 920px;
    margin: 0 auto;
    display: flex;
    border: 1px solid #706767;
    background-color: rgba(15, 34, 67, 0.05);
    .right {
      width: 260px;
      height: 605px;
      border-right: 1px solid #706767;
    }
    .left {
      position: relative;
      overflow: hidden;
      .roomName {
        display: flex;
        align-items: center;
        justify-content: space-between;
        width: 660px;
        height: 55px;
        border-bottom: 1px solid #706767;
        padding: 0 25px;
        span {
          color: black;
        }
        .tool {
          font-size: 20px;
          color: #706767;
        }
        .tool:hover {
          cursor: pointer;
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
        .unconnect{
          text-align: center;
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
      .group {
        padding: 0px;
        ul{
          list-style: none;
          margin: 0;
          padding: 0;
          display: flex;
          flex-wrap: wrap;
          justify-content: space-evenly;
          column-gap: 20px;
          row-gap: 10px;
          padding-bottom: 16px;
          border-bottom: 1px solid #d1c5c5;
          li{
            .user{
              width: 32px;
              height: 50px;
              div{
                text-align: center;
              }
            }
          }
        }
      }
    }
  }
}
.hideBox {
  position: absolute;
  right: -260px;
  top: 55px;
  width: 260px;
  height: 550px;
  transition: all 0.5s;
  background: white;
  z-index: 1;
}
.showBox {
  position: absolute;
  right: 0px;
  top: 55px;
  width: 260px;
  height: 550px;
  transition: all 0.5s;
  background: white;
  border-left: 1px solid #706767;
  z-index: 1;
}
</style>