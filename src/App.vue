<template>
  <div class="container">
    <div v-if="!isLoggedIn">
      <button class="button" v-on:click="loginToMeta">
        Login with Metamask
      </button>
    </div>
    <div v-else>
      <div class="balance_dashboard">
        <button v-if="balance == null" class="button" v-on:click="getBalance">
          Get Balance
        </button>
        <div v-else>
          <hr />
          <h2>Chain ID: {{ chainId }}</h2>
          <hr />
          <h2>Balance: {{ balance }} ETH</h2>
          <hr />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Web3 from "web3/dist/web3.min.js";
export default {
  name: "App",
  data() {
    return {
      isLoggedIn: window.localStorage.getItem("isLoggedIn")
        ? window.localStorage.getItem("isLoggedIn")
        : false,
      balance: null,
      chainId: null,
    };
  },
  methods: {
    async loginToMeta() {
      if (typeof window.ethereum !== "undefined") {
        // MetaMask is installed
        try {
          // Request account access
          await window.ethereum.request({ method: "eth_requestAccounts" });

          //update login state and logins status in local storage
          this.isLoggedIn = true;
          if (window.localStorage) {
            window.localStorage.setItem("isLoggedIn", true);
          }
        } catch (err) {
          // User denied access
          console.error("Error while connecting to Metamask");
        }
      } else {
        alert("Unable to detect MetaMask");
      }
    },
    async getBalance() {
      let web3 = new Web3(window.ethereum);
      let accounts = await web3.eth.getAccounts();
      let chainId = await web3.eth.getChainId();
      let balance = await web3.eth.getBalance(accounts[0]);
      this.chainId = chainId;
      this.balance = await web3.utils.fromWei(balance, "ether"); //from Wei to ETH
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

.button {
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

.container {
  padding: 5rem;
  margin-top: 10rem;
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
