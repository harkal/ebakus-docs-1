## Free transactions

![alt text](/img/ebakus_if.jpg "dApp interfaces with ebakus")

One of the main reasons we decided to built ebakus was to improve usability of decentralized applications. When we started looking into viable business models and ideas for dApps, we quickly found out that fees and complex resource management systems really hindered usability. The requirement for users to own an initial token balance in order to interact with decentralized applications was really hurting onboarding of new users that had no previous experience with blockchain applications and it was one of the first problems we built ebakus to solve.

Blockchains today use fees in order to achieve two main goals. First, to mitigate malicious unsolicited flooding of the network with a huge number of transactions in order to affect the quality of service for normal operations, and use up storage and processing capacity. Second, to incentivise the miners or block producers as they collect those fees.

Adding fees to every transaction greatly hinders the usability of a blockchain. One of our main design goal with ebakus software is to provide free transactions. We do this by solving the two aforementioned problems. We solve the incentive problem by using inflation. The block producers don’t depend on the fees of transactions as the software constantly creates new ebakus coins as a reward for them. So now, we have to address the spamming of the network in order to make the inflation truly compensate for the lack of fees additionally to maintaining the quality of service.

We achieve this by utilising an algorithm that uses proof of work in combination with proof of stake. The initial invention of PoW was actually for use in mitigating network denial of service. Most blockchains ended up using fees in order to make it expensive to attack the network, while killing usability and eventually failing to maintain quality of service.

In ebakus  the network maintains a PoW quantum value. This is the minimum work required by a non stake holder -we will come back to that- for the unit operation on the network. Each operation on the network requires to consume a number PoW quanta in order to be accepted depending on the complexity of computation, the storage requirements, etc. The block providers adjust this PoW quantum to the level that allows normal operation of the network. For example if the network is idle with very little transactions being processed, transactions will require very low PoW to be accepted. In the event someone starts spamming the network with transactions the PoW quantum will be increased so it becomes computationally expensive for him to continue doing so.

However, in order to achieve quality of service for the legitimate users of the network ebakus does not operate entirely on the global PoW quantum. The global PoW quantum essentially the work quantum required by accounts holding zero amount of ebakus coins. The actual work quantum accepted is adjusted for each account as a function of its stake. Accounts that hold more ebakus coins -and hence have more stake- have a lesser PoW quantum.

The PoW quantum is adjusted in a way that, at anytime, the network resources are allocated proportionally to all the stake holders sending transactions.

Ebakus wallets will be able to recalculate the PoW required to send a transaction, so even in cases that the account has zero stake and the network is congested, the user experience will be smooth.

## Flow for PoW

1. Create a transaction object
2. Retrieve nonce for this transaction
3. Estimate Gas needed for this transaction
4. Get suggested difficulty for our account at current network state
5. Calculate PoW for this transaction

    !!! tip
        From this point, don't change any tx properties, as they have been used for calculating the PoW.

6. Send the transaction

## Sending transaction examples

Below you can find examples of sending a simple transaction either through go-ebakus console or your dApp's web3.js.

### From go-ebakus console

```js
> var tx = {
    from: eth.coinbase,
    to: '0x8f10d3a6283672ecfaeea0377d460bded489ec44',
    value: web3.toWei(10),
    nonce: eth.getTransactionCount(eth.coinbase)
};
> tx.gas = eth.estimateGas(tx);

> var txWithPow = eth.calculateWorkNonce(tx, eth.suggestDifficulty(eth.coinbase));

> eth.sendTransaction(txWithPow);
```

### From web3.js of your dApp

```js
var tx = {
    from: '0xB2b3510C106E8e04Acfb9841e2213500167100f3',
    to: '0x8f10d3a6283672ecfaeea0377d460bded489ec44',
    value: web3.toWei(10),
    nonce: eth.getTransactionCount('0xB2b3510C106E8e04Acfb9841e2213500167100f3')
};
tx.gas = eth.estimateGas(tx);

var txWithPow = eth.calculateWorkNonce(tx, eth.suggestDifficulty('0xB2b3510C106E8e04Acfb9841e2213500167100f3'));

eth.sendTransaction(txWithPow);
```
