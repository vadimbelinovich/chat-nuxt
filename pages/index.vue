<template>
  <v-layout column justify-center align-center>
    <v-flex xs12 sm8>
      <v-card min-width="290">
        <v-snackbar v-model="snackbar" :timeout="3000" top>
          {{ message }}
          <v-btn dark text @click="snackbar = false">Close</v-btn>
        </v-snackbar>
        <v-card-title>
          <h1>Nuxt chat</h1>
        </v-card-title>
        <div id="app">
          <v-app id="inspire">
            <v-form ref="form" v-model="valid" lazy-validation>
              <v-text-field
                v-model="name"
                :counter="16"
                :rules="nameRules"
                label="Ваше имя"
                required
              ></v-text-field>

              <v-text-field v-model="room" :rules="roomRules" label="Введите комнату" required></v-text-field>

              <v-btn :disabled="!valid" color="primary" class="mr-4" @click="submit">Войти</v-btn>
            </v-form>
          </v-app>
        </div>
      </v-card>
    </v-flex>
  </v-layout>
</template>

<script>
import { mapMutations } from "vuex";
export default {
  layout: "empty",
  head: {
    title: "Welcome to Nuxt!"
  },
  data: () => ({
    valid: true,
    name: "",
    snackbar: false,
    message: "",
    nameRules: [
      v => !!v || "Введите имя",
      v => (v && v.length <= 16) || "Имя должно быть не длиннее 16 символов"
    ],
    room: "",
    roomRules: [v => !!v || "Введите комнату"]
  }),
  sockets: {
    connect() {
      console.log("Client IO");
    }
  },
  mounted() {
    const { message } = this.$route.query;
    if (message === "noUser") this.message = "Введите данные";
    else if (message === "leftChat") this.message = "Вы вышли из чата";
    this.snackbar = !!this.message;
  },
  methods: {
    ...mapMutations(["setUser"]),
    submit() {
      this.$refs.form.validate();
      const user = {
        name: this.name,
        room: this.room
      };
      this.$socket.emit("userJoined", user, data => {
        if (typeof data === "string") {
          console.error(data);
        } else {
          user.id = data.userId;
          this.setUser(user);
          this.$router.push("/chat");
        }
      });
    }
  }
};
</script>