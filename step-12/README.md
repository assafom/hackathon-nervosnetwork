# Port Truffle and React based DApp to Nervos' Polyjuice

## How I ported my Ethereum DApp to Polyjuice (and how you can too)

In this article we'll walk through porting an Ethereum DApp that runs on Truffle and React (based on create-react-app) to Nervos Network's Polyjuice framework.

If you're here, I assume you know what these terms mean, and you have your own DApp you want to port, or at least understand what that entails.

Note that there are a few other resources that might be of interest to you:
- Nervos' team have supplied very clear instructions on how to port a DApp, however, in their example they deploy their contract via the DApp front end. If you're using Truffle for deploying your contracts, that will be less useful to you. [See here](https://gitcoin.co/issue/nervosnetwork/grants/8/100026214).
- If you just want a basic truffle project to use as a base, or to copy it's config file, you can use [simple-storage-v2](https://github.com/RetricSu/simple-storage-v2).
- For other different tutorial on how to port your DApp to Polyjuice, you can see the [Submissions at this hackathon](https://gitcoin.co/issue/nervosnetwork/grants/16/100026367).

But if you have your own DApp using Truffle that you'd like to port, this tutorial is for you and will include all the info from the previous bullet points. Let's begin.

The porting consists of few simple steps:

* Truffle - deploying smart contracts:
  * Change truffle's network to a Godwoken testnet RPC, and set the providers to support Polyjuice.

* Front-end:
  * Change the Web3 provider to one that supports Polyjuice
  * Use a supplied module for translating addresses Ethereum <-> Godwoken
  * 

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

And where I previously created Web3 by executing ```const web3 = new Web3(window.ethereum);```, I change it to use the configured Polyjuice provider.
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

At this point I know I wasn't finished, but I thought that the front end should at least be running now so let's check it.

After getting an error when trying to run, I realised I should build the client using:
```
npm run-script build
```

Now the client has started, however, it couldn't load the contracts.

This is the method I was using to get a Web3 contract instance:

<img src="https://github.com/assafom/hackathon-nervosnetwork/blob/main/step-12/contract-set.png" width="500">

Upon debugging, I realised that networkId is coming as a hex string (116e1), while the PrisaleContract object.networks held the networkId as a dec (71393). So I changed the hex string to decimal using ParseInt.
Also, while looking at this code, I realised I use window.ethereum, which we substituted for the PolyjuiceHttpProvider previously. I'm not sure if it makes a difference in this scenario as the front end was working now, but I decided to change it also to use the PolyjuiceHttpProvider we set up earlier.

After these changes, this is how the code looked like:

<img src="https://github.com/assafom/hackathon-nervosnetwork/blob/main/step-12/contract-set-fixed.png" width="500">

And when running in the browser, I saw:

<img src="https://github.com/assafom/hackathon-nervosnetwork/blob/main/step-12/running1.png" width="500">

Yay, basically running!

But I knew we aren't finished yet; there are two things we haven't done. We haven't set a gas limit to the transactions, and we haven't converted Ethereum addresses to Godwoken addresses. And indeed, trying to execute a command that changes the contract's state (and not just reads info) has failed.

After adding dependedy, npm install, babel
