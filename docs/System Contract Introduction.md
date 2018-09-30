# FISCO BCOS System Contract Introduction
**Author:fisco-dev**  

<!-- TOC -->

- [FISCO BCOS System Contract Introduction](#fisco-bcos-system-contract-introduction)
    - [Design Overview](#design-overview)
    - [How it works](#how-it-works)
        - [System Proxy](#system-proxy)
        - [Node Management](#node-management)
        - [CA Management](#ca-management)
        - [Permissions Management](#permissions-management)
        - [Configuration Management](#configuration-management)
    - [Customizations](#customizations)
        - [Example 1 - Custom Business Contract](#example-1---custom-business-contract)
        - [Example 2 - Custom Permission Contract](#example-2---custom-permission-contract)

<!-- /TOC -->

## Design Overview

To meet the requirements of access control, identity authentication, configuration management, permissions management and etc., the system contracts, which is a set of powerful, flexible and extendable smart contracts, will be deployed when initial the network.

The system contract is deployed by admin during initialization. All nodes' agreement is necessary for post-init upgrade.

FISCO BCOS system contract is composite of five modules: System Proxy, Node Management, CA Management, Permissions Management and Configuration Management. System contracts is extendable and can be called by both core system and DAPP. There could be one or more smart contracts in a module. The modules are below:

![module structure](./assets/systemcontract_module.png)

## How it works

Code path: systemcontractv2/. Brief each modules as below:

### System Proxy

SystemProxy.sol, the system proxy's implementation, provides a mapping between route and contract address, unified system contract interface. In SystemProxy.sol, routing info is hold by a mapping field named as '_routes'. The value of mapping is structured as below:

```python
struct SystemContract {
    address _addr;		#contract address
    bool _cache;		#cache flag
    uint _blocknumber;		#block height when the contract is active
}	
```

Key functions:

| function       | input parameters                                     | output parameters                                   | description                   |
| -------- | :--------------------------------------- | -------------------------------------- | -------------------- |
| getRoute | string key#route name                          | address#contract address<br>bool#catch flag<br>uint # block height | get route information               |
| setRoute | string key#route name<br>address addr#contract address<br>bool cache#cache flag<br>unit blocknumber #block height | N/A                                      | set route<br>overwrite if route name exists |



### Node Management

NodeAction.sol, the node management's implementation, provides nodes' registration, management and maintenance. Node joins or quits the chain must be controlled by node management contract.
Three node types: Core, Full, and Light.

```solidity
enum NodeType{
        None,
        Core,
        Full,
        Light
    }
```

Structure for node information:

```python
struct NodeInfo{
        string id;
        string ip;
        uint port;
        NodeType category;
        string desc;
        string CAhash;
        string agencyinfo;
        uint idx;
        uint blocknumber;       #block height
    }
```

Key functions:

| function           | input parameters                                     | output parameters       | description                  |
| ------------ | :--------------------------------------- | ---------- | ------------------- |
| registerNode | string _id<br>string _ip<br>uint _port<br>NodeType _category<br>string _desc<br>string _CAhash<br>string _agencyinfo<br>uint _idx | bool #result | register node<br>Ignore if the node exists |
| cancelNode   | string _id                   | bool #result | cancel node<br>Ignore if the node not exists  |



### CA Management

CAAction.sol, the CA management's implementation, provides nodes' certificate registration, management and maintenance. Node joins or quits the chain must controlled by CA management contract if certificate verification enabled.

Structure for certificate data:

```python
struct CaInfo{
        string  hash;		#certificate hash
        string pubkey;		#certificate public key
        string orgname;		#organization name
        uint notbefore;		#certificate effective date
        uint notafter;		#certificate expire date
        CaStatus status;	#certificate status
        string    whitelist;	#IP whitelist
        string    blacklist;	#IP blacklist
        uint    blocknumber;	#block height
      }
```

Key functions:

| function     | input parameters                                     | output parameters                                     | description                        |
| ------ | ---------------------------------------- | ---------------------------------------- | ------------------------- |
| update | string _hash<br>string _pubkey<br>string _orgname<br>uint _notbefore<br>uint _notafter<br>CaStatus _status<br>string _whitelist<br>string _blacklist | bool #result                               | update certificate<br>create certificate if certificate not exists |
| get    | string _hash                        | string#certificate hash<br>string#certificate public key<br>string#organization name<br>uint#certificate effective date<br>uint#certificate expire date<br>CaStatus#certificate status<br>uint##block height | get certificate information                    |



### Permissions Management

Permissions management's design principles: 1, One external account only belongs to one role. 2, One role only has one permission list. 3, Permission is identified by a combination of function and its contract address.

Permission module are composite of 4 contracts: [TransactionFilterChain.sol](https://github.com/FISCO-BCOS/FISCO-BCOS/blob/master/systemcontract/TransactionFilterChain.sol), [TransactionFilterBase.sol](https://github.com/FISCO-BCOS/FISCO-BCOS/blob/master/systemcontract/TransactionFilterBase.sol), [AuthorityFilter.sol](https://github.com/FISCO-BCOS/FISCO-BCOS/blob/master/systemcontract/AuthorityFilter.sol), [Group.sol](https://github.com/FISCO-BCOS/FISCO-BCOS/blob/master/systemcontract/Group.sol).

TransactionFilterChain.sol, the implementation of Filter pattern, provides an unified function - process - for permission checking. It holds an address list of Filter contract extends from TransactionFilterBase. All permissions will be checked by calling process function of each Filter contract in sequence.

A process method is mandatory for each filter which is extend from its base contract - TransactionFilterBase.sol
AuthorityFilter, inheriting TransactionFilterBase, checks user group's permission.

Group.sol handles the concept of Role. It defines a role by flag the permissions.

Key functions:

| contract                   | function            | input parameters                                     | output parameters      | description      |
| --------------------- | ------------- | ---------------------------------------- | --------- | ------- |
| TransactionFilterBase | process       | address origin #external address<br>address from#from account address<br>address to#to account address<br>string func#contract address<br>string input#transaction input| bool#result | permission checking    |
| Group                 | setPermission | address to#to account address<br>string func#contract address<br>bool permission#permission flag | bool#result | set permission |



### Configuration Management

ConfigAction.sol, the configuration management implementation, manages all configurable information. The configuration is consistent on all nodes by broadcasting transaction on chain, and only the admin can make configuration change.

Key functions:

| function   | input parameters                              | output parameters                     | description    |
| ---- | --------------------------------- | ------------------------ | ----- |
| set  | string key #parameter<br>string value#config information value | N/A                        | set configuration |
| get  | string key #parameter                   | string #config information<br> uint#block height | get configuration |

key parameters:

| parameter                  | description                           | default value       | recommend value         |
| -------------------- | ---------------------------- | --------- | ----------- |
| maxBlockHeadGas      | Gas spend limitation for each block (Hex)                | 200000000 | 20000000000 |
| intervalBlockTime    | an interval btw block generation(ms) (Hex)               | 1000      | 1000        |
| maxBlockTranscations | configure the max transaction in a block(Hex)                 | 1000      | 1000        |
| maxNonceCheckBlock   | Trace back max previous block number to avoid nonce duplication.(Hex)         | 1000      | 1000        |
| maxBlockLimit        | max delay for transaction commit(Hex) | 1000      | 1000        |
| maxTranscationGas    | Gas spend limitation for each transaction(Hex)               | 20000000  | 20000000    |
| CAVerify             | CA verification flag                       | FALSE     | FALSE       |

## Customizations

### Example 1 - Custom Business Contract

A case for the customized business, steps are below:

1. Implement 'set' and 'get' base on business requirement.
2. Deploy business contract and get contract address.
3. Call the 'setRoute' method in SystemProxy to register contract to route info.
4. Business smart contract is ready for calling.

how to call the business contract:

1. Call the 'getRoute' method in SystemProxy to get contract address.
2. Get configured information by calling the 'get' method with address in step 1.

### Example 2 - Custom Permission Contract

Permission checking can be extended by adding new Filter.

1. Create a Filter contract based on TransactionFilterBase. The custom permission checking should be put into the process method. 
2. Deploy custom permission contract and get contract address.
3. Call the 'getRoute' method in SystemProxy to get contract address of TransactionFilterChain.
4. Register custom filter contract by calling 'addFilter' method in TransactionFilterChain.
5. The contract is ready for calling.
