<script setup lang="ts">
import {computed, onMounted, ref} from 'vue'
import {BrowserProvider, ethers} from "ethers";
import NavbarComponent from "@/components/NavbarComponent.vue";
import {STAKING_ABI, STAKING_CONTRACT_ADDRESS} from "@/constants";

const currentAccount = ref();
const stakedNfts = ref<number[]>([]);
const isLoading = ref(true);

let stakingContract: ethers.Contract;

onMounted(async () => {
  try {
    const ethereum = await checkEthereum();

    const accounts = await ethereum.request({method: "eth_accounts"});

    if (accounts.length !== 0) {
      currentAccount.value = accounts[0];

      await initContract()

      stakedNfts.value = await fetchStakedNfts()
    } else {
      console.log("No authorized account found");
    }

    ethereum.on("accountsChanged", (accounts: string | any[]) => {
      if (accounts.length === 0) {
        console.log("Please connect to MetaMask.");
      } else if (accounts[0] !== currentAccount.value) {
        currentAccount.value = accounts[0];
        console.log("Account changed");
      }
    });
  } catch (error) {
    console.log(error);
  } finally {
    isLoading.value = false;
  }
})

// Check if Ethereum is available
const checkEthereum = async () => {
  const { ethereum } = window;

  if (!ethereum) {
    alert("Get MetaMask!");
    window.open("https://metamask.io/download.html", "_blank");
    throw new Error("Ethereum not found");
  } else {
    console.log("We have the ethereum object", ethereum);
  }

  return ethereum;
}

const connectWallet = async () => {
  try {
    const ethereum = await checkEthereum();

    const accounts = await ethereum.request({method: "eth_requestAccounts"});

    currentAccount.value = accounts[0];

    stakedNfts.value = await fetchStakedNfts()

  } catch (error) {
    console.log(error);
  }
}

const buttonLabel = computed(() => {
  return currentAccount.value ? 'Unstake' : 'Connect Wallet'
})

const buttonClick = () => {
  if (currentAccount.value) {
    unstake()
  } else {
    connectWallet()
  }
}

const initContract = async () => {
  const ethereum = await checkEthereum();

  const provider = new BrowserProvider(ethereum);
  const signer = await provider.getSigner();

  stakingContract = new ethers.Contract(STAKING_CONTRACT_ADDRESS, STAKING_ABI, signer)
}

const fetchStakedNfts = async () => {
  if (stakingContract) {
    const stakedNfts = await stakingContract.tokensStakedByOwner(currentAccount.value);

    console.log("stakedNfts", stakedNfts);
    return stakedNfts;
  }

  return [];
}

const unstake = async () => {
  try {
    await stakingContract?.unstake(stakedNfts.value)
    window.location.reload()
  } catch (error: any) {
    alert(error?.revert.args[0] || error.message || 'Something went wrong')
  }
}

</script>

<template>
  <NavbarComponent :is-connected="!!currentAccount" :action="connectWallet"/>


  <main>
    <h1>Unstake your NFTs</h1>
    <p>Unstake your NFTs to get your original NFT back</p>


    <p v-if="stakedNfts.length">You staked the following NFTs:</p>
    <p v-else>No staked Nft with this address</p>
    <span v-for="nft in stakedNfts" :key="nft">{{ `${nft} ` }}</span>

    <button v-if="!isLoading && stakedNfts.length" @click="buttonClick">{{ buttonLabel }}</button>
  </main>

  <img alt="meka ape monkey" class="logo" src="./assets/meka_ape_monkey.png"/>
</template>

<style scoped>
.logo {
  height: 80vh;
  position: absolute;
  bottom: 0;
  z-index: 0;
}

main {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-top: 25px;
}

h1 {
  font-size: 3rem;
  font-weight: 900;
  margin: 0;
}

span {
  display: contents;
}

button {
  margin-top: 20px;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  background-color: #000;
  color: #fff;
  font-size: 1.2rem;
  font-weight: 700;
  cursor: pointer;
  z-index: 1;
}

button:hover {
  background-color: #333;
}

button:active {
  transition: all 0.1s ease-in-out;
  transform: scale(0.95);
}
</style>
