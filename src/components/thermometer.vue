<style scoped>
body {
  font-family: Helvetica Neue, Arial, sans-serif;
  font-size: 14px;
  color: #444;
}

table {
  border: 2px solid #42b983;
  border-radius: 3px;
  background-color: #fff;
}

th {
  background-color: #42b983;
  color: rgba(255, 255, 255, 0.66);
  cursor: pointer;
  -webkit-user-select: none;
  -moz-user-select: none;
  -user-select: none;
}

td {
  background-color: #f9f9f9;
}

th,
td {
  min-width: 120px;
  padding: 10px 20px;
}

th.active {
  color: #fff;
}

th.active .arrow {
  opacity: 1;
}

.arrow {
  display: inline-block;
  vertical-align: middle;
  width: 0;
  height: 0;
  margin-left: 5px;
  opacity: 0.66;
}

.arrow.asc {
  border-left: 4px solid transparent;
  border-right: 4px solid transparent;
  border-bottom: 4px solid #fff;
}

.arrow.dsc {
  border-left: 4px solid transparent;
  border-right: 4px solid transparent;
  border-top: 4px solid #fff;
}

#search {
  margin-bottom: 10px;
}
</style>
<template>
  <div>
    <table>
      <thead>
        <tr>
          <th v-for="
        (item, key, index) in data" v-if="key !== 'data'">{{ key }}</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td v-for="
        (item, key, index) in data" v-if="key !== 'data'">{{ item }}</td>
        </tr>
      </tbody>
    </table>
    <div>
      <ve-line :set-option-opts="false" :data="chartData" :data-zoom="chartDataZoom"></ve-line>
      <!-- <button @click="changeTest">change</button> -->
    </div>
  </div>
</template>

<script>
const axios = require("axios");
import Socket from "../socket";
import VeLine from "v-charts/lib/line.common";
import "echarts/lib/component/dataZoom";
export default {
  data() {
    this.chartDataZoom = [
      {
        type: "slider"
      }
    ];
    return {
      msg: "Welcome to Your Vue.js App",
      messages: [],
      token: {},
      chartData: {
        columns: ["Time", "Temperature", "Humidity", "CO2"],
        rows: []
      },
      data: {}
    };
  },
  methods: {
    handleMessage(msg) {
      console.warn("send", msg);
      // this.chartData.rows.push( { Time: "2019-04-12T05:26:08.683", Temperature: "27.8" })
      const obj = JSON.parse(msg);
      if (obj.hasOwnProperty("notification")) {
        console.warn("SUCCESS");
        this.chartData.rows.push({
          Time: obj.notification.timestamp,
          Temperature: obj.notification.parameters.Temperature.value,
          Humidity: obj.notification.parameters.Humidity.value,
          CO2: obj.notification.parameters.CO2.value
        });
        // this.chartData.rows.push({'Time':obj.notification.timestamp,'CO2':obj.notification.parameters.CO2.value, 'Temperature':obj.notification.parameters.Temperature.value, 'Humidity':msg.notification.parameters.Humidity.value});
        console.warn("this.chartData push success");
      }
    },
    changeTest() {
      this.chartData.rows.push({
        Time: "1/1",
        Temperature: Math.random() * 1000
      });
    },
    init() {
      this.token = this.$route.query;
      this.GetApi();
      Socket.$on("message", this.handleMessage);

      Socket.send(
        JSON.stringify({
          action: "authenticate",
          token: this.token.accessToken
        })
      );

      Socket.send(
        JSON.stringify({
          action: "notification/subscribe",
          deviceId: "1540281688669",
          names: ["measurements"]
        })
      );
    },
    GetApi() {
      axios({
        method: "get",
        url: "https://earth.comismart.com/api/rest/device/1540281688669",
        data: { login: this.username, password: this.password },
        headers: {
          "Content-Type": "application/json",
          Accept: "application/json",
          Authorization: "Bearer" + this.token.accessToken
        }
      }).then(
        response => {
          if(response.status === '401'){
            this.GetReload();
            return;
          }
          this.data = response.data;
          console.log("response", response);
        },
        error => {
          this.password = "";
          this.username = "";
          this.resError = true;
          console.log("error", error);
        }
      );
    },
    GetReload() {
      axios({
        method: "POST",
        url: "https://earth.comismart.com/auth/rest/token/refresh",
        data: { refreshToken: this.token.refreshToken },
        headers: {
          "Content-Type": "application/json",
          Accept: "application/json"
        }
      }).then(
        response => {
          this.token = response.accessToken;
          this.GetApi();
          console.log("responseTocken", response);
        },
        error => {
          console.log("error", error);
        }
      );
    }
  },
  created() {
    this.init();
    // window.addEventListener(
    //   "beforeunload",
    //   () => {
    //     //I CAN ACCESS TO this VARIABLE
    //     console.warn("23423432", this.$route);
    //   },
    //   false
    // );
  },
  beforeDestroy() {
    Socket.$off("message", this.handleMessage);
  },
  components: { VeLine }
};
</script>