<template>
  <div>
    <v-app-bar color="#003b8f" dark>
      <v-toolbar-title> <h3>Geo-IoT</h3> </v-toolbar-title>
      <v-spacer></v-spacer>
    </v-app-bar>
    <l-map
      ref="map"
      id="map"
      :zoom="zoom"
      :center="center"
      :minZoom="min"
      style="position: fixed !important; padding-top: 64px"
    >
      <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>
      <!-- <l-control-zoom :position="zoomPosition" /> -->
      <!-- <template v-for="(marker, index) in markers">
        <l-marker
          :key="index"
          :lat-lng="marker"
          :icon="showIcon()"
        />
      </template> -->
      <!-- <l-marker
        :lat-lng="markers"
        :icon="showIcon()"
        @click="callRightNav(markers)"
      /> -->
    </l-map>

    <v-navigation-drawer
      v-model="drawer"
      :width="900"
      fixed
      app
      right
      hide-overlay
    >
      <template v-slot:prepend>
        <v-list-item two-line>
          <v-list-item-content>
            <v-list-item-title>
              <v-btn v-on="on" @click="closeDrawer(false)" icon>
                <v-icon dark left> mdi-arrow-left </v-icon>
              </v-btn>
              Informações
            </v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </template>

      <v-divider></v-divider>

      <template>
        <v-simple-table>
          <template v-slot:default>
            <tbody>
              <tr>
                <td>{{ "Ultimo toque" }}</td>
                <td>{{ "Mon Dec 27 15:40:19 BRT 2021" }}</td>
              </tr>
            </tbody>
          </template>
        </v-simple-table>
      </template>
      <div class="row">
        <div class="col">
          <ul class="list-group" style="height: 500px; overflow: scroll">
            <li
              class="
                list-group-item
                d-flex
                justify-content-between
                align-items-center
              "
              v-for="(m, idx) in messages"
              :key="'m-' + idx"
            >
              {{ m }}
            </li>
          </ul>
        </div>
      </div>
    </v-navigation-drawer>
  </div>
</template>

<script>
import SockJS from "sockjs-client";
import Stomp from "webstomp-client";
import { LMap, LTileLayer } from "vue2-leaflet";
import "leaflet.markercluster";
import L from "leaflet";
import Cookie from "js-cookie";
import img from "../../static/img/defaultIcon.png";

export default {
  name: "Home",
  components: {
    LMap,
    LTileLayer,
    // LControlZoom,
    // LMarker,
  },
  data() {
    return {
      // Mapa
      url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
      attribution:
        '&copy; <a target="_blank" href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      zoom: 8,
      center: [-6.20707, -35.34524],
      min: 0,
      zoomPosition: "topright",
      options: {
        onEachFeature: (feature, layer) => {
          layer.on("click", () => {
            layer.bindTooltip(" Olá :)");
          });

          this.markers.on("click", function (a) {
            console.log("marker " + a.layer);
          });
        },
      },

      markers: L.markerClusterGroup(),
      // websocket
      stompClient: null,
      messages: [],
      // navigation drawer
      drawer: false,
      group: null,
    };
  },
  methods: {
    async getLayer(url) {
      fetch(url, {
        method: "GET",
        headers: {
          Accept: "application/json, text/plain, */*",
          "application-token": Cookie.get("app_access_token"),
          "user-token": Cookie.get("user_access_token"),
        },
      })
        .then((response) => response.json())
        .then((res) => {
          res.forEach((element) => {
            if (element.location.value.type === "Point") {
              const marker = L.marker([
                element.location.value.coordinates[1],
                element.location.value.coordinates[0],
              ]);
              this.markers.addLayer(marker);
            }
          });
          this.map.addLayer(this.markers);
        });
    },
    showIcon: () => {
      return L.icon({
        iconUrl: img,
        iconSize: [38, 42],
      });
    },
    closeDrawer(value) {
      this.drawer = value;
    },
    callRightNav(item) {
      console.log("status da nav", item);
      this.drawer = !this.drawer;
    },
    connect: function () {
      var socket = new SockJS("http://localhost:8084/connect");
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
  mounted() {
    this.$nextTick(() => {
      this.map = this.$refs.map.mapObject;
    });
  },
  async created() {
    // this.connect();
    this.getLayer("http://127.0.0.1:8080/sgeol-dm/v2/fazenda");
  },
};
</script>
