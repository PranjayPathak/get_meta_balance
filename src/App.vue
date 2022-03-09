<template>
  <div class="login_form_container">
    <button class="login_button" v-on:click="login">Login with metamask</button>
  </div>
</template>

<script>
import Web3 from "web3/dist/web3.min.js";
export default {
  name: "App",
  components: {},
  methods: {
    async login() {
      if (typeof window.ethereum !== "undefined") {
        console.log("MetaMask is installed!");
        let web3 = new Web3(window.ethereum);
        try {
          // Request account access
          await window.ethereum.enable();
          let accounts = await web3.eth.getAccounts();
          console.log(accounts);

          let balance = await web3.eth.getBalance(accounts[0]);
          console.log(balance);
          return true;
        } catch (e) {
          // User denied access
          console.error(" User denied access");
          return false;
        }
      } else {
        console.log("Unable to detect MetaMask");
      }
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  margin-top: 60px;
}

* {
  box-sizing: border-box;
}

h1,
h2,
h3,
h4,
h5,
h6,
p,
body {
  margin: 0;
}
</style>
