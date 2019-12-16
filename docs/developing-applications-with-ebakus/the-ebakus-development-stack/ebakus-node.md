## Go Ebakus

Official golang implementation of the Ebakus protocol.

[![API Reference](
https://camo.githubusercontent.com/915b7be44ada53c290eb157634330494ebe3e30a/68747470733a2f2f676f646f632e6f72672f6769746875622e636f6d2f676f6c616e672f6764646f3f7374617475732e737667
)](https://godoc.org/github.com/ebakus/go-ebakus)
[![Go Report Card](https://goreportcard.com/badge/github.com/ebakus/go-ebakus)](https://goreportcard.com/report/github.com/ebakus/go-ebakus)

Node repo: **ebakus/go-ebakus** [![GitHub stars](https://img.shields.io/github/stars/ebakus/go-ebakus.svg?style=social&label=Star&maxAge=2592000)](https://GitHub.com/ebakus/go-ebakus)

## Running ebakus node


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

!!!Note
    If you are running an Ebakus client inside a Docker container, you should mount a data volume as the client's data directory (located at `/root/.ebakus` inside the container) to ensure that downloaded data is preserved between restarts and/or container life-cycles.

### Full node on the main Ebakus network

By far the most common scenario is people wanting to simply interact with the Ebakus network: create accounts; transfer funds; deploy and interact with contracts.

To do so:

```shell
ebakus console
```

This command will:

* Start `ebakus` in sync mode.
* Start up `ebakus` built-in interactive JavaScript console, (via the trailing `console` subcommand) through which you can invoke all official `web3` methods as well as Ebakus' own management APIs.
 This too is optional and if you leave it out you can always attach to an already running `ebakus` instance with `ebakus attach`.

### Full node on the Ebakus test network

Transitioning towards developers, if you'd like to play around with creating Ebakus contracts, you almost certainly would like to do that without any real money involved until you get the hang of the entire system.
In other words, instead of attaching to the main network, you want to join the **test** network with your node, which is fully equivalent to the main network, but with play-EBK only.

```shell
ebakus --testnet console
```

The `console` subcommand has the exact same meaning as above and they are equally
useful on the testnet too. Please see above for their explanations if you've skipped here.

Specifying the `--testnet` flag, however, will reconfigure your `ebakus` instance a bit:

* Instead of using the default data directory (`~/.ebakus` on Linux for example), `ebakus` will nest itself one level deeper into a `testnet` subfolder (`~/.ebakus/testnet` on Linux).

    !!!note
        On OSX and Linux this also means that attaching to a running testnet node requires the use of a custom endpoint since `ebakus attach` will try to attach to a production node endpoint by default.
        E.g. `ebakus attach <datadir>/testnet/ebakus.ipc`. Windows users are not affected by this.

* Instead of connecting the main Ebakus network, the client will connect to the test network, which uses different P2P bootnodes, different network IDs and genesis states.

    !!!note
        Although there are some internal protective measures to prevent transactions from crossing over between the main network and test network, you should make sure to always use separate accounts for play-money and real-money.
        Unless you manually move accounts, `ebakus` will by default correctly separate the two networks and will not make any accounts available between them.


### Programatically interfacing ebakus nodes

As a developer, sooner rather than later you'll want to start interacting with `ebakus` and the Ebakus network via your own programs and not manually through the console. To aid this, `ebakus` has built-in support for a JSON-RPC based APIs.
These can be exposed via HTTP, WebSockets and IPC (UNIX sockets on UNIX based
platforms, and named pipes on Windows).

The IPC interface is enabled by default and exposes all the APIs supported by `ebakus`, whereas the HTTP and WS interfaces need to manually be enabled and only expose a subset of APIs due to security reasons.
These can be turned on/off and configured as you'd expect.

HTTP based JSON-RPC API options:

  * `--rpc` Enable the HTTP-RPC server
  * `--rpcaddr` HTTP-RPC server listening interface (default: `localhost`)
  * `--rpcport` HTTP-RPC server listening port (default: `8545`)
  * `--rpcapi` API's offered over the HTTP-RPC interface (default: `eth,net,web3`)
  * `--rpccorsdomain` Comma separated list of domains from which to accept cross origin requests (browser enforced)
  * `--ws` Enable the WS-RPC server
  * `--wsaddr` WS-RPC server listening interface (default: `localhost`)
  * `--wsport` WS-RPC server listening port (default: `8546`)
  * `--wsapi` API's offered over the WS-RPC interface (default: `eth,net,web3`)
  * `--wsorigins` Origins from which to accept websockets requests
  * `--ipcdisable` Disable the IPC-RPC server
  * `--ipcapi` API's offered over the IPC-RPC interface (default: `admin,debug,eth,miner,net,personal,shh,txpool,web3`)
  * `--ipcpath` Filename for IPC socket/pipe within the datadir (explicit paths escape it)

You'll need to use your own programming environments' capabilities (libraries, tools, etc) to connect via HTTP, WS or IPC to a `ebakus` node configured with the above flags and you'll need to speak [JSON-RPC](https://www.jsonrpc.org/specification) on all transports.
You can reuse the same connection for multiple requests!

!!! tip
    Please understand the security implications of opening up an HTTP/WS based transport before doing so! Hackers on the internet are actively trying to subvert Ebakus nodes with exposed APIs! Further, all browser tabs can access locally running web servers, so malicious web pages could try to subvert locally available APIs!

!!! example
    Here is an example command for starting up a node on testnet with opened RPC/WS transports

    ```shell
    docker run -d --name ebakus-testnet-node \
                -v ~/ebakus:/root \
                -p 30403:30403 \
                -p 8545:8545 \
                -p 8546:8546 \
                ebakus/go-ebakus \
                --testnet \
                --rpc --rpcaddr 0.0.0.0 \
                --ws --wsaddr 0.0.0.0
    ```

    if you need to attach to the node you can simply use:

    ```shell
    docker exec -it ebakus-testnet-node /usr/local/bin/ebakus --testnet attach
    ```
