<template>
  <div>
    <q-layout view="lHh Lpr lFf">
      <q-drawer
        v-model="drawerLeft"
        show-if-above
        :width="200"
        :breakpoint="700"
        elevated
        content-class="bg-primary text-white"
      >
        <q-scroll-area class="fit">
          <div
            v-for="(prize, index) in prizes"
            :key="prize.num"
            dense
            @click="select(prize, index)"
          >
            # {{ prize.name }}
          </div>
        </q-scroll-area>
      </q-drawer>
      <q-page-container>
        <q-page padding>
          <q-header elevated class="bg-orange text-white">
            <div class="q-pa-md q-gutter-y-sm">
              <q-toolbar>
                <q-toolbar-title class="text-center"
                  >奖项名称: {{ prizeName }}</q-toolbar-title
                >
              </q-toolbar>
            </div>
          </q-header>
          <div class="q-gutter-md">
            <q-btn
              color="deep-orange"
              v-for="winner in this.winners"
              :label="winner.name"
              :key="`xs-${winner.id}`"
            ></q-btn>
          </div>
          <q-footer elevated class="bg-orange">
            <q-toolbar class="justify-center q-pa-md">
              <q-btn color="red" @click="start"> 开始抽奖 </q-btn>
              <q-btn color="red" @click="stop"> 停止 </q-btn>
            </q-toolbar>
          </q-footer>
        </q-page>
      </q-page-container>
    </q-layout>
  </div>
</template>

<script>
export default {
  data() {
    return {
      winners: [],
      prizes: [],
      index: {},
      num: 0,
      drawerLeft: false,
      prizeName: "",
      Index: 0,
      url: "ws://192.168.1.119:80/ws",
      conn: {},
      namelist: [],
      action: ""
    };
  },

  created() {
    this.initWebSocket();

    //
    //   this.conn.onmessage = function(evt) {
    // this.prizes = evt.data;
    // };
  },

  destroyed() {
    this.conn.close();
  },

  methods: {
    initWebSocket() {
      this.conn = new WebSocket(this.url);
      this.conn.onopen = this.webSocketOnOpen;
      this.conn.onmessage = this.webSocketOnMessage;
      this.conn.onclose = this.webSocketOnClose;
    },

    webSocketOnOpen() {
      this.conn.send('{"name":"listprizes", "prizeindex":0}');
    },

    webSocketOnMessage(msg) {
      var data = JSON.parse(msg.data);
      switch (data.action.name) {
        case "listprizes":
          this.prizes = data.prizes;
          break;
        //  case "getwinners":
        //    this.winners = data.winners;
        //    break;
        case "start":
          this.winners = data.winners;

          break;
        case "stop":
          this.winners = data.winners;
          break;
        case "getWinners":
          this.winners =
            data.winners.length === 0 ? new Array(this.num) : data.winners;
          break;
      }
    },
    webSocketOnClose() {},

    select(prize, index) {
      this.prizeName = prize.name;
      this.num = prize.num;
      this.Index = index;

      this.conn.send('{"name":"getWinners", "prize_index":' + this.Index + "}");
    },

    start() {
      this.conn.send('{"name":"start","prize_index":' + this.Index + "}");
    },
    stop() {
      this.conn.send('{"name":"stop","prize_index":' + this.Index + "}");
    }
  }
};
</script>
