ebakus based decentralized applications consist of two parts

1. the business logic that is developed through smart contracts in the solidity language and runs on the ebakus network
2. the client logic that interfaces with the smart contract.

To enable developers produce highly responsive and frictionless decentralized applications that are a delight for users to use, we have created the ebakus development stack.



## **The ebakus development stack **

The ebakus development stack consists of

- **[the ebakus node](https://github.com/ebakus/go-ebakus):**
  DPOS, 1 second blocks, ebakusDB

- **[web3-ebakus](https://github.com/ebakus/web3-ebakus):**
  is an extention of ethereum's popular web3 library to enable calculation of PoW and access ebakusDB

- **[the wallet-loader library](https://github.com/ebakus/ebakus-web-wallet-loader):**
  can be included on any static site and provide an interface to the ebakus blockchain through the web wallet.

- **[the ebakus web wallet](https://github.com/ebakus/ebakus-web-wallet):**

  Is the interface to the ebakus network.

The ebakus development stack allows for applications that are as easy to use, as any other application built on centralized infrastructure while remaining completely decentralized (without the need to rely on hybrid models).

## **The smart contract**

Smart contracts in the decentralized application context are the business logic and the blockchain is where they are stored and run.

They are essentially the backend of the application and they are special because they

* are immutable and produce deterministic results
* run forever
* cost nothing to run for the developers

As we mentioned before **the ebakus node** is 100% backwards compatible with ethereum. This allows you to use the vast resources available for developing smart contracts for solidity and apply everything you learn to ebakus to enjoy no fees, low latency and high throughput. Moreover ebakus extends solidity with ebakusDB a transactional decentralized Database that makes handling large datasets a breeze.

You can try out examples and tutorials you find through

1. the ebakus version of [Remix IDE](https://remix.ebakus.com) that will help you write, test and deploy solidity contracts for the ebakus network
2. our own [block explorer](https://explorer.ebakus.com)

If you already understand the basics of developing smart contracts you may find interesting [this article that introduces the ebakusDB](https://medium.com/ebakus/introducing-ebakusdb-part-1-7ebe5013c0d0).



## **The client logic**

The client logic on the other hand works a bit differently as the ebakus development stack utilizes the embedded wallet that drastically improves usability. Lets see just how this works in a bit more detail.

![alt text](../img/ebakus_if.jpg "ebakus interface")

The proposed way to handle client logic uses three components

1. the web3-ebakus: loads a web3 instance and extends it with web3 specific functions that allow you to access data stored in ebakusDB
2. the wallet loader library: loads the ebakus web wallet in an iframe and provides an interface between your dApp and the ebakus wallet without giving you access to uses keys. This method of loading the wallet allows many dApps share the same wallet when running in the same browser.

3. the ebakus web wallet hosted by a third party or yourself. Allows you to interface with the ebakus blockchain, send transactions, call contract functions and write data to the ebakusDB.

if you already have a dApp running on ethereum or you are following a tutorial written for ethereum this article about [migrating your ethereum based dApp to ebakus](./migrating-your-ethereum-d-app-to-ebakus.md) will be usefull.
