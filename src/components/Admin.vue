<template>
  <v-container>
    <v-row>
      <v-col class="d-flex" sm="4"
        ><v-text-field
          hint="模糊查询邮箱地址"
          label="搜索用户"
          v-model="searchEmail"
        ></v-text-field
      ></v-col>
      <v-col class="d-flex" sm="5">
        <v-btn large @click="searchSubmit">搜索</v-btn>
        <v-btn large @click="searchApply">挂号查询</v-btn>
        <v-dialog v-model="dialogRst" persistent max-width="290">
          <template v-slot:activator="{ on, attrs }">
            <v-btn large v-bind="attrs" v-on="on"> 重置密码 </v-btn>
          </template>
          <v-card>
            <v-card-title class="text-h5"> 确定重置密码？ </v-card-title>
            <v-card-text>将会重置选中所有用户的密码，确认重置？</v-card-text>
            <v-form
              ><v-container
                ><v-text-field
                  v-model="newPassword"
                  label="Password"
                  type="password"
                  required /></v-container
            ></v-form>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="green darken-1" text @click="dialogRst = false">
                取消
              </v-btn>
              <v-btn
                color="green darken-1"
                text
                @click="
                  dialogRst = false;
                  resetPwd();
                "
              >
                确定
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>

        <v-dialog v-model="dialogDel" persistent max-width="290">
          <template v-slot:activator="{ on, attrs }">
            <v-btn large v-bind="attrs" v-on="on"> 删除用户 </v-btn>
          </template>
          <v-card>
            <v-card-title class="text-h5"> 确定删除用户？ </v-card-title>
            <v-card-text>删除用户将会删除所有相关信息，请谨慎操作</v-card-text>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="green darken-1" text @click="dialogDel = false">
                取消
              </v-btn>
              <v-btn
                color="green darken-1"
                text
                @click="
                  dialogDel = false;
                  deleteUsers();
                "
              >
                确定
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog v-model="dialog" width="500">
          <template v-slot:activator="{ on, attrs }">
            <v-btn large v-bind="attrs" v-on="on"> 修改 </v-btn>
          </template>

          <v-card>
            <v-form ref="form">
              <v-container>
                <v-text-field v-model="updateEmail" label="E-mail" required />
                <v-text-field v-model="updatePhone" label="Phone" required />
                <v-text-field v-model="updateName" label="Name" required />
                <v-select
                  :items="roles"
                  label="Roles"
                  v-model="updateRole"
                ></v-select>

                <v-text-field
                  v-model="updateAff"
                  label="Aff"
                  v-if="updateRole == 'doctor'"
                />
                <v-text-field
                  v-model="updateInfo"
                  label="Info"
                  v-if="updateRole == 'doctor'"
                />

                <v-btn color="success" @click="updateUser"> 提交 </v-btn>

                <v-btn color="blue" @click="getInfo"> 复制 </v-btn>
                <v-btn color="red" @click="reset"> 重置 </v-btn>
              </v-container>
            </v-form>

            <v-divider></v-divider>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="primary" text @click="dialog = false"> 关闭 </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog></v-col
      >
    </v-row>
    <v-data-iterator :items="ansUsers">
      <template v-slot:default="props">
        <v-row>
          <v-col
            v-for="item in props.items"
            :key="item.name"
            cols="12"
            sm="6"
            md="4"
            lg="3"
          >
            <v-card>
              <v-card-title class="subheading font-weight-bold">
                <v-col
                  ><v-checkbox
                    v-model="selectedUsers"
                    :value="item.id"
                    :label="item.name"
                    >{{ item.email }}</v-checkbox
                  ></v-col
                >
              </v-card-title>

              <v-divider></v-divider>

              <v-list dense>
                <v-list-item>
                  <v-list-item-content>email:</v-list-item-content>
                  <v-list-item-content class="align-end">
                    {{ item.email }}
                  </v-list-item-content>
                </v-list-item>

                <v-list-item>
                  <v-list-item-content>phone:</v-list-item-content>
                  <v-list-item-content class="align-end">
                    {{ item.phone }}
                  </v-list-item-content>
                </v-list-item>

                <v-list-item>
                  <v-list-item-content>role:</v-list-item-content>
                  <v-list-item-content class="align-end">
                    {{ item.role }}
                  </v-list-item-content>
                </v-list-item>
              </v-list>
            </v-card>
          </v-col>
        </v-row>
      </template>
    </v-data-iterator>
    <v-data-table
      v-model="selectedApply"
      :headers="headers"
      :items="ansApply"
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
          <v-btn @click="deleteApply">删除</v-btn>
        </v-toolbar>
      </template>
    </v-data-table>
  </v-container>
</template>

<script>
import { post } from "../utils/http";

export default {
  name: "Apply",
  data() {
    return {
      singleSelect: false,
      dialogDel: false,
      dialogRst: false,
      ansUsers: [],
      ansApply: [],
      searchEmail: "",
      dialog: false,
      selectedUsers: [],
      roles: ["admin", "normal", "doctor"],
      updateName: "",
      updateEmail: "",
      updatePhone: "",
      updateRole: "",
      originalId: "",
      newPassword: "",
      selectedApply: [],
      headers: [
        {
          text: "挂号ID",
          align: "start",
          sortable: false,
          value: "id",
        },
        { text: "预约时间", value: "atime" },
        { text: "处理时间", value: "archivetime" },
        { text: "患者名字", value: "nname" },
        { text: "医生名字", value: "dname" },
      ],
    };
  },
  methods: {
    searchSubmit: function () {
      this.ansUser = "";
      post("admin/search", { email: this.searchEmail }).then((res) => {
        if (res.data.code == "0") {
          this.ansUsers = JSON.parse(res.data.msg);
        }
      });
    },
    getInfo: function () {
      if (this.selectedUsers.length) {
        for (var i of this.ansUsers) {
          if (i.id == this.selectedUsers[0]) {
            this.updateName = i.name;
            this.updatePhone = i.phone;
            this.updateEmail = i.email;
            this.updateRole = i.role;
          }
        }
      }
    },
    resetPwd: function () {
      if (this.selectedUsers.length) {
        var changeUsers = this.ansUsers.filter((user) => {
          return this.selectedUsers.indexOf(user.id) != -1;
        });
        for (var i of changeUsers) {
          post("admin/modifypassword", {
            email: i.email,
            passwd: this.newPassword,
          });
        }
      }
    },
    updateUser: function () {
      post("admin/modify", {
        id: this.selectedUsers[0],
        email: this.updateEmail,
        role: this.updateRole,
        phone: this.updatePhone,
        name: this.updateName,
        aff: this.updateAff,
        info: this.updateInfo,
      });
    },
    deleteUsers: function () {
      var deleteUsers = this.ansUsers.filter((user) => {
        return this.selectedUsers.indexOf(user.id) != -1;
      });
      for (var i of deleteUsers) {
        post("admin/deleteuser", {
          email: i.email,
        });
      }
    },
    searchApply: function () {
      this.ansApply = [];
      if (this.selectedUsers.length) {
        let selectedUsers = this.selectedUsers;
        selectedUsers.every((user) => {
          return post("apply/show", { id: user }).then((res) => {
            if (res.data.code == "0") {
              this.ansApply = this.ansApply.concat(res.data.msg);
            }
          });
        });
      }
      var ans = [];
      var ansMap = {};
      for (var i in this.ansApply) {
        if (!ansMap[i.id]) {
          ansMap[i.id] = true;
          ans.push(i);
        }
      }
      this.ansApply = ans;
    },
    deleteApply: function () {
      if (this.selectedApply.length) {
        for (var i of this.selectedApply) {
          post("apply/delete", i);
        }
      }
    },
    reset: function () {
      this.$refs.form.reset();
    },
  },

  mounted: function () {},
};
</script>