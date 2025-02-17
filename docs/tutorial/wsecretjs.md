# wSecretJS

Now that we have a wallet connected, and you see how easy it is to get setup, the next step is to start interacting with the blockchain directly. For that we will need a SecretJS client.

## Background
SecretJS is a really powerful library. It wraps cosmjs which handles all sorts of tricky stuff like connecting to a wallet, signing transactions, and other low level stuff. What it adds to Cosmjs is the stuff that is important to Secret Network, namely client-side encryption. The problem, from out "lets get you off the ground fast and easy" perspective, is that the power and complexity come with a trade-off, it is quite difficult to use. In addition, SecretJS is mainly focused on interacting with contracts, not with core Secret Network modules like staking or governance. Meaning anything you want to do on that front needs to be manually built using similarly low level stuff. In other words its hard. 

Wouldn't it be great if their was a way to bundle all that, every interaction you will have with the blockchain in an app, into one clean, easy to use wrapper? We thought so too, so we have added wSecretJS to the stack.

Well, let's use it shall we?

## wSecretJS Client

The first thing we are going to need is to import two functions from Core Griptape: `createScrtCLient` and  `useWallet`. 

```javascript
import { coinConvert, createScrtClient, useWallet } from '@stakeordie/griptape.js'
```

This may seem a little strange at first. Didn't we already import `useWalletStore` in the last section? What is `useWallet` and why do we need it? 

Well, remember how Griptape.js comes in two parts, core and flavored. So far we used core Griptape directly only to get the `coinConvert` utility.  Griptape-vue is were we have seen, among other things, state management related stuff. This is because Pinia, the state management library, is a Vue.JS library. So when we imported `useWalletStore`, what were were importing was a Pinia store that lives in the griptape-vue layer. Underneath that, is `useWallet`, which is not a pinia store, and lives in core griptape library, and it's purpose is to act as the connective tissue needed to interact with a wallet, in this case Keplr. I hope that wasn't too confusing. Later when we start to use **Griptape Contract Stores** most of this will be abstracted away, but for now it is good to see how it works.

So first we will import those libraries and create a wSecretJS client.

**/src/App.vue**
```html
...

<script>
import { coinConvert, createScrtClient, useWallet } from '@stakeordie/griptape.js'

const wallet = await useWallet();
const wscrtClient = await createScrtClient('https://api.holodeck.stakeordie.com', wallet);

export default {

}
</script>

...
```

## First Query

Lets use the trusty Secret Counter contract to test. First we'll just query the contract and output the result.

**/src/App.vue**
```html
<template>
  <div>
    <header>
      <div class="logo">GRIPTAPE.JS</div>
      <wallet-info></wallet-info>
    </header>
    <main>
      <div>{{ count }}</div>
    </main>
  </div>
</template>

<script>
import { coinConvert, createScrtClient, useWallet } from '@stakeordie/griptape.js'

const wallet = await useWallet()
const wscrtClient = await createScrtClient('https://api.holodeck.stakeordie.com', wallet)

const secretCounterAddress = 'secret1w97ynhe099cs5p433dvlaqhsxrszudz2n3f56h'

export default {
  created() {
    //this.getActiveAuctions()
    this.getCount();
  },
  data() {
    return {
      count: 0
    }
  },
  methods: {
    async getCount() {
      const msg = {'get_count':{}}

      const res = await wscrtClient.queryContract(secretCounterAddress, msg)

      this.count = res.count
    }
  }
}
</script>
```

The above code used a vue.js lifecycle hook `created() {}` to trigger the method `getTheCount()`. `getTheCount` then queries the contract `secretCounterAddress = 'secret1w97ynhe099cs5p433dvlaqhsxrszudz2n3f56h'` and assigns the response value to the data oject propery `theCount`. Finally `theCount` is included in the vue template and is outputted to the screen.

This might not seem like much, but we just queried a contract on chain. That is no small accomplishment. Now lets execute a transaction to show the second part of the equation.

## First Transaction

```html
<template>
  <div>
    <header>
      <div class="logo">GRIPTAPE.JS</div>
      <wallet-info></wallet-info>
    </header>
    <main>
      <div>Count is: {{ count }}</div>
      <button @click="increment">+</button>
      <button @click="decrement">-</button>
    </main>
  </div>
</template>

<script>
import { coinConvert, createScrtClient, useWallet } from '@stakeordie/griptape.js'

const wallet = await useWallet()
const wscrtClient = await createScrtClient('https://api.holodeck.stakeordie.com', wallet)

const secretCounterAddress = 'secret1w97ynhe099cs5p433dvlaqhsxrszudz2n3f56h'

export default {
  created() {
    this.getCount();
  },
  data() {
    return {
      count: 0
    }
  },
  methods: {
    async increment() {
      const handleMsg = { 'increment': { } }
      await wscrtClient.executeContract(secretCounterAddress, handleMsg)
      this.getCount()
    },
    async getCount() {
      const msg = { 'get_count': { } }
      const res = await wscrtClient.queryContract(secretCounterAddress, msg)
      this.count = res.count
    }
  }
}
</script>
```

There you have it, a fully functional dApp on Secret Network.

To sum up, by clicking the button we trigger method `increment`. Increment use the `wsjs.executeContract()` method to send a transaction to the chain. Then, in turn, `getTheCount()` is called which queries to get the updated number.

## Up Next

So this is great, it works, but it has issues. For one the state of the contract is being represented in this one component. That is okay when the app only has one, but what about when it gets bigger. When you have 5 components, 20, 100? 

This problem is one that application frameworks like Vue and React have dealt with by extracting the state and it's associate logic into stores that are accessible throughout the app. When building Griptape, we knew we wanted do this early so that apps have reactive state by default. And this is where **Griptape Contract Stores** come in.
