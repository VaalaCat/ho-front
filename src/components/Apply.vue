<template>
  <v-container>
    <v-row v-if="role == 'normal'">
      <v-col class="d-flex" sm="4">
        <v-menu
          v-model="menu1"
          :close-on-content-click="false"
          :nudge-right="40"
          transition="scale-transition"
          offset-y
          min-width="auto"
        >
          <template v-slot:activator="{ on, attrs }">
            <v-text-field
              v-model="date"
              label="选择预约日期"
              prepend-icon="mdi-calendar"
              readonly
              v-bind="attrs"
              v-on="on"
            ></v-text-field>
          </template>
          <v-date-picker v-model="date" @input="menu1 = false"></v-date-picker>
        </v-menu>
      </v-col>
      <v-col class="d-flex" sm="4">
        <v-menu
          ref="menu"
          v-model="menu2"
          :close-on-content-click="false"
          :nudge-right="40"
          :return-value.sync="time"
          transition="scale-transition"
          offset-y
          max-width="290px"
          min-width="290px"
        >
          <template v-slot:activator="{ on, attrs }">
            <v-text-field
              v-model="time"
              label="选择预约时间"
              prepend-icon="mdi-clock-time-four-outline"
              readonly
              v-bind="attrs"
              v-on="on"
            ></v-text-field>
          </template>
          <v-time-picker
            v-if="menu2"
            v-model="time"
            full-width
            @click:minute="$refs.menu.save(time)"
          ></v-time-picker>
        </v-menu>
      </v-col>
      <v-col class="d-flex" sm="2">
        <v-select
          :items="Object.keys(doctorlist)"
          label="医生"
          v-model="doc"
        ></v-select>
      </v-col>
      <v-col cols="3" sm="2"><v-btn @click="submit" large>预约</v-btn></v-col>
    </v-row>
    <v-row v-if="role == 'normal'">
      <v-spacer></v-spacer>
      <v-col class="d-flex" sm="4"> </v-col>
    </v-row>
    <v-spacer></v-spacer>

    <v-data-table
      v-model="selected"
      :headers="headers"
      :items="applylist"
      :single-select="singleSelect"
      item-key="id"
      show-select
      class="elevation-1"
    >
      <template v-slot:top>
        <v-toolbar flat>
          <v-switch v-model="singleSelect" label="单选" class="pa-3"></v-switch>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>
          <v-btn @click="show">刷新</v-btn>
          <v-btn @click="disallow" v-if="role == 'normal'">取消</v-btn>
          <v-btn @click="disallow" v-if="role == 'doctor'">驳回</v-btn>
          <v-btn @click="accept" v-if="role == 'doctor'">接受</v-btn>
        </v-toolbar>
      </template>
    </v-data-table>
    <v-col v-if="!role"
      ><v-alert color="red" type="error"> 此功能需要登陆 </v-alert></v-col
    >
    <v-col v-if="networkError"
      ><v-alert color="red" type="error"> 网络错误 </v-alert></v-col
    >
    <v-col v-if="optsucc"
      ><v-alert color="green" type="success"> 操作成功 </v-alert></v-col
    >
    <v-col v-if="applylist.length == 0 && role"
      ><v-alert color="blue"> 您目前没有预约 </v-alert></v-col
    >
    <v-col v-if="ans != ''"
      ><v-alert color="green" type="success"> 预约成功 </v-alert></v-col
    >
  </v-container>
</template>

<script>
import { get, post } from "../utils/http";

export default {
  name: "Apply",
  data() {
    return {
      ans: "",
      time: null,
      date: null,
      menu1: false,
      menu2: false,
      applylist: [],
      role: false,
      selected: [],
      networkError: false,
      optsucc: false,
      doctorlist: {},
      doc: "",
      headers: [
        {
          text: "挂号ID",
          align: "start",
          sortable: false,
          value: "id",
        },
        { text: "预约时间", value: "atime" },
        { text: "处理状态", value: "archivetime" },
        { text: "患者名字", value: "nname" },
        { text: "医生名字", value: "dname" },
      ],
    };
  },
  methods: {
    submit: function () {
      post("apply/add", {
        did: this.doctorlist[this.doc],
        atime: this.date + " " + this.time,
      }).then((res) => {
        if (res.data.code == "0") {
          this.ans = res.data.msg;
        } else {
          this.ans = "null";
        }
      });
    },
    show: function () {
      this.optsucc = false;
      this.ans = "";
      get("apply/show").then((res) => {
        console.log(res.data);
        this.applylist = res.data.msg;
        this.role = res.data.role;
      });
      get("introduction/all").then((res) => {
        this.doctorlist = res.data.msg;
        console.log(res.data.msg);
      });
    },
    accept: function () {
      post("apply/opt", { func: "accept", list: this.selected }).then(
        (res) => {
          console.log(res.data);
          this.networkError = false;
          if (res.data.code == "0") {
            this.optsucc = true;
          }
        },
        (err) => {
          if (err.status == 401) {
            this.role = false;
            this.networkError = false;
          } else {
            this.networkError = true;
          }
        }
      );
    },
    disallow: function () {
      post("apply/opt", { func: "disallow", list: this.selected }).then(
        (res) => {
          console.log(res.data);
          this.networkError = false;
          if (res.data.code == "0") {
            this.optsucc = true;
          }
        },
        (err) => {
          if (err.status == 401) {
            this.role = false;
            this.networkError = false;
            this.optsucc = false;
          } else {
            this.networkError = true;
            this.optsucc = false;
          }
        }
      );
    },
    reset: function () {
      this.$refs.form.reset();
    },
  },
  mounted: function () {
    this.show();
  },
};
</script>