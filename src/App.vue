<template>
  <v-app id="inspire">
    <v-navigation-drawer v-model="drawer" app>
      <v-card>
        <v-card-title>
          <v-avatar size="56">
            <img alt="user" src="https://oss.vaala.ink/img/avatar.png" />
          </v-avatar>
          <p class="ml-3">Vaala Cat</p>
          <br />
        </v-card-title>
      </v-card>
      <v-spacer></v-spacer>
      <v-btn
        text
        block
        @click="
          func = 'sgk';
          drawer = !drawer;
        "
        >社工库</v-btn
      >
    </v-navigation-drawer>

    <v-app-bar app>
      <v-app-bar-nav-icon @click="drawer = !drawer"></v-app-bar-nav-icon>

      <v-toolbar-title>VaalaCloud</v-toolbar-title>
      <v-spacer></v-spacer>
      <div v-if="loggedIn">
        <v-btn text rounded @click="logout">登出</v-btn>
      </div>
      <div v-else>
        <v-btn text rounded @click="reg = false">登录</v-btn>
        <v-btn text rounded @click="reg = true">注册</v-btn>
      </div>
    </v-app-bar>

    <v-main>
      <v-expand-transition>
        <div v-if="!loggedIn">
          <v-expand-transition v-if="!reg">
            <Login />
          </v-expand-transition>
          <v-expand-transition v-else>
            <Reg />
          </v-expand-transition>
        </div>
      </v-expand-transition>
      <v-expand-transition>
        <div v-if="func === 'sgk'">
          <Sgk />
        </div>
      </v-expand-transition>
    </v-main>
  </v-app>
</template>

<script>
import Login from "./components/Login";
import Sgk from "./components/Sgk";
import Reg from "./components/Reg";
import { get } from "./utils/http";

export default {
  data: () => ({
    drawer: null,
    test: "",
    loggedIn: false,
    reg: false,
    networkError: false,
    timer: "",
  }),
  methods: {
    loginCheck: function () {
      get("user").then(
        (res) => {
          if (res.data.code == "0") {
            this.loggedIn = true;
            this.networkError = false;
          } else {
            this.loggedIn = false;
            this.networkError = true;
          }
        },
        (err) => {
          if (err.status == 401) {
            this.loggedIn = false;
            this.networkError = true;
          }
        }
      );
    },
    logout: function () {
      localStorage.clear();
      location.reload();
    },
  },
  mounted() {
    this.loginCheck();
    this.timer = setInterval(this.loginCheck, 50 * 1000);
  },
  components: {
    Login,
    Sgk,
    Reg,
  },
};
</script>