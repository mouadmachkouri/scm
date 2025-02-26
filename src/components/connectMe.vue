<template>
  <div>
    <h1>Connect to MetaMask</h1>
    <button @click="connectWallet">Connect Wallet</button>
    <p v-if="account">Connected Account: {{ account }}</p>
    <button @click="payWithBNB(0.1, '0xf536638c27082D99f1df3CFC71966353e903F04A')">
      Pay 0.1 BNB
    </button>
  </div>
</template>

<script>
import Web3 from "web3";

export default {
  data() {
    return {
      account: null,
      web3: null,
    };
  },
  methods: {
    async connectWallet() {
      if (window.ethereum) {
        try {
          // Request account access
          const accounts = await window.ethereum.request({
            method: "eth_requestAccounts",
          });
          this.account = accounts[0];

          // Initialize web3
          this.web3 = new Web3(window.ethereum);
        } catch (error) {
          console.error("User  denied account access", error);
        }
      } else {
        alert("MetaMask is not installed. Please install it to use this app.");
      }
    },
    async payWithBNB(amount, recipient) {
      if (!this.web3) {
        alert("Please connect your wallet first.");
        return;
      }

      const valueInWei = this.web3.utils.toWei(amount.toString(), "ether"); // Convert BNB to Wei

      try {
        const transactionParameters = {
          to: recipient, // The recipient's address
          from: this.account, // The user's account
          value: valueInWei, // Amount in Wei
        };

        // Send the transaction
        await window.ethereum.request({
          method: "eth_sendTransaction",
          params: [transactionParameters],
        });
        alert("Payment successful!");
      } catch (error) {
        console.error("Payment failed", error);
      }
    },
  },
};
</script>

<style scoped>
button {
  padding: 10px 20px;
  font-size: 16px;
}
</style>
