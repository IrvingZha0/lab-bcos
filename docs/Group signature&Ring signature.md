# Group signature & Ring signature

## Catalog
<!-- TOC -->
- [1 Introduction](#1-introduction)
    - [1.1 Case study](#11-case-study)
    - [1.2 Sample code](#12-sample-code)
- [2 Deployment](#2-deployment)
    - [2.1 Enable Group signature & Ring signature ethcall](#21-enable-group-signature--ring-signature-ethcall)
    - [2.2 Disable Group signature & Ring signature ethcall](#22-disable-group-signature--ring-signature-ethcall)
- [3 Usage](#3-usage)
- [4 Matters need attention](#4-matters-need-attention)

<!-- TOC -->

<br>

## 1 Introduction

### 1.1 Case study

**(1) Group signature cases**


- **Case 1**： Members (Consumer user) or subordinates organization in the organization on behalf of the organization put group signature information to the blockchain. When verifiers want to verify the group signature, they can only know which group the signature belongs but cannot know who is the signer. So group signature can keep signer anonymity and make sure the signature cannot be modified.
Group signature can be used for auctions, anonymous deposits and credit reports etc.
- **Case 2**：The business user sends the generated group signature to the consortium chain organization (such as webank) through AMOP, and the organization unified put the collected group signature information (such as bidding, reconciliation, etc.) to the blockchain. When verifiers want to verify the group signature, they cannot know who is the signer thus to keep signer anonymity. But the regulator can track the signer information through trusted third parties to ensure the traceability of the signature.

<br>

**(2) Ring signature cases**


- **Case 1 (Anonymous vote)**：Members (Consumer user) in the organization sign the vote and put the signature to the blockchain via the trusted agent(such as webank), When verifiers want to verify the ring signature, they can only know which agent the signature belongs but cannot know who is the signer/voter.

- **Other cases (Such as anonymous deposit, credit reports etc.)**：The cases are similar to group signature anonymous deposit and credit reports. The only difference is that NO ONE can track the identity of the signer.

- **Anonymous transaction**：In UTXO model, the ring signature algorithm can be used to anonymous transactions, and no one can track the sender and receiver of the transaction.

<br>


### 1.2  Sample code

Sample code for group signature & ring signature：

| <div align = left>Module</div>             | <div align = left>Code path</div>                                       | <div align = left>Description</div>                                       |
| -------------- | ---------------------------------------- | ---------------------------------------- |
| Dependency module       | scripts/install_pbc.sh<br>deploy_pbc.sh  | pbc and pbc-sig lib are used for group signature，install the pbc and pbc-sig by calling deploy_pbc.sh |
| source code for group signature & ring signature    | deps/src/group_sig_lib.tgz               | source code for group signature & ring signature                          |
| Compile Module           | cmake/FindPBC.cmake<br>cmake/ProjectGroupSig.cmake | compile cmake file related to group signature & ring signature   |
| Verification implement | libevm/ethcall/EthcallGroupSig.h<br>libevm/ethcall/EthcallRingSig.h | use ethcall to call the group/ring signature lib   |

FISCO BCOS supports configure whether enable the ethcall for group signature & ring signature(by default it is disabled). 
|   |   |   |   |
|---|---|---|---|
| enable ethcall  | dependencies will compile  |  long compile time | group/ring signature enabled  |
| disable ethall  | dependencies won't compile  |  short compile time | group/ring signature disabled  |

<br>

  
## 2 Deployment

Ensure FISCO BCOS deployed before ethcall deployment. ([How to deploy FISCO BCOS](https://github.com/FISCO-BCOS/FISCO-BCOS/tree/master/doc/manual) )

### 2.1 Enable Group signature & Ring signature ethcall


**(1) Install dependencies**


**① Install basic dependencies**


Install git, dos2unix and lsof before deploying FISCO BCOS：

- git: get the latest code
- dos2unix && lsof: uploaded file handling in linux during windows upload file to linux.

For centos and ubuntu OS can use below command lines to install these dependencies:
(If installation failed，check the configuration of yum and ubuntu)

```bash
[centos]
sudo yum -y install git
sudo yum -y install dos2unix
sudo yum -y install lsof

[ubuntu]
sudo apt install git
sudo apt install lsof
sudo apt install tofrodos
ln -s /usr/bin/todos /usr/bin/unxi2dos
ln -s /usr/bin/fromdos /usr/bin/dos2unix
```

**② Install pbc and pbc-sig for group signature**


Install pbc and pbc-sig before using ethcall for group signature，FISCO BCOS provides pbc and pbc-sig deploy script(both centos and ubuntu OS supported):

```bash
# Use dos2unix to format script, in case the Windows files upload to Linux cannot be parsed correctly
bash format.sh
# Use deploy_pbc.sh script to install pbc and pbc-sig
sudo bash deploy_pbc.sh
```
<br>

**(2) enable ethcall for group/ring signature**


FISCO BCOS group/ring signature use -DGROUPSIG=ON or -DGROUPSIG=OFF to control whether need compile, by default -DGROUPSIG=OFF.

```bash
# create build file for store the compiled file
cd FISCO-BCOS && mkdir -p build && cd build
# enable group/ring signature when cmake create Makefile
#**centos OS：
cmake3 -DGROUPSIG=ON -DEVMJIT=OFF -DTESTS=OFF -DMINIUPNPC=OFF ..
#**ubuntu OS:
cmake -DGROUPSIG=ON -DEVMJIT=OFF -DTESTS=OFF -DMINIUPNPC=OFF ..
```

<br>

**(3) Compile and start fisco bcos**

```bash
#编译fisco-bcos
make -j4 #注: 这里j4表明用4线程并发编译，可根据机器CPU配置调整并发线程数
#运行fisco-bcos: 替换节点启动的可执行文件，重启节点:
bash start.sh
```

<br>


### 2.2 Disable Group signature & Ring signature ethcall

**(1) 关闭群签名&&环签名ethcall编译开关**


编译fisco bcos时，将cmake的-DGROUPSIG编译选项设置为OFF可关闭群签名&&环签名ethcall功能：

```bash
#新建build文件用于存储编译文件
cd FISCO-BCOS && mkdir -p build && rm -rf build/* && cd build
#关闭群签名&&环签名ethcall开关
#centos系统：
cmake3 -DGROUPSIG=OFF -DEVMJIT=OFF -DTESTS=OFF -DMINIUPNPC=OFF ..
#ubuntu系统：
cmake -DGROUPSIG=OFF -DEVMJIT=OFF -DTESTS=OFF -DMINIUPNPC=OFF ..
```
<br>

**(2) 编译并运行fisco bcos**

```bash
#编译fisco-bcos
make -j4 #注: 这里j4表明用4线程并发编译，可根据机器CPU配置调整并发线程数
#运行fisco-bcos: 替换节点启动的可执行文件，重启节点:
bash start.sh
```

<br>



## 3 Usage

在区块链上使用群签名&&环签名功能，还需要部署以下服务：

**(1) 群签名&&环签名客户端： [sig-service-client](https://github.com/FISCO-BCOS/sig-service-client)**

sig-service-client客户端提供了以下功能：

- 访问[群签名&&环签名服务](https://github.com/FISCO-BCOS/sig-service)rpc接口；
- 将群签名&&环签名信息写入链上(发交易)

具体使用和部署方法请参考[群签名&&环签名客户端操作手册](https://github.com/FISCO-BCOS/FISCO-BCOS/tree/master/doc/manual).

<br>

**(2) 群签名&&环签名服务：[sig-service](https://github.com/FISCO-BCOS/sig-service)**


sig-service签名服务部署在机构内，为群签名&&环签名客户端([sig-service-client](https://github.com/FISCO-BCOS/sig-service-client))提供了如下功能：

- 群生成、群成员加入、生成群签名、群签名验证、追踪签名者身份等rpc接口；
- 环生成、生成环签名、环签名验证等rpc接口；

在FISCO BCOS中，sig-service-client客户端一般先请求该签名服务生成群签名或环签名，然后将获取的签名信息写入到区块链节点；区块链节点调用群签名&&环签名ethcall验证签名的有效性。

sig-service的使用和部署方法请参考[群签名&&环签名RPC服务操作手册](https://github.com/FISCO-BCOS/sig-service).

<br>



## 4 Matters need attention 

**(1) 群签名&& 环签名ethcall兼容老版本FISCO BCOS** 

<br>

(2) 启用群签名&&环签名ethcall，并调用相关功能后，**不能关闭该ethcall接口，否则验证区块时，群签名和环签名相关的数据无法找到验证接口，从而导致链异常退出**


若操作者在开启并使用群签名&&环签名特性后，不小心关闭了该ethcall功能，可通过回滚fisco bcos到开启群签名&&环签名ethcall时的版本来使链恢复正常；

<br>

**(3) 同一条链的fisco bcos版本必须相同**


即：若某个节点开启了群签名&&环签名验证功能，其他链必须也开启该功能，否则在同步链时，无群签名&&环签名ethcall实现的节点会异常退出；

<br>

(4) 使用群签名&&环签名链上验证功能前，**必须先部署[群签名&&环签名服务](https://github.com/FISCO-BCOS/sig-service)和[群签名&&环签名客户端](https://github.com/FISCO-BCOS/sig-service-client)**

[客户端sig-service-client](https://github.com/FISCO-BCOS/sig-service-client)向链上发签名信息，[群签名&&环签名服务](https://github.com/FISCO-BCOS/sig-service)为客户端提供签名生成服务

<br>

 