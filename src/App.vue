<template>
  <div class="login_form_container">
    <button
      v-if="login === false"
      class="login_button"
      v-on:click="loginToMeta"
    >
      Login with Metamask
    </button>
    <h2 v-else>Your balance is {{ balance }} ETH</h2>
  </div>
</template>

<script>
import Web3 from "web3/dist/web3.min.js";
export default {
  name: "App",
  data() {
    return {
      login: false,
      balance: "...",
    };
  },
  components: {},
  methods: {
    async loginToMeta() {
      if (typeof window.ethereum !== "undefined") {
        // MetaMask is installed
        let web3 = new Web3(window.ethereum);
        try {
          // Request account access
          await window.ethereum.request({ method: "eth_requestAccounts" });

          let accounts = await web3.eth.getAccounts();
          let balance = await web3.eth.getBalance(accounts[0]);
          this.balance = balance;
          this.login = true;
        } catch (err) {
          // User denied access
          console.info(" User denied access");
        }
      } else {
        alert("Unable to detect MetaMask");
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

body {
  background-color: rgb(185, 194, 248);
}

.login_button {
  padding: 1rem;
  border-radius: 5px;
  background-color: blueviolet;
  color: aliceblue;
  font-weight: 600;
  font-size: 1rem;
}
.login_button:hover {
  background-color: darkorchid;
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
