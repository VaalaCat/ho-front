<template>
  <v-container>
    <v-row>
      <v-col class="d-flex" sm="2">
        <v-select
          :items="aff"
          label="科室"
          v-model="selected"
        ></v-select> </v-col
      ><v-col class="d-flex" sm="2" @click="show"
        ><v-btn x-large>查询</v-btn></v-col
      >
    </v-row>

    <v-spacer></v-spacer>
    <v-expansion-panels>
      <v-expansion-panel v-for="(item, i) in doctor" :key="i">
        <v-expansion-panel-header> {{ item.name }} </v-expansion-panel-header>
        <v-expansion-panel-content>
          {{ item.info }}
        </v-expansion-panel-content>
      </v-expansion-panel>
    </v-expansion-panels>
  </v-container>
</template>

<script>
import { get } from "../utils/http";

export default {
  name: "Introduction",
  data: () => {
    return {
      doctor: null,
      aff: [],
      selected: null,
    };
  },
  methods: {
    show: function () {
      get("/introduction", { aff: this.selected }).then((res) => {
        this.doctor = res.data.msg;
      });
    },
  },
  mounted() {
    get("/introduction").then((res) => {
      this.aff = res.data.msg;
    });
  },
};
</script>