<template>
  <div>
    <!-- <v-app-bar color="#003b8f" dark>
      <v-toolbar-title> <h3>Georeferenciamento de dispositivos IoT</h3> </v-toolbar-title>
      <v-spacer></v-spacer>
    </v-app-bar> -->

    <l-map
      ref="map"
      id="map"
      :zoom="zoom"
      :options="mapOptions"
      :center="center"
      :minZoom="min"
      style="position: fixed !important; padding-top: 64px"
    >
      <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>

      <l-control position="topleft">
        <v-img
          :width="120"
          :src="require(`../../static/img/logo2.png`)"
        ></v-img>
      </l-control>

      <l-control position="bottomleft">
        <v-col>
          <v-card>
            <v-list-item>
                <h2>Legenda</h2>
            </v-list-item>
              <template v-for="(item, index) in itemsLegenda">
                <v-list-item v-if="item.action" :key="item.title">
                  <v-list-item-action>
                    <v-icon :color="item.color">{{ item.action }}</v-icon>
                  </v-list-item-action>

                  <v-list-item-content>
                    <v-list-item-title>{{ item.title }}</v-list-item-title>
                  </v-list-item-content>
                </v-list-item>

                <v-divider
                  v-else-if="item.divider"
                  :key="index"
                  :inset="inset"
                ></v-divider>
              </template>
          </v-card>
        </v-col>
      </l-control>

      <l-control-zoom :position="zoomPosition" />

      <v-marker-cluster>
        <template v-for="(marker, index) in markers">
          <l-marker
            :key="index"
            :lat-lng="marker.latlng"
            @click="callRightNav(marker)"
            :icon="showIcon(marker)"
          >
            <l-tooltip>{{ marker.id }}</l-tooltip>
          </l-marker>
        </template>
      </v-marker-cluster>
    </l-map>

    <v-navigation-drawer
      v-model="drawer"
      :width="500"
      fixed
      app
      right
      hide-overlay
    >
      <template v-slot:prepend>
        <v-list-item two-line>
          <v-list-item-content>
            <v-list-item-title>
              <v-btn @click="closeDrawer(false)" icon>
                <v-icon dark left> mdi-arrow-right </v-icon>
              </v-btn>
               Informações
            </v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </template>

      <v-divider></v-divider>

      <template>
        <v-simple-table dark>
          <template v-slot:default>
            <thead>
              <tr>
                <th class="text-left">Atributo</th>
                <th class="text-left">Valor</th>
              </tr>
            </thead>
            <tbody>
              <template v-for="(item, key) in items">
                <tr :key="item" v-if="item.type === 'Property'">
                  <td>{{ formatInfomationField(key) }}</td>
                  <td>{{ item.value }}</td>
                </tr>
              </template>
            </tbody>
          </template>
        </v-simple-table>
      </template>

      <!-- <div class="row">
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
      </div> -->
    </v-navigation-drawer>
  </div>
</template>

<script>
import SockJS from "sockjs-client";
import Stomp from "webstomp-client";
import {
  LMap,
  LTileLayer,
  LMarker,
  LTooltip,
  LControlZoom,
  LControl,
} from "vue2-leaflet";
import Vue2LeafletMarkerCluster from "vue2-leaflet-markercluster";
import "leaflet.markercluster";
import L from "leaflet";
import Cookie from "js-cookie";
import mapa from "../../static/img/mapa.png";
import mapa2 from "../../static/img/mapa2.png";

export default {
  name: "Home",
  components: {
    LMap,
    LTileLayer,
    LControlZoom,
    LMarker,
    "v-marker-cluster": Vue2LeafletMarkerCluster,
    LTooltip,
    LControl,
  },
  data() {
    return {
      // Mapa
      url: "https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",
      attribution:
        '&copy; <a target="_blank" href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      zoom: 4,
      center: [-15.20707, -45.34524],
      min: 0,
      markers: [],
      zoomPosition: "topright",
      mapOptions: {
        zoomControl: false,
        attributionControl: false,
        measureControl: true,
      },
      // websocket
      stompClient: null,
      messages: [],

      // navigation drawer
      drawer: false,
      group: null,
      items: null,

      // legenda
      itemsLegenda: [
        {
          action: "mdi-map-marker",
          title: "Sem alteração recente no dispositivo",
          color: "#564b4c",
        },
        {
          divider: true,
        },
        {
          action: "mdi-map-marker",
          title: "Houve alguma alteração no dispositivo",
          color: "#a100b4",
        },
      ],
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
              this.markers.push({
                id: element.id,
                latlng: marker._latlng,
                modification: false,
              });
            }
          });
        });
    },
    showIcon: (layer) => {
      console.log("Layer", layer);
      if (layer.modification) {
        return L.icon({
          iconUrl: mapa2,
          iconSize: [38, 42],
        });
      } else {
        return L.icon({
          iconUrl: mapa,
          iconSize: [38, 42],
        });
      }
    },
    closeDrawer(value) {
      this.drawer = value;
    },
    callRightNav(element) {
      let url =
        "http://localhost:8080/sgeol-dm/v2/fazenda/find-by-id?entity-id=" +
        element.id;
      console.log("URL", url);
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
          this.items = res;
        });

      element.modification = false;
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
    startTask: function () {
      if (this.stompClient != null) {
        this.stompClient.send("/ws/start");
      } else {
        alert("Please connect first");
      }
    },
    handleMessageReceipt: function (messageOutput) {
      console.log("Mensagem recebida", messageOutput);
      for (let i = 0; i < this.markers.length; i++) {
        if (messageOutput === this.markers[i].id) {
          console.log("OK", this.markers[i]);
          this.markers[i].modification = true;
        }
      }
      this.messages.push(messageOutput);
    },
    formatInfomationField(str) {
      let strUpperCaseOne = str[0].toUpperCase() + str.slice(1);
      return strUpperCaseOne.replace(/_/g, " ");
    },
  },
  mounted() {
    this.$nextTick(() => {
      this.map = this.$refs.map.mapObject;
    });
  },
  async created() {
    this.connect();
    this.getLayer("http://127.0.0.1:8080/sgeol-dm/v2/fazenda");
  },
};
</script>
