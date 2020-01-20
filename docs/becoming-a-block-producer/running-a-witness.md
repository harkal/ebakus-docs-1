# Becoming a block producer

For becoming a block producer you have to get a synced node with the network. To reach to that state, let's get the needed ingredients to start our node.

Let's set a folder in our system where we will store both our secrets and the data of our node.

```bash
export BASE_PATH=~/ebakus-producer

mkdir -p ${BASE_PATH}
```

!!! important
    This doc refers on how to become a block producer on the **mainnet** which is not launched yet. If you would like to experiment on **testnet** add the `--testnet` flag.

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

## Start the producer

We start our producer, having opened the `30403` port in the docker for connecting to the P2P network. We then set our `nodekey`, `etherbase` and `password` as well unlocking the account.

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

## Monitor syncing

Now we have to wait for the node to be synced. We can check the progress by attaching to the docker container like:

```bash
docker exec -it ebakus-producer ebakus attach
```

Inside the attached console we can check if we are currently syncing which has to return `false`.

```js
> eth.syncing
false
```

And also to check if the blockNumber is the latest one in our [explorer](https://explorer.ebakus.com/blocks) as well.

```js
> eth.blockNumber
```


## Elect as a witness

For electing ourselves as a witness we have to send a transaction on the main system contract `0x0000000000000000000000000000000000000101`. For doing so we need to attach to the docker container:

```bash
docker exec -it ebakus-producer ebakus attach
```

And send the transaction:

```js
> var tx = {
    from: eth.coinbase,
    to: '0x0000000000000000000000000000000000000101',
    data: '0xe0a1ea960000000000000000000000000000000000000000000000000000000000000001',
    nonce: eth.getTransactionCount(eth.coinbase)
};
> tx.gas = eth.estimateGas(tx);

> var txWithPow = eth.calculateWorkNonce(tx, eth.suggestDifficulty(eth.coinbase));

> eth.sendTransaction(txWithPow);
```

!!! info
    For more information on PoW check [here](../developing-applications-with-ebakus/proof-of-work.md).

## Start the producer

Once you are ready, set node active for producing.

```bash
docker exec -it ebakus-producer ebakus attach
```

```js
> miner.start()
true
```

!!! tip
    After the initial sync and being added as a witness you can add `--mine` flag in the docker startup script for auto mining. Also you ask docker to automatically restart when something happens `--restart unless-stopped` in the docker params.


## Be voted

In order to produce a block you need to be in the top 100 witnesses. For doing this help the community and ask others to vote you.

!!! tip
    Don't forget to vote for yourself!!! 😀
