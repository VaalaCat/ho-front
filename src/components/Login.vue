<template>
  <v-dialog v-model="dialog" width="500">
    <template v-slot:activator="{ on, attrs }">
      <v-btn text rounded v-bind="attrs" v-on="on"> 登录 </v-btn>
    </template>

    <v-card>
      <v-form ref="form">
        <v-container>
          <v-text-field
            v-model="email"
            :rules="emailRules"
            label="E-mail"
            required
          />

          <v-text-field
            v-model="password"
            label="Password"
            type="password"
            :rules="passwordRules"
            required
          />

          <v-expand-transition>
            <v-col v-if="loginError">
              <v-alert color="red" type="error"> 用户名或密码错误 </v-alert>
            </v-col>
          </v-expand-transition>

          <v-expand-transition>
            <v-col v-if="loginSuccessful">
              <v-alert color="green" type="success"> 登录成功 </v-alert>
            </v-col>
          </v-expand-transition>

          <v-expand-transition>
            <v-col v-if="networkError">
              <v-alert color="red" type="error"> 网络异常 </v-alert>
            </v-col>
          </v-expand-transition>

          <v-btn color="success" @click="submit"> 登录 </v-btn>

          <v-btn color="error" @click="reset"> 重置 </v-btn>
        </v-container>
      </v-form>

      <v-divider></v-divider>

      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn color="primary" text @click="dialog = false"> 关闭 </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
import { post } from "../utils/http";

export default {
  name: "Login",
  data() {
    return {
      email: "",
      password: "",
      loginError: false,
      loginSuccessful: false,
      networkError: false,
      errorMessage: "",
      dialog: false,
      emailRules: [
        (v) => !!v || "E-mail is required",
        (v) => /.+@.+/.test(v) || "E-mail must be valid",
      ],
      passwordRules: [(v) => !!v || "Password is required"],
    };
  },
  methods: {
    submit: function () {
      let check = this.$refs.form.validate();
      if (check) {
        post("login", { email: this.email, password: this.password }).then(
          (res) => {
            if (res.data.code == "0") {
              this.loginSuccessful = true;
              this.loginError = false;
              setTimeout(() => {
                location.reload();
              }, 200);
            } else {
              this.loginSuccessful = false;
              this.loginError = true;
            }
            this.networkError = false;
          },
          (err) => {
            if (err.status == 401) {
              this.loginSuccessful = false;
              this.loginError = true;
              this.networkError = false;
            } else {
              this.networkError = true;
            }
          }
        );
      }
    },
    reset: function () {
      this.$refs.form.reset();
    },
  },
};
</script>