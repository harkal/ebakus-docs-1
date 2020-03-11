The system contract adds support for fundamental actions needed by the system as a native contract. This contract supports actions like stake/unstake, vote/unvote, as well storing/retrieving the ABI for developers owned contracts.

**System contract address**: `0x0000000000000000000000000000000000000101`.

In order to use this contract, first and foremost you have to know the systems' contract ABI. You can get this:

- either copy/paste it from [here](https://explorer-api.ebakus.com/abi/0x0000000000000000000000000000000000000101){:target="_blank" rel="noopener"}

- by calling the `getAbiForAddress` on the system contract itself with code similar to this [gist](https://gist.github.com/ziogaschr/61c4d3ba3b1e47f10164a296e3222511#file-web3-ebakus-examples-html-L24-L61){:target="_blank" rel="noopener"}

- by fetching it from our Explorer API
    ```sh
    curl https://explorer-api.ebakus.com/abi/0x0000000000000000000000000000000000000101
    ```

!!! example "Example on how to use the system contract"
    ```js tab="Web3.js"
    var systemContractAddress = '0x0000000000000000000000000000000000000101';
    var stakeSystemContractABI = [{ "inputs": [{ "name": "amount", "type": "uint64" }], "name": "stake", "outputs": [], "stateMutability": "nonpayable", "type": "function" }];
    var systemContract = new web3.eth.Contract(stakeSystemContractABI, systemContractAddress)

    var account = '0x...';
    var stakeAmount = 3 * 10000; // 3 EBK

    var tx = {
        from: account,
        to: systemContractAddress,
        data: systemContract.methods.stake(stakeAmount).encodeABI()
    }

    web3.eth.estimateGas(tx)
        .then(function (gas) {
            tx.gas = gas
            return web3.eth.suggestDifficulty(account)
        })
        .then(function (difficulty) {
            return web3.eth.calculateWorkForTransaction(tx, difficulty)
        })
        .then(function (txWithPow) {
            return web3.eth.sendTransaction(txWithPow)
        })
        .then(function (receipt) {
            console.log('Tx receipt', receipt)
        })


    // or
    // systemContract.methods.stake(stakeAmount).send({from: account})
    //    .then(function (receipt) {
    //        console.log('Tx receipt', receipt)
    //    })
    ```

    ```js tab="go-ebakus console"
    var systemContractAddress = '0x0000000000000000000000000000000000000101';
    var stakeSystemContractABI = [{"inputs":[{"name":"amount","type":"uint64"}],"name":"stake","outputs":[],"stateMutability":"nonpayable","type":"function"}];
    var systemContract = eth.contract(stakeSystemContractABI).at(systemContractAddress);

    var balance = web3.fromWei(eth.getBalance(eth.coinbase));
    var stakeAmount = parseInt(balance * 10000);

    systemContract.stake(stakeAmount);
    ```

## Methods documentation

### stake(uint64)

Stakes EBK amount for account, which can be used for voting and for needing less Proof of Work for sending a transaction to the network. Staked amount remains to the user account and is not being taken by the system. At any point user can unstake it and use it.

!!! tip "Voting"
    With regards voting, all voted producers (up to 20) will receive the whole EBK stake evenly.

!!! tip
    While staking and in case user has EBK pending for unstaking, then the system contract will first use the unstaking EBK before using the liquid amount from your wallet.

### getStaked() uint64

Retrieve the staked amount for current user.

### unstake(uint64)

Unstake existing staked EBK amount for account. Unstaking will hold the EBK tokens from being available to the account for **3 days** until they mature.

!!! tip "Voting"
    If wallet has voted before and unstakes the whole amount, votes will not be removed, but will only be disabled. The next time wallet will stake EBK, votes will be re-enabled.

### claim()

For unstaked EBK and after they mature within **3 days**, user has to call claim method in order EBK are available in the wallet as liquid balance.

### vote(address[])

In order for the system to operate normally and be performant, users has to vote the most trustful and powerfull producers. For doing so, vote has to be used. It accepts and array of account addresses. A wallet can vote up to 20 producers.

!!! Requirement
    Account must have staked amount.

!!! tip
    Every time vote gets called, it has to set all the voted addresses and not only the new ones.

### unvote()

This command will unvote all the producers already voted.

### electEnable(bool)

When a witness wants to set its node active for producing, she has call the electEnable command in order to be active. It's very important that the producer remains online and is synced the whole time.

!!! danger
    If the producer has issues or wants to go in maintenance mode, she has to call `electEnable(false) beforehand. Once finished, she can enable the producer again without losing the ranking.

### storeAbiForAddress(address,string)

For the ecosystem to work better (explorers, wallets) and so as anyone can use the deployed contracts, other developers has to know the contract ABI. For this reason the system contract allows the developer to upload the contract ABI to the chain. It is suggested to happen immediatelly after the contract deploy.

!!! warning
    This action can happen only once. The ABI of a contract can't be changed once uploaded.

### getAbiForAddress(address) string

In order to be able to call a contract, its ABI is needed, for this reason Ebakus allows to fetch a contract ABI, if available. The response will be a JSON string.
