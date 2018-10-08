# Regulated Zero-knowledge proof

To meet the regulation requirement, when FISCO BCOS providing users zero-knowledge proof, it also satisfies that regulators can regulate every transaction.

## 1. Glossary

**[Zero-knowledge proof](#https://en.wikipedia.org/wiki/Zero-knowledge_proof)**: Let you validate the truth of something without revealing how you know that truth or sharing the content of this truth with the verifier.

**Zero-knowledge proof on blockchain**: User needs convert the data which needs to maintain secrecy also needs to be verified to proof and provide the proof for blockchain nodes. The blockchain node verifies the correctness of the secret data without knowing the user's secret data. Nobody can speculate the secret data by the proof, so there is regulatory risk.

**FISCO-BCOS regulated Zero-knowledge proof**: In FISCO BCOS, any zero-knowledge proof operation can be decrypted and regulated by the regulator. FISCO BCOS node is the verifier.

## 2. Underlying library

[libzkg：Regulated Zero-knowledge proof library](https://github.com/FISCO-BCOS/libzkg)

## 3. Scenario

**(1)One-to-one anonymous regulated transfer**

In FISCO BCOS, user can do one-to-one anonymous transfer without exposing identity and transfer amount. In the meantime, the regulator can decrypt the anonymous transfer. More details：[zkg-tx1to1](https://github.com/FISCO-BCOS/zkg-tx1to1)

![](assets/1-1_anonymous_transfer.png)

**(2)Coming soon...**