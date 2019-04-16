<template>
  <div class="hello">
    <h2>Login</h2>username
    <input type="text" v-model="username">
    <p class="redclass">{{dataCheck.usermsg}}</p>
    <br>password
    <input type="text" v-model="password">
    <p class="redclass">{{dataCheck.pwdmsg}}</p>
    <br>

    <button @click=" checkInfo()">login</button>
    <br>
    <br>
    <div v-if="resError" class="redclass">登入錯誤(Error code 401)</div>
  </div>
</template>

<script>
const axios = require("axios");
export default {
  name: "HelloWorld",
  data() {
    return {
      // 帳號
      username: "",
      // 密碼
      password: "",
      // response 錯誤
      resError: false,
      // 檢核錯誤訊息
      dataCheck: {
        status: false,
        usermsg: "",
        pwdmsg: ""
      }
    };
  },
  methods: {
    checkInfo() {
      let reg = /['"#$%^*]/g;
      if (this.username === "") {
        this.dataCheck.usermsg = "尚未輸入帳號";
        this.dataCheck.status = true;
      } else if (reg.exec(this.username) !== null) {
        this.dataCheck.usermsg = "請勿輸入符號：'\"#$%^*";
        this.dataCheck.status = true;
      } else {
        this.dataCheck.usermsg = "";
      }
      if (this.password === "") {
        this.dataCheck.pwdmsg = "尚未輸入密碼";
        this.dataCheck.status = true;
      } else if (reg.exec(this.password) !== null) {
        this.dataCheck.pwdmsg = "請勿輸入符號：'\"#$%^*";
        this.dataCheck.status = true;
      } else {
        this.dataCheck.usermsg = "";
      }

      if (!this.dataCheck.status) {
        this.postApi();
      }
    },
    postApi() {
      axios({
        method: "post", //you can set what request you want to be
        url: "https://earth.comismart.com/auth/rest/token",
        data: { login: this.username, password: this.password },
        headers: {
          "Content-Type": "application/json",
          'Accept': "application/json"
        }
      }).then(
        response => {
          console.log("response", response.data);
          this.$router.push({ name: 'thermometer',query: response.data});
        },
        error => {
          this.password = "";
          this.username = "";
          this.resError = true;
          console.log("error", error);
        }
      );
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1,
h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.redclass {
  color: red;
}
</style>
