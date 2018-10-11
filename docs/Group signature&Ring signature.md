# Group signature & Ring signature
**Author: fisco-dev**  

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
- **Case 2**：The business user sends the generated group signature to the consortium blockchain organization (such as webank) through AMOP, and the organization unified put the collected group signature information (such as bidding, reconciliation, etc.) to the blockchain. When verifiers want to verify the group signature, they cannot know who is the signer thus to keep signer anonymity. But the regulator can track the signer information through trusted third parties to ensure the traceability of the signature.

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

**(2) Enable/disable ethcall for group/ring signature**


FISCO BCOS group/ring signature use -DGROUPSIG=ON or -DGROUPSIG=OFF to control whether need compile, by default -DGROUPSIG=OFF.

Enable:
```bash
# create build file for store the compiled file
cd FISCO-BCOS && mkdir -p build && cd build
# enable group/ring signature when cmake create Makefile
#**centos OS：
cmake3 -DGROUPSIG=ON -DEVMJIT=OFF -DTESTS=OFF -DMINIUPNPC=OFF ..
#**ubuntu OS:
cmake -DGROUPSIG=ON -DEVMJIT=OFF -DTESTS=OFF -DMINIUPNPC=OFF ..
```
Disable:
```bash
# create build file for store the compiled file
cd FISCO-BCOS && mkdir -p build && rm -rf build/* && cd build
# disable group/ring signature when cmake create Makefile
#centos OS：
cmake3 -DGROUPSIG=OFF -DEVMJIT=OFF -DTESTS=OFF -DMINIUPNPC=OFF ..
#ubuntu OS：
cmake -DGROUPSIG=OFF -DEVMJIT=OFF -DTESTS=OFF -DMINIUPNPC=OFF ..
```
<br>

<br>

**(3) Compile and start fisco bcos**

```bash
# compile fisco-bcos
make -j4 #Note: j4 stand for use 4 threads complicating compile, can be configured as needs.
# start fisco-bcos:
bash start.sh
```

<br>


## 2 Other dependencies

**(1) Client： [sig-service-client](https://github.com/FISCO-BCOS/sig-service-client)**

sig-service-client provide below features：

- Access [sig-service)](https://github.com/FISCO-BCOS/sig-service) rpc interface.
- Put signature on the blockchain(make transaction)

Detail usage and deployment[ Group signature & Ring signature client guidebook](https://github.com/FISCO-BCOS/FISCO-BCOS/tree/master/doc/manual).

<br>

**(2) group signature & ring signature service：[sig-service](https://github.com/FISCO-BCOS/sig-service)**


sig-service is deployed in the organization and provide below features for [sig-service-client](https://github.com/FISCO-BCOS/sig-service-client):

- Create Group, add group member, generate group signature, verify group signature, track signer's identity.
- Create Ring, generate ring signature, verify ring signature.

In FISCO BCOS, sig-service-client always request the group/ring signature first, then put the signature on the blockchain. Blockchain nodes verify the signature by invoking ethcall.

sig-service's usage and deployment [group signature & ring signature RPC guidebook](https://github.com/FISCO-BCOS/sig-service).

<br>



## 4 Note

**(1) Group signature & ring signature is backward compatible** 

<br>

(2) After enable ethcall, **you cannot stop ethcall service，otherwise the interface is not reachable when doing the verification, and causing abnormally exit**

If ethcall is stopped by mistake, you can revert back FISCO BCOS to the original version, the version when you enable the ethcall, to bring blockchain back to works.

<br>

**(3) The FISCO BCOS's version must be same on the same blockchain**

If a node enabled group/ring signature verification service, then all the other nodes must enable as well. Otherwise the nodes which are not enabled verification service will abnormally exit.
<br>

(4) Before invoking group/ring signature verification service, ** you must deploy [group signature & ring signature RPC guidebook](https://github.com/FISCO-BCOS/sig-service) and [sig-service-client](https://github.com/FISCO-BCOS/sig-service-client)**

[sig-service-client](https://github.com/FISCO-BCOS/sig-service-client) responsible for putting the signature on the blockchain，[group signature & ring signature RPC guidebook](https://github.com/FISCO-BCOS/sig-service) responsible for providing signature generating service
<br>

 