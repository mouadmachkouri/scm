<template>
  <div>
    <h1>Connect</h1>
    <button @click="connectWallet">Connect Wallet</button>

    <div v-if="account">
      <p>Connected Account: {{ account }}</p>
      <p>Your Balance: {{ balance }} {{ nativeCurrency }}</p>
      <p>Current Network: {{ networkName }}</p>
      <p>Current {{ nativeCurrency }} Price: ${{ cryptoPrice.toFixed(2) }}</p>
      <button @click="payWithCrypto('0xf536638c27082D99f1df3CFC71966353e903F04A')">
        Pay {{ cryptoAmount.toFixed(6) }} {{ nativeCurrency }} (~$50)
      </button>
    </div>

  </div>
</template>

<script>
import { ref, onMounted, watch } from "vue";
import Web3 from "web3";

export default {
  setup() {
    const account = ref(null);
    const balance = ref(null);
    const networkName = ref("Unknown");
    const nativeCurrency = ref("ETH");
    const web3 = ref(null);
    const cryptoPrice = ref(0);
    const cryptoAmount = ref(0.1); // Default

    const networkMap = {
      "0x1": { name: "Ethereum Mainnet", currency: "ETH", coingeckoId: "ethereum" },
      "0x38": { name: "Binance Smart Chain", currency: "BNB", coingeckoId: "binancecoin" },
      "0x89": { name: "Polygon Mainnet", currency: "MATIC", coingeckoId: "matic-network" },
      "0xa86a": { name: "Avalanche Mainnet", currency: "AVAX", coingeckoId: "avalanche-2" },
      "0xa4b1": { name: "Arbitrum One", currency: "ETH", coingeckoId: "ethereum" },
      "0xa": { name: "Optimism", currency: "ETH", coingeckoId: "ethereum" },
    };

    const connectWallet = async () => {
      if (window.ethereum) {
        try {
          const accounts = await window.ethereum.request({ method: "eth_requestAccounts" });
          account.value = accounts[0];
          web3.value = new Web3(window.ethereum);
          await getNetwork();
          await getBalance();
        } catch (error) {
          console.error("User denied account access", error);
        }
      } else {
        alert("MetaMask is not installed. Please install it to use this app.");
      }
    };

    const getBalance = async () => {
      if (account.value && web3.value) {
        try {
          const balanceWei = await web3.value.eth.getBalance(account.value);
          balance.value = web3.value.utils.fromWei(balanceWei, "ether");
        } catch (error) {
          console.error("Error fetching balance", error);
        }
      }
    };

    const getNetwork = async () => {
      if (window.ethereum) {
        try {
          const chainId = await window.ethereum.request({ method: "eth_chainId" });

          if (networkMap[chainId]) {
            networkName.value = networkMap[chainId].name;
            nativeCurrency.value = networkMap[chainId].currency;
            await fetchCryptoPrice(networkMap[chainId].coingeckoId);
          } else {
            networkName.value = `Unknown Network (Chain ID: ${chainId})`;
            nativeCurrency.value = "Unknown";
            cryptoPrice.value = 0;
          }

          await getBalance();
        } catch (error) {
          console.error("Error fetching network", error);
        }
      }
    };
    const networkData = {
      "0x1": { name: "Ethereum", decimals: 18, symbol: "ETH" },
      "0x38": { name: "Binance Smart Chain", decimals: 18, symbol: "BNB" },
      "0x89": { name: "Polygon", decimals: 18, symbol: "MATIC" },
      "0xa86a": { name: "Avalanche", decimals: 18, symbol: "AVAX" },
    };

    const fetchCryptoPrice = async (coingeckoId) => {
      try {
        const response = await fetch(`https://api.coingecko.com/api/v3/simple/price?ids=${coingeckoId}&vs_currencies=usd`);
        const data = await response.json();
        if (data[coingeckoId]) {
          cryptoPrice.value = data[coingeckoId].usd;
          cryptoAmount.value = 50 / cryptoPrice.value; // Convert $50 to crypto amount
        }
      } catch (error) {
        console.error("Error fetching price", error);
      }
    };
    const isLoading = ref(false);
    const payWithCrypto = async (recipient) => {
      if (!web3.value) {
        alert("Please connect your wallet first.");
        return;
      }

      if (!cryptoPrice.value || cryptoPrice.value <= 0) {
        alert("Error fetching crypto price. Please try again.");
        return;
      }

      if (isLoading.value) return;
      isLoading.value = true;

      try {
        const amount = cryptoAmount.value;
        console.log(`Amount to send: ${amount} ${nativeCurrency.value}`);

        const chainId = await web3.value.eth.getChainId();
        const chainIdHex = '0x' + chainId.toString(16);
        console.log(`Chain ID: ${chainIdHex}`);
        const decimals = 18;
        const weiAmount = Math.round(amount * Math.pow(10, decimals));
        const weiHex = '0x' + weiAmount.toString(16);
        console.log(`Wei amount (hex): ${weiHex}`);

        const transactionParameters = {
          to: recipient,
          from: account.value,
          value: weiHex,
        };

        await window.ethereum.request({
          method: "eth_sendTransaction",
          params: [transactionParameters],
        });

        alert(`Payment of ${amount.toFixed(6)} ${nativeCurrency.value} successful!`);
        await getBalance();
      } catch (error) {
        console.error("Payment failed:", error);
        alert(`Payment failed: ${error.message || "Unknown error"}`);
      } finally {
        isLoading.value = false;
      }
    };



    onMounted(() => {
      if (window.ethereum) {
        window.ethereum.on("chainChanged", async () => {
          await getNetwork();
          await getBalance();
        });

        window.ethereum.on("accountsChanged", async (accounts) => {
          if (accounts.length === 0) {
            account.value = null;
            balance.value = null;
            networkName.value = null;
          } else {
            account.value = accounts[0];
            await getBalance();
          }
        });
      }
    });

    watch(nativeCurrency, async () => {
      if (networkMap[nativeCurrency.value]) {
        await fetchCryptoPrice(networkMap[nativeCurrency.value].coingeckoId);
      }
    });

    return {
      account,
      balance,
      networkName,
      nativeCurrency,
      cryptoPrice,
      cryptoAmount,
      connectWallet,
      payWithCrypto,
    };
  },
};
</script>

<style scoped>
button {
  padding: 10px 20px;
  font-size: 16px;
}
</style>
