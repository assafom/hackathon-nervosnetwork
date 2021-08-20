# Port Truffle and React based DApp to Nervos' Polyjuice

## How I ported my Ethereum DApp to Polyjuice (and how you can too)

In this article we'll walk through porting an Ethereum DApp that runs on Truffle and React to Nervos Network's Polyjuice framework.

If you're here, I assume you know what these terms mean, and you have your own DApp you want to port, or at least understand what that entails.

Note that there are a few other resources that might be of interest to you:
- Nervos' team have supplied very clear instructions on how to port a DApp, however, in their example they deploy their contract via the DApp front end. If you're using Truffle for deploying your contracts, that will be less useful to you. [See here](https://gitcoin.co/issue/nervosnetwork/grants/8/100026214).
- If you just want a basic truffle project to use as a base, or to copy it's config file, you can use [simple-storage-v2](https://github.com/RetricSu/simple-storage-v2).
- For other different tutorial on how to port your DApp to Polyjuice, you can see the [Submissions at this hackathon](https://gitcoin.co/issue/nervosnetwork/grants/16/100026367).

But if you have your own DApp using Truffle that you'd like to port, this tutorial is for you and will include all the info from the previous bullet points. Let's begin.

The porting consists of few simple steps:

* Back-end:
  * Change truffle's network to a Godwoken testnet RPC, and set the providers to support Polyjuice.

* Front-end:
  * Change the Web3 provider to one that supports Polyjuice
  * Use a supplied module for translating addresses Ethereum <-> Godwoken
  * 
