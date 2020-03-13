# Becoming a block producer

For becoming a block producer you have to get a synced node with the network. To reach to that state, let's get the needed ingredients to start our node.

Let's set a folder in our system where we will store both our secrets and the data of our node.

```bash
export BASE_PATH=~/ebakus-producer

mkdir -p ${BASE_PATH}
```

!!! important
    This doc refers on how to become a block producer on the **mainnet**. If you would like to experiment on **testnet** add the `--testnet` flag.

## Create an account for the block producer

### Create a password file for your account

It's up to you to decide how to generate the password for your account. Here you can find an example command:

```bash
openssl rand -base64 32 > ${BASE_PATH}/passwd
```

### Create the account

```bash
docker run --rm --name ebakus-producer \
    -v ${BASE_PATH}:/root \
    -p 30403:30403 \
    -p 30403:30403/udp \
    ebakus/go-ebakus \
        account new \
            --password /root/passwd
```

!!! example "Example output"
    ```
    Your new key was generated

    Public address of the key:   0xA356eF85BB1740eC494B3c4eDA230aBd64D571F8
    Path of the secret key file: /root/.ebakus/keystore/UTC--2019-12-17T13-44-28.902899300Z--a356ef85bb1740ec494b3c4eda230abd64d571f8
    ```

From the above we need to note our public address. That is `0xA356eF85BB1740eC494B3c4eDA230aBd64D571F8` in this example.

## Starting the producer

We start our producer, having opened the `30403` port in the docker for connecting to the P2P network. We then set our `etherbase` and `password` as well unlocking the account.

```bash
docker run -d --name ebakus-producer \
    -v ${BASE_PATH}:/root \
    -p 30403:30403 \
    -p 30403:30403/udp \
    ebakus/go-ebakus \
        --etherbase 0xA356eF85BB1740eC494B3c4eDA230aBd64D571F8 \
        --unlock 0xA356eF85BB1740eC494B3c4eDA230aBd64D571F8 \
        --password /root/passwd
```

## Stopping the producer

It is important that you properly stop a running produser. Simply killing the ebakus process, or the docker container, will probably result in a corrupted state database that will require a complete resync of the node.

To avoid the constly resync, it is important to allow the producer to gracefully shutdown. We do this by sending it the SIGTERM signal. This is accomplished by the command below:

```bash
docker kill --signal=SIGTERM ebakus-producer
```

!!! info
    Stopping a producer will result in missed blocks. Read the `Step down as a witness` section in this guide on how to avoid this.

## Monitor syncing

Now we have to wait for the node to be synced. We can check the progress by attaching to the docker container like:

```bash
docker exec -it ebakus-producer ebakus attach
```

Inside the attached console we can check if we are currently syncing which has to return `false`.

```js
eth.syncing

// -- Output --
// false
```

And also to check if the blockNumber is the latest one in our [explorer](https://explorer.ebakus.com/blocks) as well.

```js
eth.blockNumber
```

## Control the producer

Once you are ready, set node active for producing.

```bash
docker exec -it ebakus-producer ebakus attach
```

```js
miner.start()

// -- Output --
// true
```

!!! tip
    After the initial sync and being added as a witness you can add `--mine` flag in the docker startup script for auto mining. Also you ask docker to automatically restart when something happens `--restart unless-stopped` in the docker params.


## Allow to be voted as a witness

For electing ourselves as a witness we have to send a transaction on the [system contract](/developing-applications-with-ebakus/system-contract) `0x0000000000000000000000000000000000000101`. For doing so we need to attach to the docker container:

```bash
docker exec -it ebakus-producer ebakus attach
```

To set "[electEnable](/developing-applications-with-ebakus/system-contract/#electenablebool)" to `true` you must again send a transaction to the system like this:

```js
var systemContractAddress = '0x0000000000000000000000000000000000000101';
var electEnableSystemContractABI = [{"inputs":[{"name":"enable","type":"bool"}],"name":"electEnable","outputs":[],"stateMutability":"nonpayable","type":"function"}];
var systemContract = eth.contract(electEnableSystemContractABI).at(systemContractAddress);

systemContract.electEnable(true);
```

??? info "Information on Proof of Work (PoW)"
    For more information on PoW check [here](/developing-applications-with-ebakus/proof-of-work.md).

## Step down as a witness

There are cases when you will need to step down from producing blocks. These include:

* Experiencing technical difficulties
* During software updates
* No longer wanting to participate in the Ebakus network

In these cases or any other when you will not be able/willing to produce blocks, it is important that you inform the network of your intention by setting "[electEnable](/developing-applications-with-ebakus/system-contract/#electenablebool)" to `false`. Users are monitoring the network for producers that miss blocks, and you don't want to be seen as a bad performing producer possibly resulting in lost votes. Setting "[electEnable](/developing-applications-with-ebakus/system-contract/#electenablebool)" to `false` will prevent you from losing blocks while you are experiencing any difficulty.

To set "[electEnable](/developing-applications-with-ebakus/system-contract/#electenablebool)" to `false` you must again send a transaction to the system like this:

```js
var systemContractAddress = '0x0000000000000000000000000000000000000101';
var electEnableSystemContractABI = [{"inputs":[{"name":"enable","type":"bool"}],"name":"electEnable","outputs":[],"stateMutability":"nonpayable","type":"function"}];
var systemContract = eth.contract(electEnableSystemContractABI).at(systemContractAddress);

systemContract.electEnable(false);
```

## Be voted

In order to produce a block you need to be in the top 100 witnesses. For attracting others to vote you, you can help the community and ask others to vote you.

!!! tip
    Don't forget to vote for yourself!!! 😀

    The easier way to vote is by visiting the [Ebakus explorer statistics](https://explorer.ebakus.com/statistics) and use the Ebakus wallet.

    Although, in case you want to vote yourself from within the go-ebakus node, here is some example code. This example will stake your whole balance at the moment running the command.

    **Stake whole balance**

    ```js
    var systemContractAddress = '0x0000000000000000000000000000000000000101';
    var stakeSystemContractABI = [{"inputs":[{"name":"amount","type":"uint64"}],"name":"stake","outputs":[],"stateMutability":"nonpayable","type":"function"}];
    var systemContract = eth.contract(stakeSystemContractABI).at(systemContractAddress);

    var balance = web3.fromWei(eth.getBalance(eth.coinbase));
    var stakeAmount = parseInt(balance * 10000);

    systemContract.stake(stakeAmount);
    ```

    **Vote yourself**

    ```js
    var systemContractAddress = '0x0000000000000000000000000000000000000101';
    var voteSystemContractABI = [{"inputs":[{"name":"addresses","type":"address[]"}],"name":"vote","outputs":[],"stateMutability":"nonpayable","type":"function"}];
    var systemContract = eth.contract(voteSystemContractABI).at(systemContractAddress);

    systemContract.vote([eth.coinbase]);
    ```
