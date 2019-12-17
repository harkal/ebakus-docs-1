# Ebakus node - Go Ebakus [![GitHub stars](https://img.shields.io/github/stars/ebakus/go-ebakus.svg?style=social&label=ebakus/go-ebakus&maxAge=2592000)](https://github.com/ebakus/go-ebakus)

Official Ebakus protocol implementation in Go.

[![API Reference](
https://camo.githubusercontent.com/915b7be44ada53c290eb157634330494ebe3e30a/68747470733a2f2f676f646f632e6f72672f6769746875622e636f6d2f676f6c616e672f6764646f3f7374617475732e737667
)](https://godoc.org/github.com/ebakus/go-ebakus)
[![Go Report Card](https://goreportcard.com/badge/github.com/ebakus/go-ebakus)](https://goreportcard.com/report/github.com/ebakus/go-ebakus)

## Running ebakus node

By far the most common scenario is people wanting to simply interact with the Ebakus network: create accounts; transfer funds; deploy and interact with contracts.

One of the quickest ways to get Ebakus up and running on your machine is through the Docker images of the ebakus node.

- `ebakus/go-ebakus:latest` is the latest development version of Ebakus (default)
- `ebakus/go-ebakus:{version}` is the stable version of Ebakus at a specific version number

To pull an image and start a node, run these commands:

```shell
docker run -d --name ebakus-node \
        -v ~/ebakus:/root \
        -p 30403:30403 \
        ebakus/go-ebakus
```

It will create a persistent volume in your home directory (~/ebakus) for
saving your blockchain.

Do not forget `--rpc --rpcaddr 0.0.0.0`, if you want to access RPC from other containers
and/or hosts. By default, `ebakus` binds to the local interface and RPC endpoints is not accessible from the outside.

The image has the following ports automatically exposed:

- `8545` TCP, used by the HTTP based JSON RPC API
- `8546` TCP, used by the WebSocket based JSON RPC API
- `8547` TCP, used by the GraphQL API
- `30403` TCP and UDP, used by the P2P protocol running the network

!!!note
    If you are running an Ebakus client inside a Docker container, you should mount a data volume as the client's data directory (located at `/root/.ebakus` inside the container) to ensure that downloaded data is preserved between restarts and/or container life-cycles.

### Access the ebakus node running in docker container

Now that the ebakus node is up and running, and in order to get access to its built-in interactive JavaScript console we have to run:

```shell
docker exec -it ebakus-node /usr/local/bin/ebakus --testnet attach
```

!!!tip "Built-in interactive JavaScript console"
    Within the console you can invoke all official `web3` methods as well as Ebakus' own management APIs.

### Sync with the Ebakus test network

Transitioning towards developers, if you'd like to play around with creating Ebakus contracts, you almost certainly would like to do that without any real money involved until you get the hang of the entire system.
In other words, instead of attaching to the main network, you want to join the **test** network with your node, which is fully equivalent to the main network, but with play-EBK only.

In order to achive this we have to add `--testnet` flag after our `ebakus/go-ebakus` in the [start up command](#running-ebakus-node). Like so:

```shell
docker run -d --name ebakus-node \
           -v ~/ebakus:/root \
           -p 30403:30403 \
           ebakus/go-ebakus \
           --testnet
```

!!!tip
    Don't forget to add the `--testnet` flag in the `attach` command also.

!!!note
    Although there are some internal protective measures to prevent transactions from crossing over between the main network and test network, you should make sure to always use separate accounts for play-money and real-money.
    Unless you manually move accounts, `ebakus` will by default correctly separate the two networks and will not make any accounts available between them.


### Programatically interfacing ebakus nodes

As a developer, sooner rather than later you'll want to start interacting with `ebakus` and the Ebakus network via your own programs and not manually through the console. To aid this, `ebakus` has built-in support for a JSON-RPC based APIs.
These can be exposed via HTTP, WebSockets and IPC (UNIX sockets on UNIX based
platforms, and named pipes on Windows).

The IPC interface is enabled by default and exposes all the APIs supported by `ebakus`, whereas the HTTP,WS and GraphQL interfaces need to manually be enabled and only expose a subset of APIs due to security reasons.
These can be turned on/off and configured as you'd expect.

HTTP based JSON-RPC API options:

* `--rpc` Enable the HTTP-RPC server
* `--rpcaddr` HTTP-RPC server listening interface (default: `localhost`)
* `--rpcport` HTTP-RPC server listening port (default: `8545`)
* `--rpcapi` API's offered over the HTTP-RPC interface (default: `db,dpos,eth,net,web3`)
* `--rpccorsdomain` Comma separated list of domains from which to accept cross origin requests (browser enforced)
* `--rpcvhosts` Comma separated list of virtual hostnames from which to accept requests (server enforced). Accepts `*` wildcard. (default: `localhost`)
* `--ws` Enable the WS-RPC server
* `--wsaddr` WS-RPC server listening interface (default: `localhost`)
* `--wsport` WS-RPC server listening port (default: `8546`)
* `--wsapi` API's offered over the WS-RPC interface (default: `db,dpos,eth,net,web3`)
* `--wsorigins` Origins from which to accept websockets requests
* `--ipcdisable` Disable the IPC-RPC server
* `--ipcapi` API's offered over the IPC-RPC interface (default: `admin,db,debug,dpos,eth,miner,net,personal,shh,txpool,web3`)
* `--ipcpath` Filename for IPC socket/pipe within the datadir (explicit paths escape it)
* `--graphql` Enable the GraphQL server
* `--graphql.addr` GraphQL server listening interface (default: `localhost`)
* `--graphql.port` GraphQL server listening port (default: `8547`)
* `--graphql.corsdomain` Comma separated list of domains from which to accept cross origin requests (browser enforced)
* `--graphql.vhosts` Comma separated list of virtual hostnames from which to accept requests (server enforced). Accepts `*` wildcard. (default: `localhost`)

You'll need to use your own programming environments' capabilities (libraries, tools, etc) to connect via HTTP, WS or IPC to a `ebakus` node configured with the above flags and you'll need to speak [JSON-RPC](https://www.jsonrpc.org/specification) on all transports.
You can reuse the same connection for multiple requests!

!!! tip
    Please understand the security implications of opening up an HTTP/WS/GraphQL based transport before doing so! Hackers on the internet are actively trying to subvert Ebakus nodes with exposed APIs! Further, all browser tabs can access locally running web servers, so malicious web pages could try to subvert locally available APIs!

!!! example
    Here is an example command for starting up a node on testnet with opened RPC/WS transports and attached to the internal node console.

    ```shell
    docker run -ti --name ebakus-testnet-node \
               -v ~/ebakus:/root \
               -p 30403:30403 \
               -p 8545:8545 \
               -p 8546:8546 \
               ebakus/go-ebakus \
               --testnet \
               --rpc --rpcaddr 0.0.0.0 \
               --ws --wsaddr 0.0.0.0 \
               console
    ```
