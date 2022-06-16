<template>
  <!-- vuetify模板 -->
  <v-app>
    <!-- 头部 -->
    <v-app-bar dense disdark>
      <v-row>
        <h1 color="primary" class="mx-auto" text-center>ToDoList</h1>
      </v-row>
    </v-app-bar>
    <!-- 布局 -->
    <v-main>
      <!-- 组件 网站内容居中和水平填充-->
      <v-container>
        <!-- 表单 -->
        <v-form v-model="valid">
          <!-- v-col 的容器组件 -->
          <v-row>
            <!-- v-col 包裹内容，它必须是 v-row 的直接子代 -->
            <v-col cols="12" sm="12" md="12">
              <!-- 弹窗 -->
              <v-alert
                v-model="alert"
                border="left"
                close-text="Close Alert"
                color="deep-purple accent-4"
                dark
                dismissible
              >
                入力した文字の長さが正しくない！ </v-alert
              >
            </v-col>
            <v-col cols="12" sm="11" md="11">
              <!-- 入力框 -->
              <v-text-field
                label="名前を入力してください"
                solo
                dense
                v-model="inputtext"
              ></v-text-field>
            </v-col>
            <v-col cols="12" sm="1" md="1">
              <v-btn @click="add" depressed color="primary">
                追加
              </v-btn>
            </v-col>
            <v-col cols="12" sm="12" md="12">
              <v-menu
                ref="menu"
                v-model="menu"
                :close-on-content-click="false"
                :return-value.sync="time"
                transition="scale-transition"
                offset-y
                min-width="auto"
              >
                <template v-slot:activator="{ on, attrs }">
                  <v-text-field
                    v-model="time"
                    label="カレンダー"
                    prepend-icon="mdi-calendar"
                    readonly
                    v-bind="attrs"
                    v-on="on"
                  ></v-text-field>
                </template>
                <v-date-picker
                  v-model="time"
                  no-title
                  scrollable
                  color="red darken-1"
                  :min="nowDate"
                  max="2022-08-1"
                  locale="jp"
                >
                  <v-spacer></v-spacer>
                  <v-btn text color="primary" @click="menu = false">
                    Cancel
                  </v-btn>
                  <v-btn text color="primary" @click="$refs.menu.save(time)">
                    OK
                  </v-btn>
                </v-date-picker>
              </v-menu>
            </v-col>
            <v-col cols="12" sm="2" md="2">
              <b>未完成検索：</b>
            </v-col>
            <v-col cols="12" sm="9" md="9">
              <v-text-field
                solo
                dense
                type="text"
                class="form-control"
                placeholder="未完成todo名前のキーワード検索"
                label="未完成todo名前のキーワード検索"
                aria-describedby="button-addon2"
                v-model="findtext"
              />
            </v-col>
            <v-col cols="12" sm="1" md="1">
              <v-btn class="btn btn-outline-secondary" @click="find">
                検索
              </v-btn>
            </v-col>
            <v-col>
              <h4>全{{ total }}件中、{{ finish }}件完成した</h4>
            </v-col>
          </v-row>
          <v-simple-table height="380px" cols="12" sm="12" md="12">
            <template v-slot:default>
              <thead>
                <tr>
                  <th class="text-center">ID</th>
                  <th class="text-center">Name</th>
                  <th class="text-center">Status</th>
                  <th class="text-center">Delete</th>
                  <th class="text-center">Time</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="item in items" :key="item.id">
                  <td>{{ item.id }}</td>
                  <td>{{ item.name }}</td>
                  <td>
                    <v-btn
                      type="button"
                      class="btn btn-primary"
                      @click="$store.dispatch('changeState', { item: item })"
                    >
                      <span v-if="item.status == 0"> todo </span>
                      <span v-else class="done"> done </span>
                    </v-btn>
                  </td>
                  <td>
                    <v-btn
                      type="button"
                      class="btn btn-danger"
                      @click="$store.dispatch('remove', { item: item })"
                    >
                      削除
                    </v-btn>
                  </td>
                  <td>
                    {{ item.time }}
                  </td>
                </tr>
              </tbody>
            </template>
          </v-simple-table>
        </v-form>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
/* eslint-disable */
import { mapState } from "vuex";

export default {
  data: () => ({
    inputtext: "",
    alert: false,
    findtext: "",
    time: null,
    menu: false,
    picker: new Date(Date.now() - new Date().getTimezoneOffset() * 60000)
      .toISOString()
      .substr(0, 10),
    nowDate: new Date().toISOString().slice(0,10),
  }),

  computed: {
    ...mapState(["items"]),
    finish() {
      let sum = 0;
      for (const item of this.items) {
        sum += Number(item.status);
      }
      return sum;
    },
    total() {
      // return this.$store.state.items.length;
      return this.items.length;
    },
  },

  created() {
    this.$store.dispatch("getItems");
  },

  methods: {
    add() {
      if (
        this.inputtext !== "" &&
        this.inputtext.length <= 64 &&
        this.time !== null
      ) {
        this.$store.dispatch("add", {
          name: this.inputtext,
          status: 0,
          time: this.time,
        });
        this.inputtext = "";
        this.time = "";
      } else if (this.inputtext == "") {
        // alert("入力した文字の長さが正しくない！");
        this.alert = true;
        this.inputtext = "";
        this.time = "";
      } else {
        alert("日程を入力してください！");
      }
    },

    find() {
      console.log("find function");
      if (this.findtext !== "" && this.findtext.length <= 64) {
        console.log(this.findtext);
        this.$store.dispatch("find", this.findtext);
        this.findtext = "";
      } else {
        alert("入力した文字の長さが正しくない！");
        this.findtext = "";
      }
    },
    allowedDates: (val) => Date.parse(val) > Date.now() - 8.64e7,
  },
};
</script>

<style scoped>
div {
  background: rgb(250, 234, 236);
}
.container {
  margin: 0 auto;
  min-height: 100vh;
  justify-content: center;
  align-items: center;
  text-align: center;
}
.title {
  font-family: "Raleway", sans-serif;
  font-size: 80px;
  color: rgb(190, 10, 115);
}

.done {
  color: rgb(26, 6, 73);
}
</style>
/* eslint-enable */
