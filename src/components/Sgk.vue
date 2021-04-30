<template>
  <v-app>
    <v-container>
      <v-form ref="form">
        <v-text-field
          v-model="data"
          label="QQ|微博UID|电话|邮箱"
          type="text"
          :rules="dataRules"
          required
        />
        <v-expand-transition>
          <v-alert text v-if="ans !== '' && ans !== 'null'">{{
            ans
          }}</v-alert></v-expand-transition
        >
        <v-expand-transition>
          <v-alert text v-if="ans === 'null'" type="error">
            没有数据
          </v-alert></v-expand-transition
        >
        <v-btn @click="submit"> 查询 </v-btn>
      </v-form>
    </v-container>
  </v-app>
</template>

<script>
import { get } from "../utils/http";

export default {
  name: "Sgk",
  data() {
    return {
      data: "",
      ans: "",
      dataRules: [
        (v) => !!v || "Data is required",
        (v) => v.length <= 50 || "Data must less than 50 characters",
      ],
    };
  },
  methods: {
    submit: function () {
      let check = this.$refs.form.validate();
      if (check) {
        get("sgk", { data: this.data }).then((res) => {
          if (res.data.code == "0") {
            this.ans = res.data.msg;
          } else {
            this.ans = "null";
          }
        });
      }
    },
    reset: function () {
      this.$refs.form.reset();
    },
  },
};
</script>