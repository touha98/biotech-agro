<template>
  <v-app>
    <v-app-bar v-if="!hideBarFor.includes($route.name) && !printMode" dark app>
      <v-app-bar-nav-icon
        v-if="userState"
        @click.stop="drawer = !drawer"
      ></v-app-bar-nav-icon>
      <v-toolbar-title class="headline text-uppercase">
        <span>biotech &nbsp;</span>
        <span class="font-weight-light">agrovat</span>
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <v-toolbar-items>
        <v-btn dark @click="printScr" icon>
          <v-icon>mdi-printer</v-icon>
        </v-btn>
      </v-toolbar-items>
    </v-app-bar>

    <v-navigation-drawer
      v-if="!hideNavFor.includes($route.name) && !printMode"
      app
      class="secondary base"
      v-model="drawer"
    >
      <template v-if="userState != null">
        <v-list-item v-if="userState.name" to="/profile">
          <v-list-item-avatar>
            <v-img v-if="userState.image" :src="userState.image"></v-img>
            <v-avatar color="primary" v-else>
              <v-icon v-if="userState.name" dark text>
                {{ userState.name.slice(0, 1) }}
              </v-icon>
            </v-avatar>
          </v-list-item-avatar>
          <v-list-item-content>
            <v-list-item-title>{{ userState.name }}</v-list-item-title>
            <v-list-item-subtitle>{{
              userState.designation
            }}</v-list-item-subtitle>
          </v-list-item-content>
        </v-list-item>

        <v-divider></v-divider>

        <v-list v-if="userState.name" dense>
          <v-list-item
            :to="item.route"
            v-for="item in links"
            :key="item.title"
            link
          >
            <v-list-item-icon>
              <v-icon>{{ item.icon }}</v-icon>
            </v-list-item-icon>

            <v-list-item-content>
              <v-list-item-title>{{ item.title }}</v-list-item-title>
            </v-list-item-content>
          </v-list-item>
          <v-divider></v-divider>
        </v-list>
      </template>
      <v-list-item v-if="loggedIn" link>
        <v-list-item-icon>
          <v-icon>mdi-logout</v-icon>
        </v-list-item-icon>

        <v-list-item-content @click="signout">
          <v-list-item-title>Sign Out</v-list-item-title>
        </v-list-item-content>
      </v-list-item>
      <v-footer dark class="primary" absolute
        >{{ app.name }} {{ app.version }}</v-footer
      >
    </v-navigation-drawer>

    <v-content>
      <v-container v-if="printMode">
        <v-row>
          <v-list two-line class="mx-auto transparent" max-width="360px">
            <v-list-item>
              <v-list-item-avatar class="ma-0 pa-0" size="60">
                <img
                  ref="invLogo"
                  class="float-left"
                  src="data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz4NCjwhLS0gR2VuZXJhdG9yOiBBZG9iZSBJbGx1c3RyYXRvciAyMi4wLjAsIFNWRyBFeHBvcnQgUGx1Zy1JbiAuIFNWRyBWZXJzaW9uOiA2LjAwIEJ1aWxkIDApICAtLT4NCjxzdmcgdmVyc2lvbj0iMS4xIiBpZD0iTGF5ZXJfMSIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxuczp4bGluaz0iaHR0cDovL3d3dy53My5vcmcvMTk5OS94bGluayIgeD0iMHB4IiB5PSIwcHgiDQoJIHZpZXdCb3g9IjAgMCA3MjAgNzIwIiBzdHlsZT0iZW5hYmxlLWJhY2tncm91bmQ6bmV3IDAgMCA3MjAgNzIwOyIgeG1sOnNwYWNlPSJwcmVzZXJ2ZSI+DQo8c3R5bGUgdHlwZT0idGV4dC9jc3MiPg0KCS5zdDB7ZmlsbDpub25lO3N0cm9rZTojMDAwMDAwO3N0cm9rZS1taXRlcmxpbWl0OjEwO30NCjwvc3R5bGU+DQo8Zz4NCgk8ZWxsaXBzZSBjbGFzcz0ic3QwIiBjeD0iMzY0IiBjeT0iMzYzIiByeD0iMjM2LjUiIHJ5PSIxMzQuNSIvPg0KCTxlbGxpcHNlIGNsYXNzPSJzdDAiIGN4PSIzNjQiIGN5PSIzNjMiIHJ4PSIyMzYuNSIgcnk9IjEzNC41Ii8+DQoJPGc+DQoJCTxwYXRoIGQ9Ik01MDMuOCwyMzMuMWw0MC4xLTY0LjNMNTI2LDE1Ny42bC00Mi43LDY4LjNjLTYuNC0yLTEzLTMuOS0xOS44LTUuNmw0NS4zLTcyLjVMNDkxLDEzNi42bC00OS4zLDc4LjkNCgkJCWMtNi44LTEuMy0xMy43LTIuNC0yMC44LTMuM2w1NC04Ni40TDQ1NywxMTQuNmwtNTkuNCw5NS4xYy0xMC43LTAuOC0yMS42LTEuMi0zMi42LTEuMmMtMTQxLjEsMC0yNTUuNSw2OC43LTI1NS41LDE1My41DQoJCQljMCw1NC4zLDQ3LDEwMi4xLDExNy44LDEyOS4zTDE4Niw1NTcuNWwxNy45LDExLjJsNDMuOS03MC4yYzYuMywyLDEyLjgsMy44LDE5LjUsNS40TDIxOS43LDU4MGwxNy45LDExLjJsNTEuNi04Mi41DQoJCQljNi45LDEuMywxNCwyLjQsMjEuMiwzLjNsLTU1LjIsODguNGwxNy45LDExLjJsNjAuNy05Ny4yYzEwLjMsMC44LDIwLjcsMS4xLDMxLjMsMS4xYzE0MS4xLDAsMjU1LjUtNjguNywyNTUuNS0xNTMuNQ0KCQkJQzYyMC41LDMwOCw1NzQsMjYwLjQsNTAzLjgsMjMzLjF6IE0zNjQsNDk3LjVjLTEzMC42LDAtMjM2LjUtNjAuMi0yMzYuNS0xMzQuNVMyMzMuNCwyMjguNSwzNjQsMjI4LjVTNjAwLjUsMjg4LjcsNjAwLjUsMzYzDQoJCQlTNDk0LjYsNDk3LjUsMzY0LDQ5Ny41eiIvPg0KCTwvZz4NCgk8Zz4NCgkJPHBhdGggZD0iTTE3Nyw0MDYuM2wyOC4xLTEwMC42aDU2YzEyLjYsMCwyMi4xLDEuNSwyOC40LDQuNmM3LjksMy43LDEwLjgsOS43LDguNSwxNy43Yy0xLjgsNi42LTcsMTIuMS0xNS40LDE2LjQNCgkJCWMtNy40LDMuOC0xNi40LDYuMy0yNi44LDcuMmwtMC40LDEuM2MxMC4yLDAuNywxOC4yLDIuNiwyMy44LDUuNmM3LjksNC4zLDEwLjcsMTAuNiw4LjQsMTguOGMtMi43LDkuNi0xMC4yLDE2LjktMjIuNywyMi4yDQoJCQljLTEwLjgsNC41LTIzLjQsNi44LTM3LjksNi44SDE3N3ogTTIzNS4yLDMxNC40bC05LjcsMzQuN2g2LjdjNCwwLDcuNi0wLjIsMTAuNS0wLjdjMy0wLjUsNS45LTEuNSw4LjctMi45DQoJCQljMy43LTEuOSw2LjctNC4xLDguOS02LjhjMi4zLTIuNiwzLjgtNS41LDQuNy04LjVjMS41LTUuMiwwLjUtOS4yLTIuNy0xMi4xYy0zLjMtMi44LTguNi00LjMtMTUuOS00LjMNCgkJCUMyNDMsMzEzLjgsMjM5LjIsMzE0LDIzNS4yLDMxNC40eiBNMjIzLjEsMzU3LjdsLTExLjIsNDAuMWgxMC44YzYuNSwwLDEyLjktMS43LDE5LjEtNS4yYzYuMy0zLjUsMTAuMi04LjIsMTEuOS0xNC4xDQoJCQljMi4xLTcuNywwLjUtMTMuMi01LTE2LjRDMjQ0LDM1OS4xLDIzNS40LDM1Ny43LDIyMy4xLDM1Ny43eiIvPg0KCQk8cGF0aCBkPSJNMzExLjksNDA3bDI4LjMtMTAxLjRjMi41LDAsNS42LDAuMSw5LjQsMC40YzMuOCwwLjIsNiwwLjQsNi42LDAuNGMyLjUsMCw0LjUsMCw2LjEtMC4xYzYuMy0wLjQsOS44LTAuNiwxMC40LTAuNg0KCQkJTDM0NC41LDQwN2MtMi41LDAtNC41LTAuMS02LTAuMWMtNi4xLTAuNC05LjUtMC42LTEwLTAuNmMtMi41LDAtNS43LDAuMS05LjYsMC40QzMxNC44LDQwNi45LDMxMi41LDQwNywzMTEuOSw0MDd6Ii8+DQoJCTxwYXRoIGQ9Ik01NTIuMiwzNTUuNWMtNC4yLDE1LTE1LjMsMjcuNS0zMy4yLDM3LjVjLTE4LjksMTAuNS00MS4yLDE1LjgtNjcuMSwxNS44Yy0yNC40LDAtNDIuNi00LjYtNTQuNi0xMy45DQoJCQljLTEyLTkuMi0xNS42LTIyLjItMTEtMzguOGM0LjMtMTUuMywxNS4yLTI3LjgsMzIuOS0zNy42YzE4LjUtMTAuMyw0MS0xNS41LDY3LjUtMTUuNWMyOC4xLDAsNDcuNSw1LjksNTguNCwxNy43DQoJCQlDNTUzLjYsMzMwLjIsNTU2LDM0MS44LDU1Mi4yLDM1NS41eiBNNDgyLjgsMzExLjljLTE1LjUsMC0yOC40LDMuOS0zOC43LDExLjhjLTEwLjgsOC4zLTE4LjUsMjAuNS0yMy4xLDM2LjcNCgkJCWMtMywxMC44LTIuMSwxOS44LDIuOSwyN2M1LjgsOC40LDE2LjIsMTIuNSwzMS40LDEyLjVjMTUuOCwwLDI5LjMtNC44LDQwLjctMTQuM2MxMC4yLTguNywxNy4yLTE5LjYsMjAuOS0zMi45DQoJCQljMy0xMC45LDIuMS0yMC4xLTIuNy0yNy42QzUwOC40LDMxNi4zLDQ5OCwzMTEuOSw0ODIuOCwzMTEuOXoiLz4NCgk8L2c+DQo8L2c+DQo8Zz4NCjwvZz4NCjxnPg0KPC9nPg0KPGc+DQo8L2c+DQo8Zz4NCjwvZz4NCjxnPg0KPC9nPg0KPGc+DQo8L2c+DQo8Zz4NCjwvZz4NCjxnPg0KPC9nPg0KPGc+DQo8L2c+DQo8Zz4NCjwvZz4NCjxnPg0KPC9nPg0KPGc+DQo8L2c+DQo8Zz4NCjwvZz4NCjxnPg0KPC9nPg0KPGc+DQo8L2c+DQo8L3N2Zz4NCg=="
                />
              </v-list-item-avatar>
              <v-list-item-content>
                <v-list-item-title class="display-1"
                  >Biotech Agrovat</v-list-item-title
                >
                <v-list-item-subtitle class="black--text"
                  >10 Arambag, Dhaka-1000</v-list-item-subtitle
                >
              </v-list-item-content>
            </v-list-item>
          </v-list>
        </v-row>
      </v-container>
      <router-view />
    </v-content>
  </v-app>
</template>

<script>
import firebase from "./firebaseConfig";
const { version, name } = require("../package.json");
import { mapGetters, mapActions } from "vuex";
export default {
  name: "App",
  data: () => ({
    drawer: null,
    links: [
      { icon: "mdi-home", title: "home", route: "/" },
      { icon: "mdi-spray-bottle", title: "Products", route: "/products" },
      {
        icon: "mdi-sign-real-estate",
        title: "product statement",
        route: "/statement"
      },
      { icon: "mdi-warehouse", title: "Stock", route: "/stock" },
      { icon: "mdi-playlist-check", title: "Challan", route: "/challan" },
      { icon: "mdi-store", title: "Dealers", route: "/dealers" },
      {
        icon: "mdi-cash-multiple",
        title: "Transactions",
        route: "/transactions"
      },
      { icon: "mdi-playlist-plus", title: "Orders", route: "/orders" },
      {
        icon: "mdi-format-list-bulleted",
        title: "Invoices",
        route: "/invoices"
      },
      { icon: "mdi-information", title: "About", route: "/about" }
    ],
    hideNavFor: ["inv-print", "login"],
    hideBarFor: ["inv-print", "login"]
  }),
  computed: {
    ...mapGetters(["userState", "printMode"]),
    loggedIn() {
      return firebase.auth().currentUser;
    },

    app() {
      return { version, name };
    }
  },
  mounted() {},
  methods: {
    ...mapActions(["printScr"]),
    signout() {
      firebase
        .auth()
        .signOut()
        .then(
          () => {
            this.$store.state.user = null;
            this.$store.commit("emptyProduct");
            this.$router.replace("/login");
          },
          error => alert(error)
        );
    }
  }
};
</script>
