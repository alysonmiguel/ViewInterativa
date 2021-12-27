<template>
<div class="content" id="websocket">
  <div class="row" >
    <div class="col">
      <button class="btn btn-sm btn-info" @click="connect">Create connection</button>
      <button class="btn btn-sm btn-success" @click="startTask">Start Task</button>
      <button class="btn btn-sm btn-danger" @click="stopTask">Stop Task</button>
      <button class="btn btn-sm btn-primary" @click="disconnect">Close connection</button>
    </div>
  </div>
  <div class="row">
    <div class="col">
      <ul class="list-group" style="height: 500px; overflow:scroll;">
        <li class="list-group-item d-flex justify-content-between align-items-center"
            v-for="(m,idx) in messages" :key="'m-'+idx">
          {{m}}
        </li>
      </ul>
    </div>
  </div>
</div>
</template>

<script>
import SockJS from "sockjs-client";
import Stomp from "stompjs";

export default {
  name: "Home",
  data() {
    return {
      stompClient: null,
      messages: [],
    };
  },
  methods: {
    connect: function () {
      var socket = new SockJS("/connect");
      this.stompClient = Stomp.over(socket);
      var that = this;
      this.stompClient.connect({}, () => {
        that.handleMessageReceipt("Connected");
        this.stompClient.subscribe("/topic/messages", function (messageOutput) {
          that.handleMessageReceipt(messageOutput.body);
        });
      });
    },
    disconnect: function () {
      if (this.stompClient != null) {
        this.stompClient.disconnect();
      }
      this.handleMessageReceipt("Disconnected");
    },
    startTask: function () {
      if (this.stompClient != null) {
        this.stompClient.send("/ws/start");
      } else {
        alert("Please connect first");
      }
    },
    stopTask: function () {
      if (this.stompClient != null) {
        this.stompClient.send("/ws/stop");
      } else {
        alert("Please connect first");
      }
    },
    handleMessageReceipt: function (messageOutput) {
      this.messages.push(messageOutput);
    },
  },
  created() {
    // this.connect();
  },
};
</script>
