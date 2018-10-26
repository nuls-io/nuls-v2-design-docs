Chapter 1 Overall design (by google translate)

[TOC]

## 1、Business background

Blockchain is the operating system for decentralized applications. It needs cross-chain technology to connect the operating systems that run one by one, in order to form an ecological development, that is, the blockchain Internet.

Adhering to the idea of making the blockchain simpler and making NULS one of the most important blockchain infrastructures, the NULS core team decided to develop a more open, inclusive and new modular underlying architecture with a core purpose around the concession area. The blockchain is simpler to make a series of attempts.
One: rich NULS modules
The new architecture allows NULS modules to run independently, combining a basic blockchain operation framework through a standard base module interface. The NULS module does not limit any developers, supports all programming languages that can provide http services, and strives to get the most out of every blockchain technician without setting any threshold. On this basis, more possibilities are extended.

Second: building a technical community for NULS
NULS is a pure blockchain project that integrates the idea of blockchain to create a blockchain community of development, freedom, and evolution. Therefore, the vitality of NULS depends on the degree of development of the community, especially the degree of construction of the technical community. Therefore, NULS takes advantage of the new architecture design, simultaneously attaches great importance to and develops the technology community, allowing the technical community and the core development team to “distribute” from the beginning; to attract all regions of the world with open technology and ideal forward-looking. Added in the development of blockchain.

Based on the above, the NULS core team initiated the design of the new version of the architecture, and hoped that the partners in the community actively participated in the construction of NULS, contributing to the development of the blockchain industry, and contributing a bright future.

## 2、Design goals

- Define cross-chain standards to enable communication between different blockchains.
- Build a "satellite chain" to achieve asset flow between different blockchains.
- Using the microservices architecture, each module is a separate process service, without limiting the development language

Description:

- Why do you want to cross the chain?
  The blockchain has been introduced by many media as the next generation of the Internet. This argument is very reasonable, but the distance is still very far away, there are a lot of ways to go, and the blockchain has to become a real Internet, and there is also a Passing the pass is the passing of value. At present, there are two solutions: 1. A blockchain completes all applications and user value transfer. 2. A general value transfer protocol between blockchains. The first option seems simple, but it is very limited and difficult to implement. NULS believes that Option 2 is a better solution in the current environment, so in this direction, cross-chain agreements and related supporting facilities are the first step in this direction.
- From modular to microservices
  Nuls is a modular underlying infrastructure. The blockchain programs currently running are based on the Java language. Modularization only implements the modularization of the coding layer. Our goal is a more flexible, operational state module. The underlying facilities. It should be able to support anyone who wants to make a technical contribution, so you should not set the language threshold. It should be easier to extend, modify, and replace. Each of its modules should be simple, static, and should not be blocked by the blockchain. The complexity of the overall program impact. So we proposed the micro-service architecture idea, the module is more independent, the module business is more simple, the module supports multi-language development, the module is easier to expand, the module supports distributed deployment, and the module is easier to plug and unplug...

risk:

The blockchain client has higher performance requirements. Each module in the new architecture is an independent process. The process communicates through the http protocol. In the case of no initial network setup, only some simple performance tests can be used. Estimating performance metrics is likely to fail to meet the requirements of a high-performance blockchain application. Solution: Set up a simple network in a short time to test the performance more accurately.

## 3、Overall Description

### 3.1Overall architecture

![design](./image/bridge.png)

Description:

An independent satellite chain that is responsible for docking with all chains. Implement inter-chain communication in an open manner

Based on the blockchain implemented by the NULS module warehouse (blockchain in the ecosystem), a cross-chain module can be added by means of module selection, so that the ground layer can communicate with the satellite chain.

For the Ethereum and Bitcoin, the public chain that is not affected by NULS needs to implement the conversion of the protocol through a special mechanism, and adapt the public chain protocol and the NULS cross-chain protocol to achieve the purpose of unified protocol communication.

All blockchains only communicate with the satellite chain, the verification of the transaction is carried out by the satellite chain, and the parallel chains trust the verification results of the satellite chain.

- Inter-chain connection
  Each node on the blockchain runs a cross-chain module, each node connecting some of the nodes on the satellite chain. The random algorithm is used to determine which nodes are connected, to ensure the dispersion of node connections as much as possible, and to ensure the security of the network.
- Multiple algorithm adaptation
  The satellite chain supports most of the mathematical algorithms on the market, including digest algorithms, symmetric encryption, asymmetric encryption, etc., which can be used through a unified interface provided by the algorithm library.
- Community governance
  The satellite chain will have built-in community governance mechanisms, including system operating parameter modifications, protocol upgrades, malicious chain processing, community funding, and more.


## 4、 Satellite chain design

###  4.1 Satellite chain architecture

 ![layer](./image/bridge-layer.png)

* The satellite chain uses the POC consensus mechanism to combine the Byzantine fault-tolerant mechanism to confirm and package cross-chain transactions, and to achieve decentralization and performance and security.
* Each node on the satellite chain connects multiple nodes in multiple blockchains. Because the protocol is a uniformly defined NULS cross-chain protocol, it is possible to connect multiple nodes on different blockchains simultaneously.
* The satellite chain provides a chain management mechanism to manage all peer-to-peer blockchains registered in the satellite chain. The contents of the registration include chain information, asset information, cross-chain mortgages, etc.
* When an asset of another chain is received on a blockchain, the corresponding asset needs to be generated in the chain. Tokens on different blockchains are stored in other chains in the form of assets.
* A breakdown of the assets transferred to another chain in a blockchain will be stored in the satellite chain. When the asset is transferred out of the blockchain, it will be verified, and illegal assets will not be allowed to be generated from the blockchain. Malicious blockchains are handled through community mechanisms such as suspending cross-chains, suspending cross-chains, forfeiting margins, etc.
* The Api user manual will be available in the satellite chain. Any developer can develop their own wallet, browser, light wallet and other tools according to the manual.
* In order to reduce the business complexity of the satellite chain, smart contracts will not be operated in the satellite chain
* Provision of protocol provisioning in the satellite chain, which can be used for DAPP development and cross-chain protocol optimization

### 4.2 How the satellite chain run

![](./image/modules.png)

* Satellite chain architecture in a modular manner
* Each module is a microservice that can run independently
* Communication between microservices directly via http protocol
* Module does not limit development language
* Provide microkernel module responsible for service management, configuration management and data sharing
* The module of the satellite chain will be shared with the NULS main network to a certain extent, so the module of the satellite chain will be added to the NULS module warehouse just like the NULS module for direct use by applications such as "chain factory".
* Each module supports extensions at the same time as it is used, ie if the modules in the module repository can only meet some of the business requirements, the module can be extended to avoid the workload of redevelopment

### 4.3 Bottom support for the chain factory

In the future NULS ecosystem, there will be a NULS main network and several application chains. Currently, there are two basic application chains to be built in the planning, smart contract chain and chain factory application chain. The chain factory is an application, and it is also a blockchain. Users can issue their own chains in the chain. The nodes in the chain factory can choose their own nodes to run several chains to realize the sharing of hardware devices.

The chain factory is built on the NULS module, so when designing the NULS module, consider supporting multiple chains at the same time.

##  5、Core process

### 5.1 Cross-chain transaction processing flow

1. The address a in the friend chain A initiates the transaction tx_a, and transfers the aCoin to the b address of the B chain.

- The format of the b address is the address in the nuls format starting with ChainId_B. When the asset is transferred to the address, the address is not allowed to initiate a transaction on the A chain, ie the address of the other chain cannot initiate a transaction in the chain.
- Generate a transaction for the satellite chain tx_a_trans based on the cross-chain protocol and sign the cross-chain transaction with the private key of b.

2. tx_a is packed in the A chain, and after the n block is confirmed, the satellite chain is sent by the cross-chain module (independent of the basic module other than the A-chain function).

3. The cross-chain module broadcasts the transaction to the connected satellite chain node (broadcast mode: first broadcast hash, wait for the other party to obtain a complete transaction)

4. After receiving the transaction (tx_a_trans), the satellite chain node (transaction management module) first performs basic verification (format, required field, signature, chain balance, etc.), and then asks whether the connected A-chain node has a The tx_a_hash transaction is confirmed by n blocks and converted to a satellite chain format transaction, the transaction digest is tx_a_trans_hash.

2. The result of the aggregate inquiry of the satellite chain node. If more than 51% of the nodes confirm the transaction, the node approves the transaction.

- The normal node asks for any node, the consensus node queries all nodes, and the ordinary node forwards the transaction if any 3 nodes confirm it, otherwise it discards. If the consensus node gets more than 51% node confirmation, it will sign tx_a_trans_hash and broadcast the hash and signature data to the network.

6. The satellite chain consensus node summarizes the cross-chain transaction signatures in the chain. When the signer of a transaction exceeds 80% of the total number of consensus nodes, the transaction is considered to be packaged into the block.

- This function is provided by the "Transaction Management" module. When 80% signature collection is completed, the transaction is pushed to the consensus module for packaging.

7. The consensus module of the satellite chain consensus node, when packing, verifies the number of signatures and the balance of the assets of the outbound chain. If the requirements are met, the transaction is packaged into the block.

8. Block confirmation logic: When verifying the cross-chain transaction included in the block, verify the number of signatures and the balance of the assets of the transfer chain, and confirm the transaction if it meets the requirements.

9. When the block where the transaction is located is confirmed, the "transaction management module" pushes the transaction (tx_a_trans) to the target chain node.

10. After receiving the transaction, the cross-chain module of the target chain queries the connected satellite chain node whether the transaction exists and has been confirmed. If more than 51% of the nodes confirm the transaction, the node recognizes the transaction.

11. After the node recognizes the transaction, the cross-chain module converts tx_a_trans into a B-chain asset transaction and broadcasts

12. If the node is the blocker of the last 20 blocks of the B chain (pow needs to be adapted), then sign the transaction before broadcasting.

13. The other node counts the signature of the transaction. When the signer reaches 80% of the latest 20 block, the transaction can be packaged (the packager also confirms the transaction), and the packaged transaction contains all signatures.

14. After the block is confirmed, the b address generates the corresponding a asset and can be used.

15. Complete

### 5.2 Block processing flow

1. the "transaction management" module to verify the transaction, will be placed in the memory pool, waiting for packaging

2. When the "consensus module" is packaged, the "transaction management" module is called to obtain the transaction interface to be packaged to obtain a transaction that can be packaged.

- "Transaction Management" module verifies coindata and business conflicts for local transactions
- The "Transaction Management" module verifies the number of signatures and the balance of the corresponding assets of the chain for cross-chain transactions

3. "Consensus Module" generates coinbase transactions and punish transactions

4. generate block headers and package blocks

5. the broadcast block head

6. Add the block to the verification queue of the "block management" module.
7. The verification thread takes the block out and verifies it.
8. After verification, confirm each transaction in turn
9. storage transactions and block headers

10. completed

### 5.3 This chain transaction processing flow

1. Assembling business data into the transaction

2. Obtain the account balance from the "book" module, calculate the handling fee and change the value, and fill the data to the coindata.

3. Sign the transaction

4. Transfer the local transaction to the "book" module for storage.

5. broadcast transactions

6. Submit the transaction to the "Transaction Management" module for verification.

7. verify the transaction, put into the memory pool

8. The consensus module obtains the list of transactions to be packaged (the "transaction management" module re-verifies the transaction)

9. After the "block management" module passes the verification, the processor interface of the transaction is invoked to process the transaction business.

10. Stored in the "Block Management" module together with the block header

## 6、Brief description of the modules

| module name          | description                                                  |
| -------------------- | ------------------------------------------------------------ |
| kernel               | Kernel module, responsible for module management, service management, configuration management functions, is the core of the system |
| account              | Account module, responsible for storing and maintaining local account information |
| block                | Block management: used to maintain blockchain data           |
| chain                | Chain Management: Chain information for managing all access cross-chain protocols |
| consensus            | Consensus module: used to run POC consensus mechanism        |
| ledger               | Ledgermodule: record transaction information, account balance model |
| transaction          | Transaction management module: transaction collection, verification, submission of consensus block packaging, transaction processing |
| network              | Network module: connection management, node management, message reception and transmission |
| smart-contract       | Smart Contract Module: Provides all the features of a smart contract engine |
| community-governance | Community Governance Module: Implementation of Governance Program on Chain |
| event-bus            | Event Bus: Publish and Subscribe Events                      |
|                      |                                                              |





