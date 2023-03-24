<script setup lang="ts">
import {computed } from "vue";

const props = defineProps<{
  isConnected: boolean;
  action: () => void;
}>()

const connectWalletLabel = computed(() => {
  return props.isConnected ? "Connected" : "Connect Wallet";
})
</script>

<template>
  <header>
    <nav>
      <img class="logo" src="../assets/logo-meka-ape.png" alt="logo">

      <div class="wallet-connect-button" @click="action">
        <div class="wallet-connect-button">
          <span>{{ connectWalletLabel }}</span>
          <span class="dot" :class="isConnected ? 'green' : 'red'"/>
        </div>
      </div>
    </nav>
  </header>
</template>


<style scoped>
nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 40px 70px;
  height: 80px;

  margin-bottom: 30px;
}

@media (max-width: 768px) {
  nav {
    margin-bottom: 5px;
    padding: 20px 20px;
  }
}

img.logo {
  width: 108px;
  cursor: pointer;
  user-select: none;
}

.wallet-connect-button {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 120px;

  cursor: pointer;
  user-select: none;
}

.dot {
  height: 7px;
  width: 7px;
  border-radius: 50%;
  display: inline-block;
}

.red {
  background-color: #FF1A1A;
}

.green {
  background-color: #1AFF72;
}
</style>

