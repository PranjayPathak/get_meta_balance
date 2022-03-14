<template>
  <div class="container">
    <div v-if="!isLoggedIn">
      <button class="button" v-on:click="loginToMeta">
        Login with Metamask
      </button>
    </div>
    <div v-else>
      <div class="balance_dashboard">
        <div class="container">
          <button class="button" v-on:click="getBalance">
            Get ETH Balance
          </button>
          <div v-if="isBalance">
            <h4>Chain ID: {{ chainId }}</h4>
            <h4>Balance: {{ balance }} ETH</h4>
          </div>
        </div>
        <hr />
        <hr />
        <div class="container">
          <h3>Send Transaction</h3>

          <div>
            <form v-on:submit.prevent="sendTransaction">
              <label for="sendAddress">Send To:</label>
              <input id="sendAddress" type="text" v-model="recieverAddress" />
              <label for="transactionAmount">Amount (in wei):</label>
              <input
                id="transactionAmount"
                min="1"
                type="number"
                v-model="transactionAmount"
              />
              <button class="button">Send</button>
              <div>
                <div v-if="transactionDone">
                  <h4>Transaction Successful!!</h4>
                  <a
                    target="_blank"
                    v-bind:href="
                      'https://goerli.etherscan.io/tx/' + transactionHash
                    "
                    >Checkout on Etherscan -></a
                  >
                </div>
              </div>
            </form>
          </div>
        </div>
        <hr />
        <hr />
        <div class="container">
          <h3>Get TOKEN Balance</h3>

          <select v-model="selectedToken">
            <option disabled value="TOKEN">Please Select</option>
            <option>HEX</option>
            <option>SHIB</option>
          </select>

          <br />
          <br />
          <button
            v-bind:disabled="!selectedToken"
            v-on:click="getTokenBalance"
            class="button"
          >
            Get Balance
          </button>
          <br />
          <p v-if="istokenBalance">
            Balance of {{ selectedToken }} is {{ tokenBalance }}
          </p>
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
      isBalance: false,
      chainId: null,
      //for Token Balance
      selectedToken: null,
      tokenBalance: null,
      istokenBalance: false,
      //For Transaction
      recieverAddress: "0x30E0DEa1ABFf57bfA81ccB2E57A3dEA865489363",
      transactionAmount: 2,
      transactionHash: null,
      transactionDone: false,
    };
  },
  methods: {
    //Login method
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
    // Etherium balance
    async getBalance() {
      const web3 = new Web3(window.ethereum);
      const accounts = await web3.eth.getAccounts();
      const selectedAccount = accounts[0];
      let chainId = await web3.eth.getChainId();
      let balance = await web3.eth.getBalance(selectedAccount);
      this.chainId = chainId;
      this.balance = await web3.utils.fromWei(balance, "ether"); //from Wei to ETH
      this.isBalance = true;
    },
    //Fetch token balance
    async getTokenBalance() {
      const tokenAddress = {
        HEX: "0x7D1AfA7B718fb893dB30A3aBc0Cfc608AaCfeBB0",
        SHIB: "0x95aD61b0a150d79219dCF64E1E6Cc01f0B64C4cE",
      };
      const tokenABI = [
        {
          constant: true,
          inputs: [
            {
              name: "_owner",
              type: "address",
            },
          ],
          name: "balanceOf",
          outputs: [
            {
              name: "balance",
              type: "uint256",
            },
          ],
          payable: false,
          type: "function",
        },
      ];

      const web3 = new Web3(window.ethereum);
      const accounts = await web3.eth.getAccounts();
      const selectedAccount = accounts[0];
      const tokenInstance = new web3.eth.Contract(
        tokenABI,
        tokenAddress[this.selectedToken]
      );
      await tokenInstance.methods
        .balanceOf(selectedAccount)
        .call()
        .then((response) => {
          this.tokenBalance = response;
          this.istokenBalance = true;
        })
        .catch(() => {
          this.tokenBalance = 0;
          this.istokenBalance = true;
          console.log("Unable to fetch balance of selected token");
        });
    },

    //Etherium Transaction
    async sendTransaction() {
      const web3 = new Web3(window.ethereum);
      const accounts = await web3.eth.getAccounts();
      const selectedAccount = accounts[0];
      this.transactionDone = false;
      await web3.eth
        .sendTransaction({
          from: selectedAccount,
          to: this.recieverAddress,
          value: this.transactionAmount,
        })
        .on("transactionHash", (hash) => {
          this.transactionHash = hash;
        })
        .on("receipt", () => {
          alert("Transaction Successful!");
          this.transactionDone = true;
        })
        .on("error", (err) =>
          console.log("Unable to perform transaction: ", err)
        );
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
  margin-top: 10px;
}

* {
  box-sizing: border-box;
}

body {
  background-color: rgb(185, 194, 248);
}
.container {
  margin: 2rem 0;
}
.button {
  padding: 0.5rem 1rem;
  border-radius: 5px;
  background-color: blueviolet;
  color: aliceblue;
  font-weight: 600;
  font-size: 1rem;
}
.login_button:hover {
  background-color: darkorchid;
}

input,
select {
  padding: 0.5rem 1rem;
  margin: 1rem;
  border-radius: 4px;
}
</style>
