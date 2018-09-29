# FISCO BCOS System Contract Introduction
**Author：fisco-dev**  

<!-- TOC -->

- [FISCO BCOS System Contract Introduction](#fisco-bcos-system-contract-introduction)
    - [Design Overview](#design-overview)
    - [Implementation Overview](#implementation-overview)
        - [System Proxy](#system-proxy)
        - [Node Management](#node-management)
        - [CA Management](#ca-management)
        - [Permissions Management](#permissions-management)
        - [Configuration Management](#configuration-management)
    - [Custom Scalability](#custom-scalability)
        - [Example 1 - Custom Business Configuration Contract](#example-1---custom-business-configuration-contract)
        - [Example 2 - Custom Business Permissions Filter Contract](#example-2---custom-business-permissions-filter-contract)

<!-- /TOC -->

## Design Overview

FISCO BCOS in order to meet the requirements of access control, identity authentication, configuration management, permissions management, etc., when initial the network startup, a set of powerful, flexible and supporting custom scalability smart contract will be deployed, collectively referred to as system contracts.

In principle, the system contract is implemented by the blockchain administrator during initial the network startup. If redeploy the upgrade during the operating period, then the administrator needs get all nodes agreement to perform the operation.

The current FISCO BCOS system contract has five main modules, System Proxy module, Node Management module, CA Management module, Permissions Management module and Configuration Management module. System contracts can be expanded as needed, It can be used for both blockchain core and DAPP services. Each module is implemented by one or more smart contracts. The structure is as follows:

![module structure](./assets/systemcontract_module.png)

## Implementation Overview

The current FISCO BCOS system contract has a corresponding contract implementation for the System Proxy module, Node Management module, CA Management module, Permissions Management module and Configuration Management module. The contract source code path is: systemcontractv2/.

### System Proxy

SystemProxy.sol is the implementation of system proxy module. It implements a naming service that routes to the contract address and provides a unified entrace for system contract. The internal implementation is that all routing table information is maintained by the mapping type member variable _routes. The routing table data structure:

```python
struct SystemContract {
    address _addr;		#contract adress
    bool _cache;		#cache flag
    uint _blocknumber;		#block height
}	
```

Main interface：

| interface       | input parameters                                     | output parameters                                   | description                   |
| -------- | :--------------------------------------- | -------------------------------------- | -------------------- |
| getRoute | string key#route name                          | address, bool,uint # contract address, cache flag, block height | get route information               |
| setRoute | string key, address addr, bool cache# route name, contract address, cache flag, block height | N/A                                      | set route information, overwrite route name if it exists |



### Node Management

NodeAction.sol is the implementation of node management module.It implements registration, management and maintenance for all nodes. When a node in the network joins or quits, it must interact with the node management contract.
In FISCO BCOS, nodes are divided into three types: core nodes, full nodes, and light nodes.

```
enum NodeType{
        None,
        Core,
        Full,
        Light
    }
```

The node information data structure：

```python
struct NodeInfo{
        string id;   		#ID    
        string ip;   		#IP       
        uint port;   		#port
        NodeType category;	#category
        string desc;    	#description 
        string CAhash;  	#Node certificate hash
        string agencyinfo;      #other information
        uint idx;		#index
        uint blocknumber;       #block height
    }
```

Main interface：

| interface           | input parameters                                     | output parameters       | description                  |
| ------------ | :--------------------------------------- | ---------- | ------------------- |
| registerNode | string _id,string _ip,uint _port,NodeType _category,string _desc,string _CAhash,string _agencyinfo,uint _idx #节点身份ID、IP、端口、节点类型、节点描述、节点CA哈希、节点agency、节点序号 | bool #注册结果 | 注册节点 , 若该节点信息已存在, 则忽略 |
| cancelNode   | string _id #节点身份ID                       | bool #注册结果 | 注销节点, 若该节点信息不存在, 则忽略  |



### CA Management

CAAction.sol是机构证书模块的实现合约.它实现了对网络中所有节点的机构证书信息的登记、管理、维护功能.当网络启用机构证书验证功能的情况下, 网络中节点加入或退出都需要与机构证书合约进行交互.

机构证书的数据结构是：

```
struct CaInfo{
        string  hash;		#节点机构证书哈希
        string pubkey;		#证书公钥
        string orgname;		#机构名称
        uint notbefore;		#证书启用日期
        uint notafter;		#证书失效时间
        CaStatus status;	#证书状态
        string    whitelist;#IP白名单
        string    blacklist;#IP黑名单
        uint    blocknumber;#block height
      }
```

Main interface：

| interface     | input parameters                                     | output parameters                                     | description                        |
| ------ | ---------------------------------------- | ---------------------------------------- | ------------------------- |
| update | string _hash,string _pubkey,string _orgname,uint _notbefore,uint _notafter,CaStatus _status,string _whitelist,string _blacklist # 证书哈希、证书公钥、机构名称、 证书启用日期、 证书失效时间、证书状态、IP白名单、IP黑名单 | bool #更新结果                               | 更新证书信息,  若该证书信息不存在, 则新建证书记录 |
| get    | string _hash#证书哈希                        | string,string,string,uint,uint,CaStatus,uint#  证书哈希、证书公钥、机构名称、证书启用日期、证书失效时间、证书状态、生效区块块号 | 查询证书信息                    |



### Permissions Management

FISCO BCOS基于角色的身份权限设计有三要点：一个外部账户只属于一个角色；一个角色拥有一个权限项列表； 一个权限项由contract address加上合约interface来唯一标识.

当前FISCO BCOS权限管理模块主要由TransactionFilterChain.sol、TransactionFilterBase.sol、AuthorityFilter.sol、Group.sol四个合约来实现.

TransactionFilterChain是对Filter模型的实现框架.它在内部维护了一个实现继承于TransactionFilterBase的Filtercontract address列表.它对区块链核心提供了统一的权限检查interfaceprocess.process执行过程中会对Filtercontract address列表中的所有Filter依次执行process函数, 以完成所有需要的权限检查.

TransactionFilterBase是Filter的基类合约.所有Filter必须要实现它的processinterface.AuthorityFilter是继承于TransactionFilterBase的角色权限Filter实现.它的processinterface实现了对用户所属角色组的权限项进行检查逻辑.

Group是对角色的实现.它内部维护了该角色的所有权限项的mapping标志位.

Main interface：

| 合约                    | interface            | input parameters                                     | output parameters      | description      |
| --------------------- | ------------- | ---------------------------------------- | --------- | ------- |
| TransactionFilterBase | process       | address origin, address from, address to, string func, string input# 用户外部账户、交易发起账户、contract address、合约interface、交易输入数据 | bool#处理结果 | 权限检查    |
| Group                 | setPermission | address to, string func, bool perrmission# contract address、合约interface、权限标记 | bool#处理结果 | 设置角色权限项 |



### Configuration Management

ConfigAction.sol是全网配置模块的实现合约.它维护了FISCO BCOS区块链中全网运行的可配置信息. 配置信息可以通过交易广播上链来达到全网配置的一致性更新.原则上只能由区块链管理员来发出全网配置更新交易.

ConfigAction.sol的内部实现中维护了配置项信息的mapping 成员变量.

Main interface：

| interface   | input parameters                              | output parameters                     | description    |
| ---- | --------------------------------- | ------------------------ | ----- |
| set  | string key, string value# 配置项、配置值 | 无                        | 设置配置项 |
| get  | string key #配置项                   | string, uint# 配置值、block height | 查询配置值 |

当前FISCO BCOS主要有以下全网配置项：

| 配置项                  | description                           | 默认值       | 推荐值         |
| -------------------- | ---------------------------- | --------- | ----------- |
| maxBlockHeadGas      | 块最大GAS （16进制）                | 200000000 | 20000000000 |
| intervalBlockTime    | 块间隔(ms) （16进制）               | 1000      | 1000        |
| maxBlockTranscations | 块最大交易数（16进制）                 | 1000      | 1000        |
| maxNonceCheckBlock   | 交易nonce检查最大块范围（16进制）         | 1000      | 1000        |
| maxBlockLimit        | blockLimit超过当前块号的偏移最大值（16进制） | 1000      | 1000        |
| maxTranscationGas    | 交易的最大gas（16进制）               | 20000000  | 20000000    |
| CAVerify             | CA验证开关                       | FALSE     | FALSE       |

## Custom Scalability

### Example 1 - Custom Business Configuration Contract

假设业务需要利用系统合约框架, 自定义业务配置合约以对业务相关合约提供配置服务.大体可以参考以下步骤来扩展：

1. 根据业务合约需求, 实现业务配置合约的设置配置项interfaceset和查询配置值interfaceget.
2. 部署业务配置合约, 获得业务配置合约链上地址.
3. 调用System ProxySystemProxy的setRouteinterface, 将业务配置contract address注册到路由表中.
4. 至此, 业务配置合约已经完成在System Proxy的路由注册, 已可在业务交易中调用.

业务配置合约的使用方法：

1. 调用SystemProxy的getRouteinterface运行时获得业务配置contract address.
2. 通过业务配置contract address调用查询配置值interfaceget获得配置值.

### Example 2 - Custom Business Permissions Filter Contract

假设业务需要增加业务权限校验逻辑, 则可以利用权限管理合约的Filter机制来无缝扩展.大体可以参考以下步骤来扩展：

1. 继承于TransactionFilterBase实现一个业务权限Filter合约, 业务权限Filter合约根据业务需要的权限校验逻辑实现processinterface.
2. 部署业务权限Filter合约, 获得对应的contract address.
3. 调用System ProxySystemProxy的getRouteinterface, 获得TransactionFilterChaincontract address.
4. 调用TransactionFilterChain合约的addFilterinterface, 将业务权限Filtercontract address注册到Filter合约列表中.
5. 至此, 业务权限Filter合约已经启用.


