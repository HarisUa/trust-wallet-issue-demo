<script setup lang="ts">
import { SignClient } from '@walletconnect/sign-client'
import { WalletConnectModal } from '@walletconnect/modal'
import { TronWeb, utils } from 'tronweb'
import { onMounted, ref } from 'vue'

const projectId = import.meta.env.VITE_ENV_PROJECT_ID

const modal = new WalletConnectModal({
  projectId,
})

const client = ref()

onMounted(async () => {
  client.value = await SignClient.init({
    projectId,
  })
})

function isMobile() {
  const userAgent = navigator.userAgent || navigator.vendor
  const mobileRegex = /android|iphone|ipad|ipod|blackberry|iemobile|opera mini/i
  return mobileRegex.test(userAgent)
}

async function connect() {
  const { uri, approval } = await client.value.connect({
    requiredNamespaces: {
      tron: {
        chains: ['tron:0x2b6653dc'],
        methods: ['tron_signTransaction', 'tron_signMessage'],
        events: [],
      },
    },
  })

  if (!uri) {
    return
  }

  if (isMobile()) {
    const trustLink = `https://link.trustwallet.com/wc?uri=${encodeURIComponent(uri)}`
    window.location.href = trustLink
  } else {
    modal.openModal({ uri })
  }

  const session = await approval()
  const tronWeb = new TronWeb({
    fullHost: 'https://api.trongrid.io',
  })

  const account = session.namespaces.tron.accounts[0]
  const address: string = utils.address.toHex(account.split(':')[2])

  const blackHoleAddress = utils.address.toHex('T9yD14Nj9j7xAB4dbGeiX9h8unkKHxuWwb')
  const encodedTx = await tronWeb.transactionBuilder.sendTrx(blackHoleAddress, 1, address)

  try {
    await client.value.request({
      topic: session.topic,
      chainId: 'tron:0x2b6653dc',
      request: {
        method: 'tron_signTransaction',
        params: { address, transaction: encodedTx },
      },
    })

    // other flow that didn't executed due to closed tab
    // ...
  } catch (error) {
    console.error('Transaction failed:', error)
  }
}
</script>

<template>
  <div>
    <button v-on:click="connect" v-if="client">Connect</button>
    <p v-else>Loading...</p>
  </div>
</template>

<style scoped>
header {
  line-height: 1.5;
  max-height: 100vh;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

nav {
  width: 100%;
  font-size: 12px;
  text-align: center;
  margin-top: 2rem;
}

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  nav {
    text-align: left;
    margin-left: -1rem;
    font-size: 1rem;

    padding: 1rem 0;
    margin-top: 1rem;
  }
}
</style>
