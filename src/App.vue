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
              <label for="sendAddress">Send To : </label>
              <input
                id="sendAddress"
                type="text"
                v-model="recieverAddress"
              /><br />
              <label for="transactionAmount">Amount (in wei) : </label>
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
                  <a target="_blank" v-bind:href="etherscanUrl"
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
          <h3>Get Token Balance</h3>

          <select v-model="selectedToken">
            <option disabled value="TOKEN">Please Select</option>
            <option>HEX</option>
            <option>SHIB</option>
            <option>MATIC</option>
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

        <hr />
        <hr />

        <div class="container">
          <h3>Send Token Transaction</h3>
          <form v-on:submit.prevent="sendTokenTransaction">
            <label for="tokenAddress">Token Address (default DERC) : </label>
            <input id="tokenAddress" type="text" v-model="tokenAddress" />
            <br />
            <label for="recieverAddress"
              >Reciever Address (default-myDemo Account) :
            </label>
            <input
              id="recieverAddress"
              type="text"
              v-model="tokenRecieverAddress"
            />
            <br />
            <label for="transactionAmount">Value : </label>
            <input
              id="transactionAmount"
              min="0"
              type="number"
              v-model="tokenValue"
            />
            <button class="button">Send</button>
          </form>
          <div>
            <p>Chain ID: {{ transactionChainId }}</p>
            <p>NAME: {{ name }}</p>
            <p>SYMBOL: {{ symbol }}</p>
            <p>BALANCE: {{ tokenTransactionBalance }}</p>
            <p>DECIMALS: {{ decimals }}</p>
          </div>
          <div v-if="tokenTransactionDone">
            <h4>Transaction {{ transactionResponseMessage }}</h4>
            <a target="_blank" v-bind:href="etherscanUrl"
              >Checkout on Etherscan -></a
            >
          </div>
        </div>
        <hr />
        <hr />
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
      web3: null,
      selectedAccount: null,
      balance: null,
      isBalance: false,
      chainId: null,
      // for Token Balance
      selectedToken: null,
      tokenBalance: null,
      istokenBalance: false,
      //For Transaction
      recieverAddress: "0x30E0DEa1ABFf57bfA81ccB2E57A3dEA865489363",
      transactionAmount: 2,
      transactionHash: null,
      transactionDone: false,
      //For Token Transaction
      tokenAddress: "0xfe4F5145f6e09952a5ba9e956ED0C25e3Fa4c7F1",
      tokenRecieverAddress: "0x30E0DEa1ABFf57bfA81ccB2E57A3dEA865489363",
      tokenValue: 1,
      transactionResponseMessage: "",
      tokenTransactionHash: null,
      tokenTransactionDone: false,
      transactionChainId: null,
      tokenTransactionBalance: null,
      decimals: null,
      name: "",
      symbol: "",
    };
  },
  computed: {
    etherscanUrl() {
      return "https://goerli.etherscan.io/tx/" + this.transactionHash;
    },
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
      let chainId = await this.web3.eth.getChainId();
      let balance = await this.web3.eth.getBalance(this.selectedAccount);
      this.chainId = chainId;
      this.balance = await this.web3.utils.fromWei(balance, "ether"); //from Wei to ETH
      this.isBalance = true;
    },
    // Fetch token balance
    async getTokenBalance() {
      const tokenAddress = {
        HEX: "0x7D1AfA7B718fb893dB30A3aBc0Cfc608AaCfeBB0",
        SHIB: "0x95aD61b0a150d79219dCF64E1E6Cc01f0B64C4cE",
        MATIC: "0x7D1AfA7B718fb893dB30A3aBc0Cfc608AaCfeBB0",
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

      const tokenInstance = new this.web3.eth.Contract(
        tokenABI,
        tokenAddress[this.selectedToken]
      );

      try {
        let balance = await tokenInstance.methods
          .balanceOf(this.selectedAccount)
          .call();
        this.tokenBalance = balance;
      } catch {
        this.tokenBalance = 0;
        console.log("Unable to fetch balance of selected token");
      }
      this.istokenBalance = true;
    },
    async sendTokenTransaction() {
      let chainID = await this.web3.eth.getChainId();
      if (chainID !== 80001 && chainID !== 5) {
        alert("Please shift to Goerli or Mumbai Network ");
        return; //End method
      }
      const transferABI = [
        {
          constant: false,
          inputs: [
            {
              name: "_to",
              type: "address",
            },
            {
              name: "_value",
              type: "uint256",
            },
          ],
          name: "transfer",
          outputs: [
            {
              name: "",
              type: "bool",
            },
          ],
          type: "function",
        },
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
          stateMutability: "view",
          type: "function",
        },
        {
          constant: true,
          inputs: [],
          name: "name",
          outputs: [
            {
              name: "",
              type: "string",
            },
          ],
          payable: false,
          stateMutability: "view",
          type: "function",
        },
        {
          constant: true,
          inputs: [],
          name: "decimals",
          outputs: [
            {
              name: "",
              type: "uint8",
            },
          ],
          payable: false,
          stateMutability: "view",
          type: "function",
        },
        {
          constant: true,
          inputs: [],
          name: "symbol",
          outputs: [
            {
              name: "",
              type: "string",
            },
          ],
          payable: false,
          stateMutability: "view",
          type: "function",
        },
      ];

      let contract = new this.web3.eth.Contract(transferABI, this.tokenAddress);
      //Update Chain ID
      this.transactionChainId = chainID;
      try {
        //Update Token Info
        [this.tokenTransactionBalance, this.decimals, this.name, this.symbol] =
          await Promise.all([
            contract.methods.balanceOf(this.selectedAccount).call(),
            contract.methods.decimals().call(),
            contract.methods.name().call(),
            contract.methods.symbol().call(),
          ]);
      } catch (err) {
        console.log("Unable to fetch balance of selected token: ", err);
      }

      //Transaction
      await contract.methods
        .transfer(this.tokenRecieverAddress, this.tokenValue)
        .send({ from: this.selectedAccount })
        .on("transactionHash", (hash) => {
          this.tokenTransactionHash = hash;
        })
        .on("receipt", () => {
          alert("Transaction Successful!");

          this.transactionResponseMessage = "Successful!!!";
          this.tokenTransactionDone = true;
        })
        .on("error", () => {
          console.log("Unable to perform transaction ");

          this.transactionResponseMessage = "Failed!!!";
          this.tokenTransactionHash = "";
          this.tokenTransactionDone = true;
        });
    },
    //Etherium Transaction
    async sendTransaction() {
      let chainID = await this.web3.eth.getChainId();
      if (chainID !== 5) {
        alert("Please shift to Goerli Network");
        return; //End method
      }
      this.transactionDone = false;
      await this.web3.eth
        .sendTransaction({
          from: this.selectedAccount,
          to: this.recieverAddress,
          value: this.transactionAmount,
          // common: { baseChain: "goerli" },
        })
        .on("transactionHash", (hash) => {
          this.transactionHash = hash;
        })
        .on("receipt", () => {
          alert("Transaction Successful!");
          this.transactionDone = true;
        })
        .on("error", () => console.log("Unable to perform transaction: "));
    },
  },
  async mounted() {
    //Initialize web3 instance and selected account
    this.web3 = new Web3(window.ethereum);
    const accounts = await this.web3.eth.getAccounts();
    this.selectedAccount = accounts[0];
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  margin-top: 10px;
}

* {
  box-sizing: border-box;
}

body {
  background-color: rgb(185, 194, 248);
}
.container {
  display: inline-block;
  margin: 2rem 3rem;
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
label {
  font-size: 18px;
  font-weight: 600;
}
input,
select {
  padding: 0.5rem 1rem;
  margin: 1rem 0rem;
  border-radius: 4px;
}
</style>
