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
          func = 'apply';
          drawer = !drawer;
        "
        >挂号系统</v-btn
      >
      <v-btn
        text
        block
        @click="
          func = 'introduction';
          drawer = !drawer;
        "
        >医生介绍</v-btn
      >

      <v-btn
        text
        block
        @click="
          func = 'admin';
          drawer = !drawer;
        "
        v-show="role === 'admin'"
        >管理后台</v-btn
      >
    </v-navigation-drawer>

    <v-app-bar app>
      <v-app-bar-nav-icon @click="drawer = !drawer"></v-app-bar-nav-icon>

      <v-toolbar-title>医院门户</v-toolbar-title>
      <v-spacer></v-spacer>
      <div v-if="loggedIn">
        <v-btn text rounded @click="logout">登出</v-btn><Info />
      </div>
      <div v-else><Login /><Reg /></div>
    </v-app-bar>

    <v-main>
      <v-expand-transition>
        <div v-if="func === 'apply'"><Apply /></div>
      </v-expand-transition>
      <v-expand-transition
        ><div v-if="func === 'introduction'"><Introduction /></div
      ></v-expand-transition>
      <v-expand-transition
        ><div v-if="func === 'admin' && role === 'admin'"><Admin /></div
      ></v-expand-transition>
    </v-main>
  </v-app>
</template>

<script>
import Login from "./components/Login";
import Reg from "./components/Reg";
import Apply from "./components/Apply";
import Introduction from "./components/Introduction";
import Admin from "./components/Admin";
import Info from "./components/Info";
import { get } from "./utils/http";

export default {
  data: () => ({
    drawer: null,
    test: "",
    loggedIn: false,
    reg: false,
    networkError: false,
    timer: "",
    func: "introduction",
    role: "normal",
  }),
  methods: {
    loginCheck: function () {
      get("verify").then(
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
      this.admin();
    },
    logout: function () {
      localStorage.clear();
      location.reload();
    },
    admin: function () {
      get("admin/check").then((res) => {
        this.role = res.data.msg;
      });
    },
  },
  mounted() {
    this.loginCheck();
    this.timer = setInterval(this.loginCheck, 50 * 1000);
  },
  components: {
    Login,
    Reg,
    Apply,
    Introduction,
    Admin,
    Info,
  },
};
</script>