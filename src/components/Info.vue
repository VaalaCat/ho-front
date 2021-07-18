<template>
  <v-dialog v-model="dialog" width="500">
    <template v-slot:activator="{ on, attrs }">
      <v-btn text rounded v-bind="attrs" v-on="on"> 个人信息 </v-btn>
    </template>

    <v-card>
      <v-form ref="form">
        <v-container>
          <v-text-field
            v-model="oldPassword"
            label="原密码"
            type="password"
            required
          />

          <v-text-field
            v-model="newPassword"
            label="新密码"
            type="password"
            required
          />

          <v-expand-transition>
            <v-col v-if="loginError">
              <v-alert color="red" type="error"> 密码错误 </v-alert>
            </v-col>
          </v-expand-transition>

          <v-expand-transition>
            <v-col v-if="loginSuccessful">
              <v-alert color="green" type="success"> 修改成功 </v-alert>
            </v-col>
          </v-expand-transition>

          <v-expand-transition>
            <v-col v-if="networkError">
              <v-alert color="red" type="error"> 网络异常 </v-alert>
            </v-col>
          </v-expand-transition>

          <v-btn color="success" @click="submit"> 修改 </v-btn>

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
  name: "Info",
  data() {
    return {
      email: "",
      oldPassword: "",
      newPassword: "",
      loginError: false,
      loginSuccessful: false,
      networkError: false,
      errorMessage: "",
      dialog: false,
      passwordRules: [(v) => !!v || "Password is required"],
    };
  },
  methods: {
    submit: function () {
      let check = this.$refs.form.validate();
      if (check) {
        post("admin/changepwd", {
          oldPasswd: this.oldPassword,
          newPasswd: this.newPassword,
        }).then(
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