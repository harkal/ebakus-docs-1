# Running a local testnet

### Working with the ebakus full node

#### Connecting to the public testnet with ebakus node

```bash
    $ ebakus --testnet console
```

After the node syncs with the network you will be able to issue commands in the console. For example:

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
> eth.sendTransaction({
    from:'0xB2b3510C106E8e04Acfb9841e2213500167100f3',
    to: '0x8f10d3a6283672ecfaeea0377d460bded489ec44',
    value:web3.toWei(10)
})
```

As you see, no gas needs to be spend, hence no gas price is set. Calculation of PoW for the transactions is automaticaly handled by sendTransaction.

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

3. We continue on our ebakus node

    ```shell
    $ ebakus --testnet --verbosity=2 console
    ```

    You should see this:

    ```js
    Welcome to the Ebakus JavaScript console!
    instance: Ebakus/v1.8.20-unstable/darwin-amd64/go1.11.2
    coinbase: 0x32f14386cea573bba82282b6f449ee77030a96e2
    at block: 89471 (Mon, 24 Dec 2018 00:00:38 EET)
      datadir: /Users/harkal/ebakus2
      modules: admin:1.0 debug:1.0 dpos:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0
      txpool:1.0 web3:1.0
    >
    ```

4. Unlock you account

    ```js
    > personal.unlockAccount(eth.coinbase)

    Unlock account 0x....
    Passphrase: [ENTER PASSPHRASE]
    ```

5. Setup abi and bytecode

    ```js
    > var myContract = eth.contract([CONTENTS OF ABI FILE])
    > var bytecode = '0x[CONTENTS OF BIN FILE]'
    ```

    !!! tip
        Replace [CONTENTS OF ABI FILE] and [CONTENTS OF BIN FILE] with the outputs of the more commands that should still be open in another terminal window.

6. Deploy

    ```js
    > var game = myContract.new({from:eth.coinbase, data:bytecode, gas: 2000000})
    ```

7. Interact with contract

    ```js
    > game.playMove('forward')
    ```

### Running a local development testnet

#### Running the first node

```shell
$ export EBAKUS_DATADIR=/tmp/eth/1
$ mkdir -p $EBAKUS_DATADIR && ebakus --datadir=$EBAKUS_DATADIR -verbosity 9 --
testnet --preload ~/ebakus/scripts/ebakus-load.js --nodiscover console 2>>
$EBAKUS_DATADIR/ebakus.log
```

#### Running the second node (optional)

```shell
$ export EBAKUS_DATADIR=/tmp/eth/2
$ mkdir -p $EBAKUS_DATADIR && ebakus --datadir=$EBAKUS_DATADIR -verbosity 9 --
testnet --preload ~/ebakus/scripts/ebakus-load.js --nodiscover --port 30404
console 2>> $EBAKUS_DATADIR/ebakus.log
```

The `~/ebakus/scripts/ebakus-load.js` script contains some quick access utility functions as can be the following:

```js
/*
 * Developer helpful functions for Ebakus node!
 */

/*
 * -- Helpful constants
 */
var DEV_ACCOUNTS = {
    producer: {
        address: "0xB2b3510C106E8e04Acfb9841e2213500167100f3",
        prv: "32883cebda1636ff41895b8425feee5622b074545e74db7091fa216d7ddb39d4",
        pass: "Ebakus123"
        // unlocked: false,
    },

    simple: {
        address: "0x8f10d3a6283672ecfaeea0377d460bded489ec44",
        prv: "90308da4eedf50472684b0b7bcd51b55bec149a4bc7135ccfc9c87ade7efc9b5",
        pass: "Ebakus123"
    }
};

/*
 * -------- Accounts -----------
 */

// Import an account from the predefined DEV_ACCOUNTS constant
function eAccountImport(account) {
    if (personal.listAccounts.indexOf(account.address.toLowerCase()) > -1) {
        return;
    }
    personal.importRawKey(account.prv, account.pass);
}

// Unlock account
function eAccountUnlock(account) {
    if (account.unlocked) {
        return true;
    }

    // store unlocked state in DEV_ACCOUNTS
    account.unlocked = true;

    console.log('Account "' + account.address + '" unlocked!');

    // unlock producer account
    return personal.unlockAccount(account.address, account.pass, 0);
}

// Unlock producer account
function eAccountProducerUnlock() {
    return eAccountUnlock(DEV_ACCOUNTS.producer);
}

// Print balances for all account
function eCheckAllBalances() {
    var totalBal = 0;
    for (var acctNum in eth.accounts) {
        var acct = eth.accounts[acctNum];
        var acctBal = web3.fromWei(eth.getBalance(acct), "ether");
        totalBal += parseFloat(acctBal);
        console.log(
            " eth.accounts[" +
                acctNum +
                "]: \t" +
                acct +
                " \tbalance: " +
                acctBal +
                " ether"
        );
    }
    console.log(" Total balance: " + totalBal + " ether");
}

/*
 * -------- Mining -----------
 */

// Start mining
function eMiner() {
    eAccountProducerUnlock();
    // start miner
    return miner.start();
}

/*
 * -------- Transactions -----------
 */

// Producer send ether to simple account
function eProducerSendEbakusToSimple(amount) {
    var producerAccount = DEV_ACCOUNTS.producer;
    eAccountUnlock(producerAccount);
    return eth.sendTransaction({
        from: producerAccount.address,
        to: DEV_ACCOUNTS.simple.address,
        value: web3.toWei(amount || 0.01, "ether")
    });
}
function _bootstrap() {
    console.info("JS Development helper has been loaded!");
    // import producer account, once
    eAccountImport(DEV_ACCOUNTS.producer);
    // import another account, once
    eAccountImport(DEV_ACCOUNTS.simple);
    // loadScript('~/ebakus/scripts/another-script.js');
}

// run on load
_bootstrap();
```
