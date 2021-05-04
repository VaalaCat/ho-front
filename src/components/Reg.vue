<template>
  <v-dialog v-model="dialog" width="500">
    <template v-slot:activator="{ on, attrs }">
      <v-btn text rounded v-bind="attrs" v-on="on"> 注册 </v-btn>
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

          <v-text-field
            v-model="phone"
            label="Phone"
            type="text"
            :rules="phoneRules"
            required
          />

          <v-text-field
            v-model="name"
            label="Name"
            type="text"
            :rules="nameRules"
            required
          />

          <v-expand-transition>
            <v-col v-if="regError">
              <v-alert color="red" type="error"> 注册失败 </v-alert>
            </v-col>
          </v-expand-transition>

          <v-expand-transition>
            <v-col v-if="regSuccessful">
              <v-alert color="green" type="success"> 注册成功 </v-alert>
            </v-col>
          </v-expand-transition>

          <v-expand-transition>
            <v-col v-if="networkError">
              <v-alert color="red" type="error"> 网络异常 </v-alert>
            </v-col>
          </v-expand-transition>

          <v-btn color="success" @click="submit"> 注册 </v-btn>
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
  name: "Reg",
  data() {
    return {
      email: "",
      password: "",
      name: "",
      phone: "",
      regError: false,
      regSuccessful: false,
      networkError: false,
      errorMessage: "",
      dialog: false,
      nameRules: [
        (v) => !!v || "Name is required",
        (v) => v.length <= 10 || "Name must be less than 10 characters",
      ],
      emailRules: [
        (v) => !!v || "E-mail is required",
        (v) => /.+@.+/.test(v) || "E-mail must be valid",
      ],
      passwordRules: [(v) => !!v || "Password is required"],
      phoneRules: [
        (v) => !!v || "Phone Number is required",
        (v) => !/\D/.test(v) || "Must be all number",
        (v) => v.length == 11 || "Phone number must be 11 num",
      ],
    };
  },
  methods: {
    submit: function () {
      let check = this.$refs.form.validate();
      if (check) {
        post("reg", {
          email: this.email,
          password: this.password,
          phone: this.phone,
          name: this.name,
        }).then(
          (res) => {
            console.log(res.data);
            if (res.data.code == "0") {
              this.regSuccessful = true;
              this.regError = false;
            } else {
              this.regSuccessful = false;
              this.regError = true;
            }
            this.networkError = false;
          },
          (err) => {
            if (err.status == 401) {
              this.regSuccessful = false;
              this.regError = true;
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