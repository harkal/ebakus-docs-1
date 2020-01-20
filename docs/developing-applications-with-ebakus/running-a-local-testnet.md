# Running a local testnet

## Sync to the public testnet with ebakus node

For syncing with the testnet the easiest way is using docker. For doing this, you can follow the instructions [here](./the-ebakus-development-stack/ebakus-node.md#sync-with-the-ebakus-test-network) or use this command directly to start playing.

```bash
docker run -ti --name ebakus-testnet-node \
           -v ~/ebakus:/root \
           -p 30403:30403 \
           -p 8545:8545 \
           ebakus/go-ebakus \
           --testnet \
           --rpc --rpcaddr 0.0.0.0 \
           console
```

### Interact with the ebakus node

After the node syncs with the network and you attach to it you will be able to issue commands in the console. For example:

```js
> eth.getBlock(7)

{
    delegateDiff: [
        {
            DelegateAddress: "0x92f00e7fee2211d230afbe4099185efa3a516cbf",
            DelegateNumber: 0,
            Pos: 0
        },{
            DelegateAddress: "0x4ed2aca4f9b3ee904ff44f03e91fde90829a8381",
            DelegateNumber: 0,
            Pos: 1
        }
    ],
    delegates: [
        "0x92f00e7fee2211d230afbe4099185efa3a516cbf",
        "0x4ed2aca4f9b3ee904ff44f03e91fde90829a8381"
    ],
    gasLimit: 6283185,
    gasUsed: 54144,
    hash: "0x24ddb825da11ee358dc5715e9114aeb5b9e2bdd9515bf77dfe90be34388a9685",
    number: 7,
    parentHash: "0x6e42c2731e7fb8b50ffebf37fa4067b3af2ada342535ea1a1e0d5ea97dcaa1e1",
    producer: "0x4ed2aca4f9b3ee904ff44f03e91fde90829a8381",
    receiptsRoot: "0x203b201c313a20355c1a137056b782c7572d3dee465e103708045cf33dbc8c95",
    size: 792,
    timestamp: 1545512856,
    transactions: [
        "0xda07c8239b27a0e0f0a25ac41fb090a0efefe3321d0f3c667bc85d9d57b3e6b9",
        "0xcde95fc9b4295c0b4836dd0b77e663f072bdac2df16fd2a97b0fd64d4e016be9"
    ],
    transactionsRoot: "0x53c7a83d6024b31b8221c55f04ae666149cdbc968ccc39d2dd1481bd0affb13f"
}
```

As you see the API follows the ethereum compatibility mode. The low level block structure however is different, and contains information relevant to DPOS and does not contain uncles, pow, and other ethereum related data.

Using the javascript console of the Ebakus node you can deploy contracts and issue actions on existing contracts normally.

**Example of sending ebakus tokens:**

```js
var tx = {
    from: eth.coinbase,
    to: '0x8f10d3a6283672ecfaeea0377d460bded489ec44',
    value: web3.toWei(10),
    nonce: eth.getTransactionCount(eth.coinbase)
};
tx.gas = eth.estimateGas(tx);

var txWithPow = eth.calculateWorkNonce(tx, eth.suggestDifficulty(eth.coinbase));

eth.sendTransaction(txWithPow);
```

As you see, no gas needs to be spend, hence no gas price is set. For more information on PoW check [here](./proof-of-work.md).

#### Deploying your Ethereum Contracts on Ebakus

1. Compile your contract

    ```shell
    solcjs --abi game.sol
    solcjs --bin game.sol
    ```

2. Display the compiled code on the console

    ```shell
    more game_sol_game.abi
    more game_sol_game.bin
    ```

3. Move back to our synced nodes' console and unlock your account

    ```js
    > personal.unlockAccount(eth.coinbase)

    Unlock account 0x....
    Passphrase: [ENTER PASSPHRASE]
    ```

4. Setup abi and bytecode

    ```js
    > var myContract = eth.contract([CONTENTS OF ABI FILE])
    > var bytecode = '0x[CONTENTS OF BIN FILE]'
    ```

    !!! tip
        Replace `[CONTENTS OF ABI FILE]` and `[CONTENTS OF BIN FILE]` with the outputs of the more commands that should still be open in another terminal window.

5. Deploy

    ```js
    > var tx = {
        from: eth.coinbase,
        data: bytecode,
        nonce: eth.getTransactionCount(eth.coinbase)
    };

    // add constructor args, if needed
    // > tx.data = contract.getData(arg1, arg2, tx);

    > tx.gas = eth.estimateGas(tx);
    > var txWithPow = eth.calculateWorkNonce(tx, eth.suggestDifficulty(eth.coinbase));

    > var game = myContract.new(txWithPow)
    ```

6. Interact with contract

    ```js
    > game.playMove('forward')
    ```
