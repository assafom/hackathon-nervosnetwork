# Port Truffle and React CRA based DApp to Nervos' Polyjuice

## How I ported my Ethereum DApp to Polyjuice (and how you can too)

In this article we'll walk through porting an Ethereum DApp that runs on Truffle and React (based on create-react-app) to Nervos Network's Polyjuice framework.

If you're here, I assume you know what these terms mean, and you have your own DApp you want to port, or at least understand what that entails.

Note that there are a few other resources that might be of interest to you:
- Nervos' team have supplied very clear instructions on how to port a DApp, however, in their example they deploy their contract via the DApp front end. If you're using Truffle for deploying your contracts, that will be less useful to you. [See here](https://gitcoin.co/issue/nervosnetwork/grants/8/100026214).
- If you just want a basic truffle project to use as a base, or to copy it's config file, you can use [simple-storage-v2](https://github.com/RetricSu/simple-storage-v2).
- For other different tutorials on how to port your DApp to Polyjuice, you can see the [submissions at this hackathon](https://gitcoin.co/issue/nervosnetwork/grants/16/100026367).

But if you have your own DApp using Truffle and/or create-react-app that you'd like to port, this tutorial is for you and will include all the info from the previous bullet points. Let's begin.

The porting basically consists of few simple steps:

* Truffle - deploying smart contracts:
  * Change truffle's network to a Godwoken testnet RPC, and set the providers to support Polyjuice.

* Front-end:
  * Change the Web3 provider to one that supports Polyjuice
  * Use a supplied module for translating addresses Ethereum <-> Godwoken

### Truffle Deployment
The DApp I am currently working on hasn't been deployed yet, and my Truffle configuration only consisted of connecting to my local Ganache instance.

To get Truffle to connect and deploy to the Godwoken Testnet, we need to use custom providers for Ethereum libraries that will handle the changes between Ethereum and Godwoken-Polyjuice (such as different transaction structure and different signing mechanism).

Nervos has supplied us with these providers. I was using web3 (and not ethers) so this is the process I will document here.

To load environment variables, I used the dotenv node module.

First we need to install the required dependencies:

```npm install @polyjuice-provider/web3 @polyjuice-provider/truffle dotenv```

Then we add the dependencies to truffle-config.js:
```
const { PolyjuiceHDWalletProvider } = require("@polyjuice-provider/truffle");
const { PolyjuiceHttpProvider } = require("@polyjuice-provider/web3");
```

Now let's create the .env file which will contain the configuration. This configuration has been supplied by the Nervos team - other than the private key, in which you should put your own Ethereum private key.
```
WEB3_JSON_RPC=http://godwoken-testnet-web3-rpc.ckbapp.dev
ROLLUP_TYPE_HASH=0x4cc2e6526204ae6a2e8fcf12f7ad472f41a1606d5b9624beebd215d780809f6a
ETH_ACCOUNT_LOCK_CODE_HASH=0xdeec13a7b8e100579541384ccaf4b5223733e4a5483c3aec95ddc4c1d5ea5b22
PRIVATE_KEY=<ETHEREUM PRIVATE KEY HERE>
```

Back in truffle-config.js, we now need to load the environment variables:
```
const root = require("path").join.bind(this, __dirname, ".");
require("dotenv").config({ path: root(".env") });
```

Now we can access the configuration in truffle-config. Let's set up the custom providers. We set up two: a provider for Truffle (PolyjuiceHDWalletProvider) which will use a provider for web3 (PolyjuiceHttpProvider).
```
const rpc_url = new URL(process.env.WEB3_JSON_RPC);
 
const godwoken_rpc_url = process.env.WEB3_JSON_RPC;
const polyjuice_config = {
  rollupTypeHash: process.env.ROLLUP_TYPE_HASH,
  ethAccountLockCodeHash: process.env.ETH_ACCOUNT_LOCK_CODE_HASH,
  web3Url: godwoken_rpc_url,
};

const polyjuiceHttpProvider = new PolyjuiceHttpProvider(
   polyjuice_config.web3Url,
   polyjuice_config
 );
 const polyjuiceTruffleProvider = new PolyjuiceHDWalletProvider(
   [
     {
       privateKeys: [process.env.PRIVATE_KEY],
       providerOrUrl: polyjuiceHttpProvider,
     },
   ],
   polyjuice_config
 );
```
Now we only need to configure the truffle network to connect to the Godwoken RPC we configured, and use the Polyjuice provider.
```
networks: {
      development: {
       host: rpc_url.hostname, // Localhost (default: none)
       port: rpc_url.port, // Standard Ethereum port (default: none)
       gasPrice: "0", // notice: `gasPrice: 0` won't work in dryRun mode. 0 must be string type.
       network_id: "*", // Any network (default: none)
       provider: () => polyjuiceTruffleProvider,
     }
   }
```
And that's it for truffle-config. I also have a contracts_build_directory config that points to the client directory where the front end will read the contract artifact from.

#### The final truffle-config.js file
```
const { PolyjuiceHDWalletProvider } = require("@polyjuice-provider/truffle");
const { PolyjuiceHttpProvider } = require("@polyjuice-provider/web3");

const root = require("path").join.bind(this, __dirname, ".");
require("dotenv").config({ path: root(".env") });

const rpc_url = new URL(process.env.WEB3_JSON_RPC);

const godwoken_rpc_url = process.env.WEB3_JSON_RPC;
const polyjuice_config = {
  rollupTypeHash: process.env.ROLLUP_TYPE_HASH,
  ethAccountLockCodeHash: process.env.ETH_ACCOUNT_LOCK_CODE_HASH,
  web3Url: godwoken_rpc_url,
};

const polyjuiceHttpProvider = new PolyjuiceHttpProvider(
  polyjuice_config.web3Url,
  polyjuice_config
);
const polyjuiceTruffleProvider = new PolyjuiceHDWalletProvider(
  [
    {
      privateKeys: [process.env.PRIVATE_KEY],
      providerOrUrl: polyjuiceHttpProvider,
    },
  ],
  polyjuice_config
);

module.exports = {
contracts_build_directory: require("path").join(__dirname, "client/src/contracts"),
  networks: {
    development: {
      host: rpc_url.hostname, // Localhost (default: none)
      port: rpc_url.port, // Standard Ethereum port (default: none)
      gasPrice: "0", // notice: `gasPrice: 0` won't work in dryRun mode. 0 must be string type.
      network_id: "*", // Any network (default: none)
      provider: () => polyjuiceTruffleProvider,
    }
  },

  // Configure your compilers
  compilers: {
    solc: {
      version: "0.7.6"
    },
  },

  db: {
    enabled: false,
  },
};
```

If you followed this far, you can run ```truffle migrate --reset``` and see the contracts being deployed. Make sure your Ethereum address is funded [(see here)](https://gitcoin.co/issue/nervosnetwork/grants/2/100026208).

Note: my truffle deployment is a usual affair:

<img src="https://github.com/assafom/hackathon-nervosnetwork/blob/main/step-12/truffle-deploy.png" width="500">

And after the deployment, I initialise the contracts with some basic values:

<img src="https://github.com/assafom/hackathon-nervosnetwork/blob/main/step-12/truffle-init.png" width="500">

At this point, when the contracts have been deployed, we can already see something interesting: the conversion between Ethereum address and Godwoken address. When in the init script I print the contract's owner, I'm getting my Godwoken address.



### Front-end
There are basically two things that are needed here.

1. Essentially, my DApp instanstiated web3 using ```new Web3(window.ethereum)```. We need to replace this with a web3 Polyjuice provider.

2. We need to be able to convert Ethereum addresses to Godwoken addresses. This we can do with the nervos-godwoken-integration module.

Before this, if you didn't yet configure Godwoken Testnet on your MetaMask, do so now.
```
Network Name: Godwoken Testnet
RPC URL: https://godwoken-testnet-web3-rpc.ckbapp.dev
Chain ID: 71393
Currency Symbol: <Leave Empty>
Block Explorer URL: <Leave Empty>
```

So first let's navigate to the front-end directory and install the modules:
```
npm install @polyjuice-provider/web3@0.0.1-rc7 nervos-godwoken-integration@0.0.6
```

After doing so, I have created a config file (similar to the Truffle config file) with the settings for the Polyjuice provider. The file is named 'config.js' and is placed at my main src client directory.
```
export const CONFIG = {
    WEB3_PROVIDER_URL: 'https://godwoken-testnet-web3-rpc.ckbapp.dev',
    ROLLUP_TYPE_HASH: '0x4cc2e6526204ae6a2e8fcf12f7ad472f41a1606d5b9624beebd215d780809f6a',
    ETH_ACCOUNT_LOCK_CODE_HASH: '0xdeec13a7b8e100579541384ccaf4b5223733e4a5483c3aec95ddc4c1d5ea5b22'
};
```

In my React app, I instantitate Web3 from a file called getWeb3.js. So there I have imported the dependencies:
```
import { PolyjuiceHttpProvider } from '@polyjuice-provider/web3';
import { CONFIG } from './config.js';
```

And where I previously created Web3 by executing ```const web3 = new Web3(window.ethereum);```, I changed it to use the configured Polyjuice provider.
```
const godwokenRpcUrl = CONFIG.WEB3_PROVIDER_URL;
const providerConfig = {
    rollupTypeHash: CONFIG.ROLLUP_TYPE_HASH,
    ethAccountLockCodeHash: CONFIG.ETH_ACCOUNT_LOCK_CODE_HASH,
    web3Url: godwokenRpcUrl
};
const provider = new PolyjuiceHttpProvider(godwokenRpcUrl, providerConfig);
const web3 = new Web3(provider);
```

At this point I knew I wasn't finished, but I thought that the front end should at least be running now so let's check it.

The client has started, however it couldn't load the contracts.

This is the method I was using to get a Web3 contract instance:

<img src="https://github.com/assafom/hackathon-nervosnetwork/blob/main/step-12/contract-set.png" width="500">

Upon debugging, I realised that networkId is coming as a hex string (116e1), while the PrisaleContract object.networks held the networkId as a dec (71393). So I changed the hex string to decimal using parseInt.
Also, while looking at this code, I realised I use window.ethereum, which we substituted for the PolyjuiceHttpProvider previously. I'm not sure if it makes a difference in this scenario as the front end was working now, but I decided to change it also to use the PolyjuiceHttpProvider we set up earlier.

After these changes, this is how the code looked like:

<img src="https://github.com/assafom/hackathon-nervosnetwork/blob/main/step-12/contract-set-fixed.png" width="500">

And when running in the browser, I saw:

<img src="https://github.com/assafom/hackathon-nervosnetwork/blob/main/step-12/running1.png" width="500">

Yay, basically running!

But I knew we aren't finished yet; there are two things we haven't done. We haven't set a gas limit to the transactions (at the moment needed for the Polyjuice provider), and we haven't imported the AddressTranslator so we can convert Ethereum addresses to Godwoken addresses.

### Setting a 6000000 gas limit
We should add a gaslimit to every transaction, for example like this:
```
this.contract.methods.set(value).send({gas: 6000000, from: fromAddress});
```
Per my understanding there should have been a way to do it automatically when we create the web3 Contract object, but it didn't work for me. So I just changed all my transactions to send gas: 6000000.

### Adding the AddressTranslator

I started by importing the dependency into App.js:
```
import { AddressTranslator } from 'nervos-godwoken-integration';
```
And inside function App() instantiating it:
```
const addressTranslator = new AddressTranslator();
```

Then tried to run, but got the following error.

<img src="https://github.com/assafom/hackathon-nervosnetwork/blob/main/step-12/react-error.png" width="700">

After some googling, it seemed like the issue could be fixed with a Babel plugin. However, I wasn't using Babel as a depedency at the time; I'm new to front-end and didn't see the need. So I tried to install it with the required plugin, but nothing changed. I vaguely recalled seeing a similar issue in the Nervos discord, so I asked for help there.

Thankfully, danielkmak from Nervos crew replied that he knows how to fix the issue.

The problem was that create-react-app uses a minimal Babel config to process the app's dependencies, even if you try to specify otherwise in your babel config. This babel config will modify the config for your app, not the config for your dependencies.

Daniel knew of two ways to fix it:

1. Execute ```yarn eject``` and then add the needed Babel plugins to config/webpack.config.ts
2. Use [react-app-rewired](https://github.com/timarney/react-app-rewired) to modify the webpack config without ejecting.

If you want to use the yarn eject method, you can try to match your webpack config file to the one Daniel provided [here](https://github.com/chaitanyasjoshi/social-network-dapp-polyjuice/blob/29cd1550f197582c49897de6edfaecc60b6e3912/config/webpack.config.js).

I decided to try to use **react-app-rewired**.

The [react-app-rewired repo](https://github.com/timarney/react-app-rewired) contains easy instructions for installing it.

After installing, I needed to actually modify the webpack config to suit my needs, but I didn't even know how the config looks like :)

After printing, some debugging, and comparing it to Daniel's example (which was generated from yarn eject and contained more things than I needed), I managed to create a config file that's working. Note, that this was my first time using Babel/Webpack. I can not guarantee I didn't mess something up, and if you'll be using this, you probably wanna get somebody who's familiar with B/W to verify it's ok. But it seems to be working.

This is my config-overrides.js. For brevity sake I removed the comments; if you'd be using this officialy, check out [Daniel's config](https://github.com/chaitanyasjoshi/social-network-dapp-polyjuice/blob/29cd1550f197582c49897de6edfaecc60b6e3912/config/webpack.config.js). At the very least you should change the "include" object to be your source folder.
```
module.exports = function override(config, env) {
    config.module.rules.push({
        // For client source files (not dependencies)
        test: /\.(js|mjs|jsx|ts|tsx)$/,
        include: require("path").join(__dirname, "/src"),
        loader: require.resolve('babel-loader'),
        options: {
          customize: require.resolve(
            'babel-preset-react-app/webpack-overrides'
          ),
          presets: [
            [
              require.resolve('babel-preset-react-app'),
              {
                runtime: 'automatic',
              },
            ]
          ],
          plugins: [
            [
              require.resolve('babel-plugin-named-asset-import'),
              {
                loaderMap: {
                  svg: {
                    ReactComponent:
                      '@svgr/webpack?-svgo,+titleProp,+ref![path]',
                  },
                },
              },
            ],
          ].filter(Boolean),
          cacheDirectory: true,
          cacheCompression: false,
          compact: true,
        },
      },{
        // For dependencies
        test: /\.(js|mjs)$/,
        exclude: /@babel(?:\/|\\{1,2})runtime/,
        loader: require.resolve('babel-loader'),
        options: {
          babelrc: false,
          configFile: false,
          compact: false,
          presets: [
            [
              require.resolve('babel-preset-react-app/dependencies'),
              { helpers: true },
            ],
          ],
          plugins: [
            "@babel/plugin-proposal-class-properties",
            "@babel/plugin-syntax-bigint",
            "@babel/plugin-proposal-nullish-coalescing-operator",
            "@babel/plugin-syntax-jsx",
            "@babel/plugin-proposal-optional-chaining",
          ],
          cacheDirectory: true,
          cacheCompression: false,
        },
      }
    );
    return config;
  }
```

Now the app was able to load the AddressTranslator library. I've printed the user's Polyjuice address to the screen.
```
Loaded Polyjuice address: <b>{accounts && accounts[0]?addressTranslator.ethAddressToGodwokenShortAddress(accounts[0]) : undefined}</b><br/>
(Based on Ethereum address: {accounts && accounts[0]?accounts[0] : undefined})<br/>
```

There were two more things I needed to change:
1. To display a button that only the contract owner can use, I previously checked whether accounts[0]==contract.owner(). However, accounts[0] is an eth address, and contract.owner() is Polyjuice. So I converted the accounts[0] address to Polyjuice using addressTranslator.ethAddressToGodwokenShortAddress and checked if they're equal.
2. For testing/development purposes, I had some hardcoded ethereum addresses in my smart contract and front end; I realised I need to change them to their Polyjuice equivalent.

**And... that's it!**

<img src="https://github.com/assafom/hackathon-nervosnetwork/blob/main/step-12/running3.png" width="700">

**The smart contracts and app were working straight out-of-the-box**.

This is the power of Godwoken-Polyjuice I think. In the actual functionality and business logic, nothing needed to be changed. Only a minimal changes to the front end transcations (gas:6000000) and conversion of hardcoded addresses.

My DApp is utilised for OTC token presales. The admin can add a new campaign (for example for a new DEX being launched and wants people to prebuy its tokens), with a name and a price per token. Then the user can buy tokens, doing so transfering ERC20 token (mocking USDT) to the main contract using the ERC20 approval mechanism. The contract keeps tracks of how many tokens each address has bought. Then, the admin can close the sale, set the address for the new token, and distribution begins. Distribution of the presale token can happen in stages, eg. 10% is being released every month for 10 months. Each time the user calls the contract's claim function, the contract checks if any new token releases have arrived, and if so, sends the user's percentage of the token release. So if for example User A bought 10 tokens and user B bought 10 tokens, and 6 tokens have been released, and User A calls the claim button, he will receive 3 tokens. All this funcionality was working without needing changes. Pretty cool. 

You can watch a 7 minutes video of me using the deployed DApp [here](https://youtu.be/8ZSnclqk0jk).

That's it :) Hope you found this helpful. If you need help, you are welcome in the [Nervos discord](https://discord.com/invite/AqGTUE9).
