<template>
  <Drive v-show="!admin" />
  <Backstage v-show="admin" admin="admin" />
</template>

<script>
import Backstage from "./components/Backstage.vue";
import Drive from "./components/Drive.vue";
export default {
  name: "App",
  components: {
    Backstage,
    Drive,
  },
  data() {
    return {
      admin: null,
    };
  },
  beforeMount() {
    fetch("/api/admin/", {
      method: "GET",
      headers: { "Content-Type": "application/json" },
    })
      .then((req) => {
        return req.json();
      })
      .then((text) => {
        this.admin = text.admin;
      });
  },
};
</script>

<style>
#app {
  font-family: Monaco, Avenir, Helvetica, Arial, sans-serif;
}
html,
body {
  background-color: #181a1b;
  color: white;
}
</style>
