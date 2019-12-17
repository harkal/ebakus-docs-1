# Using ebakusDB in Solidity

## Introduction

Each smart contract in ebakus has its own schema defined database (ESDD). This database can support any number of tables with typed fields and indexes. A smart contract is able to perform the following operations on the data:

1. Create/Drop tables
2. Create/Drop indexes on specific fields
3. Retrieve/update/delete single or multiple rows of data
4. Do ordered range queries on these data

The ebakus software makes sure that the data are stored in such a way in order to support the above operations in the most efficient way. The smart contract should not need to implement most common query types by itself.

The EbakusDB layer is providing to the ebakus blockchain a very fast database layer that supports O(1) time and space complexity snapshots. This is essential to the operation of a blockchain system that has requirements for querying old block states. The database achieves high performance by being aware of the transactional log functionality that the layer above it is using and not reimplementing it itself. Therefore achieving ACID compliance without sacrificing performance.

Smart contracts deployed in Ethereum compatibility mode will not be able to make use of the ESDD, hence will not be able to benefit from the extra functionality and performance.

## How to install

### Truffle Installation

First install truffle via npm using `npm install -g truffle`.

> **version 5.0.0**, at the time of writting this page

Please [visit Truffle's installation guide](https://truffleframework.com/docs/truffle/getting-started/installation "Truffle installation guide") for further information and requirements.

### EbakusDB library linking

This process will allow you to both link your contract to the current on-chain library as well as deploy it in your local environment for development.

1. Place the [EbakusDB.sol](https://github.com/ebakus/ebakusdb-solidity/blob/master/EbakusDB.sol) file in your truffle `contracts/` directory.
2. Place the [EbakusDB.json](https://github.com/ebakus/ebakusdb-solidity/blob/master/EbakusDB.json) file in your truffle `build/contracts/` directory.
3. Amend the `deployment.js` file in your truffle `migrations/` directory as follows:

```js
var EbakusDB = artifacts.require('EbakusDB');
var YourContract = artifacts.require("./YourContract.sol");

module.exports = function(deployer) {
    deployer.deploy(EbakusDB, {overwrite: false});
    deployer.link(EbakusDB, YourContract);
    deployer.deploy(YourContract);
};
```

!!! note
    The `.link()` function should be called _before_ you `.deploy(YourStandardTokenContract)`.
    Also, be sure to include the `{overwrite: false}` when writing the deployer i.e. `.deploy(EbakusDB, {overwrite: false})`.
    This prevents deploying the library onto the main network at your cost and uses the library already on the blockchain. The function should still be called however because it allows you to use it in your development environment.

## Usage Example

You can read about available EbakusDB methods and their documentation inline [here](https://github.com/ebakus/ebakusdb-solidity/blob/master/truffle/contracts/EbakusDB.sol).
You can find an example contract using the EbakusDB [here](https://github.com/ebakus/ebakusdb-solidity/blob/master/truffle/contracts/examples/Example.sol).

```solidity
pragma solidity ^0.5.0;

import "./EbakusDB.sol";

contract Example {
    string TableName = "Users";

    struct User {
        uint64 Id;
        string Name;
        string Pass;
    }

    constructor() public {
        string memory tablesAbi = '[{"type":"table","name":"Users","inputs":[{"name":"Id","type":"uint64"},{"name":"Name","type":"string"},{"name":"Pass","type":"string"}]}]';

        EbakusDB.createTable(TableName, "Name", tablesAbi);
    }

    function main() external {
        // Insert entry
        User memory u = User(1, "Harry", "123");
        bytes memory input = abi.encode(u.Id, u.Name, u.Pass);
        EbakusDB.insertObj(TableName, input);

        // Get back entry
        User memory u1;
        bytes memory out = EbakusDB.get(TableName, "Name", "Harry");
        (u1.Id, u1.Name, u1.Pass) = abi.decode(out, (uint64, string, string));
    }

    // more code
}
```
