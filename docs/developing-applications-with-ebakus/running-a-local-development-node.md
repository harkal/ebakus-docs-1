# Running a local development node

While developing your own dApp it might be useful to run your own node. For doing this run:

```bash
docker run -ti --name ebakus-dev-node \
           -v ~/ebakus:/root \
           -p 30403:30403 \
           -p 8545:8545 \
           ebakus/go-ebakus \
           --dev \
           --rpc --rpcaddr 0.0.0.0 \
           --preload /root/ebakus-load.js \
           console
```

!!! tip
    Instead of the `--testnet` flag, the `--dev` one is used.

The `~/ebakus/ebakus-load.js` script contains some quick access utility functions as can be the following:

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
            "eth.accounts[" + acctNum + "]: \t" + acct + " \tbalance: " + acctBal + " ether"
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
