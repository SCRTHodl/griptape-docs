# Part 2

## Introduction

Part two is about building so lets get right to it.

First we want a fresh project to work with. For that we could go through and delete everything we did, or even better lets' just scrap what we have and start again with the starter app.

Delete the last project then run:

```bash
  degit https://github.com/stakeordie/griptape-vue-starter#main auctions-tutorial
  cd auctions-tutorial
  yarn && yarn dev
```

Voila!

![](/part-two-new-project.png)

The starter app is a common starting place for all apps which is great, but for out purposes we need to make some small changes.

First lets update `App.vue` to strip it of everything and then add back in a Logo and the wallte-info component

**/src/App.vue**
```html
  <template>
    <div>
      <header>
        <div class="logo">New Griptape App</div>
        <wallet-info></wallet-info>
      </header>
    </div>
  </template>

  <script>
    export default {
      
    }
  </script>

  <style>

  </style>
```

Next we will update `main.js`.

```javascript
import App from './App.vue'
import { gripVueJsApp } from '@stakeordie/griptape-vue.js'

import "~/@stakeordie/griptape-vue.js/dist/style.css"
import '@/assets/styles/index.scss'

const conf = {
  restUrl: 'https://api.holodeck.stakeordie.com'
}

gripVueJsApp(conf, App, (app, pinia) => {})
```
The only thing we are doing is adding the (app,pinia) => {} callback.



**auctions-factory.js**
```javascript
export const auctionsFactoryDef = {
  state: {
    auctions: {},
  },
  messages: {

  },
  queries: {
    async listAuctions() {
      const auctions = await this.scrtClient.queryContract(this.contractAddress, {"list_active_auctions":{}})
      this.auctions = auctions.list_active_auctions;
    },
  }
}
```

The `listAuctions` method will query the contract on chain and save the response in the state.

To do this we need to instantiate, or "create" the contract in `index.js`

**/src/contracts/index.js**
```javascript
import { createContract } from '@stakeordie/griptape-vue'
import { auctionsFactory } from './auctions-factory.js'

export const auctionsFactory = createContract(
  'auctions-factory',
  'secret1lqdx8va86f9cff5dsz28l97x20z67qv7d4npj8',
  auctionsFactory
)
```

Now, just like useWalletStore, we have our a custom pinia store that mirrors the contract on chain. Lets try it out.

## Display Auctions

As with the useWalletStore, we are going to mapState useAuctionsFactory, and ask for the auctions state object.

*NOTE: The wallet balance was removed from the code as it was not longer needed.

**/src/App.vue**
```html
...
<script>
  import { mapState, mapActions } from 'pinia'
  import { coinConvert } from '@stakeordie/griptape.js'
  import { useWalletStore,} from '@stakeordie/griptape-vue.js'
  import { useAuctionsFactoryStore } from '@/contracts'

  export default {
    created() {
      this.listAuctions()
    },

    computed: {
      ...mapState(useAuctionsFactoryStore, ['auctions']}),
    },

    methods: {
      ...mapActions(useAuctionsFactoryStore, ['listAuctions']),
    }
  }
</script>

```

There are a few things going on here.
- We are adding `mapActions` to go along with `mapState`. Pinia stores are made up of actions and state. When the contract is create all messages and queries are converted to actions. mapActions thus gives us access to the listAuctions query we defined in auctions-factory.js.

- We are pulling in the auctions property with mapState and the listAuctions query with mapActions.

- We are using the Vue.js lifecycle hook, to run the `listAuctions()` action. This will query the chain and save all auctions to the contract's auctions property.

- Finally we add a computed property, auctions, useing the `mapState` method.

This then culminates in the ability to use the auctions object in the template. We are actually only going to look at active auctions to reduce the size of the list.

**/src/App.vue**
```html
<template>
  <div>
    <header>
      <div class="logo">GRIPTAPE.JS</div>
      <wallet-info></wallet-info>
    </header>
    <main>
      <div class="auctions">
        <h2>Open Auctions</h2>
        <ul>
          <li v-for="(auction, index) in auctions.active" :key="index">
            <h3>{{ auction.pair }}</h3>
            <h4>{{ auction.label }}</h4>
            <div><span>Sell amount:</span> {{ auction.sell_amount }}</div>
            <div><span>Minimum bid:</span> {{ auction.minimum_bid }}</div>
            <div><span>Target close:</span> {{ auction.ends_at }}</div>
          </li>
        </ul>
      </div>
    </main>
  </div>
</template>
```

Lets clean this up a bit. Stop the server, run `yarn add moment`, then start again `yarn dev`. We will use moment to format the datetimes.

We will use the coinConvert utility as well. To use it in the template you need to add it to the methods section. The fill should now look like this:

**/src/App.vue**
```html{14-16,29,39-44}
<template>
  <div>
    <header>
      <div class="logo">GRIPTAPE.JS</div>
      <wallet-info></wallet-info>
    </header>
    <main>
      <div class="auctions">
        <h2>Open Auctions</h2>
        <ul>
          <li v-for="(auction, index) in auctions.active" :key="index">
            <h3>{{ auction.pair }}</h3>
            <h4>{{ auction.label }}</h4>
            <div><span>Sell amount:</span> {{ coinConvert(auction.sell_amount, auction.sell_decimals, 'human', 2) }}</div>
            <div><span>Minimum bid:</span> {{ coinConvert(auction.minimum_bid, auction.bid_decimals) }}</div>
            <div><span>Target close:</span> {{ formatDate(auction.ends_at) }}</div>
          </li>
        </ul>
      </div>
    </main>
  </div>
</template>

<script>
  import { mapState, mapActions } from 'pinia'
  import { coinConvert } from '@stakeordie/griptape.js'
  import { useWalletStore } from '@stakeordie/griptape-vue.js'
  import { useAuctionsFactoryStore } from '@/contracts'
  import moment from 'moment'

  export default {
    created() {
      this.listAuctions();
    },
    computed: {
      ...mapState(useAuctionsFactoryStore, ['auctions']),
    },
    methods: {
      coinConvert,
      formatDate(value) {
        if (!value) return ''

        return moment(value * 1000).format('lll')
      },
      ...mapActions(useAuctionsFactoryStore, ['listAuctions'])
    }
  }
</script>
```

And your app should look like this:

![](/tutorial/the-contract/list-auctions.png)

Next lets talk about viewing keys -->
