# Block management module design documents

[TOC]

## Chapter 1. General description

### 1.1 The module overview

#### 1.1.1 Why do you should have the block management module

All transaction data in the blockchain is stored in the block, so there is a module responsible for the saving and management of the block, so that other modules can obtain the block when verifying the data in the block and processing the business.

When the blockchain program is started for the first time, it is necessary to synchronize the full block on the network to the local. This process is generally time consuming, and the transaction cannot be initiated when the synchronization is not completed, so it is suitable for the work to be performed by a separate module.

In summary, it is necessary to provide a unified block data service for other modules, and it is also better to separate the management of the block from the specific service of the block. The modules used in the block do not have to care about the block. Get the details.

#### 1.1.2 Block management module's tasks

When the system starts, it is judged whether the height of the local block reaches the latest height of most nodes on the network. If it is not reached, the block is downloaded from the network to the local area, and the block is verified. After the verification is passed, the data is saved to the local database. This is called synchronization of blocks.

After the block synchronization is completed, the system starts normal operation and is discussed in the following two cases.

- If the self-node performs the packing block, the consensus module broadcasts the block information to the network before the packaged block is handed over to the block management module. The block management module verifies that the block is legal, and the verification passes and saves. Go to the database and respond to requests from other nodes on the network to get the block.

- If other nodes on the network are packing blocks, the local node will receive the forwarding block message sent from the network. At this time, the block information is obtained from other nodes, verified and saved.

In the abnormal case, the block verification fails, and the new block cannot be connected to the last block on the main chain. The block is regarded as a forked block and placed in the forked chain set for maintenance. When it is found that one of the bifurcation chains A is longer than the main chain B, the switching is performed, and the bifurcation chain A is the latest main chain, and the original main chain B enters the bifurcation chain assembly maintenance.

Provide block header and block query services to the outside world

#### 1.1.3 The position of block management module in the system

Block management is one of the underlying modules. The following sub-functions discuss module dependencies.

rely

* Block synchronization - depends on the communication interface of the network module, relying on the serialization tool of the tool module
* Block storage, rollback-dependent tool module database storage tool, consensus module, transaction management module
* Block forwarding - dependent on the network module's broadcast message interface

Be dependent

* The entire system can initiate transactions - block synchronization
* Consensus module: block detailed verification, packing - block query, block saving, block broadcasting, block rollback

### 1.2 Architecture diagram

![](./image/block-module/block-module.png)

## Chapter 2. Functional design

### 2.1 Functional architecture diagram

![](image/block-module/block-functions.png)

1. Provide the API to block storage, query, a rollback operation
2. Synchronous latest blocks from the Internet, a preliminary verification, validation, bifurcation without bifurcate, call consensus in a consensus based verification, the trading module dual authentication, all validation after saved to the local.
3. Block synchronization, broadcasting, forwarding message process
4. Fork chains maintenance and switch
5. Fork blocks judgment and storage

### 2.2 The module service

#### 2.2.1 Get the latest local block header

* Interface specification

1. Query the DB according to the chain ID and the latest block height of the cache to get the latest block header HASH.
2. According to the HASH query DB to get the block header byte array
3. Deserialize to block header object

* Sample request

    ```
    {
      "cmd": "bl_bestBlockHeader",
      "minVersion":"1.1",
      "params": ["888"]
    }
    ```

* Instructions of request parameters

| index | parameter | required | type    | description |
| ----- | --------- | -------- | ------- | :---------: |
| 0     | chainId   | true     | Long  |   chain ID    |
    
* response sample

    Failed
    
    ```
    {
        "version": 1.2,
        "code": 1,
        "msg": "error message",
        "result": {}
    }
    ```
    
    Success
    
    ```
    {
        "version": 1.2,
        "code": 0,
        "result": {
            "chainId": "888",
            "hash": "xxxxxxx",
            "preHash": "xxxxxxx",
            "merkleHash": "1",
            "height": 1,
            "size": 1,
            "time": 1,
            "txCount": 1,
            "packingAddress": "1",
            "reward": 0,
            "fee": 0,
            "extend": xxxxxxx,HEX
            "scriptSig": "1"
        }
    }
    ```
    
* Instructions of response parameters
  
| parameter | type      | description                                |
| --------- | --------- | ------------------------------------------ |
| chainId      | Long    | chain ID                                |
| hash      | String    | block HASH                                |
| preHash   | String    | pre block HASH                              |
| merkleHash   | String    | block's MerkleHash                              |
| height   | Long | block height                              |
| size   | Integer    | block size                              |
| time   | Long    | packing timestamp                              |
| txCount   | Integer    | count of transactions                              |
| packingAddress   | String    | packing address                              |
| reward   | Long    | Consensus reward                              |
| fee   | Long | procedure fee                             |
| extend   | String   | Extension field,HEX,contains roundIndex、roundStartTime、consensusMemberCount、packingIndexOfRound、stateRoot                              |
| scriptSig   | String    | block's signature                               |

#### 2.2.2 Get the latest local block

* Interface specification：

1. Get the latest local block header based on the chain ID
2. Query the DB according to the block head height to get the transaction HASH list.
3. Obtain transaction data from the transaction management module according to the HASH list
4. Assemble into a block object

* Sample request

    ```
    {
      "cmd": "bl_bestBlock",
      "minVersion":"1.1",
      "params": [“888”]
    }
    ```

* Instructions of request parameters

| index | parameter | required | type    | description |
| ----- | --------- | -------- | ------- | :---------: |
| 0     | chainId   | true     | Long  |   chain ID    |

* response sample 

    Failed

      ```
      {
          "version": 1.2,
          "code":1,
          "msg" :"xxxxxxxxxxxxxxxxxx",
          "result":{}
      }
      ```

    Success

    ```
    {
        "version": 1.2,
        "code": 0,
        "result": {
        	"blockHeader": {
                "chainId": "888",
                "hash": "xxxxxxx",
                "preHash": "xxxxxxx",
                "merkleHash": "1",
                "height": 1,
                "size": 1,
                "time": 1,
                "txCount": 1,
                "packingAddress": "1",
                "reward": 0,
                "fee": 0,
                "extend": xxxxxxx,HEX
                "scriptSig": "1"
        	}, //block header
        	"transactions": [
        	    {
                    "chainId": "888", //chain Id
                    "height": "1", //block height
                    "hash": "1", //transaction HASH
                    "remark": "1", //transaction remark
                    "size": "1", //transaction size
                    "time": "1", //transaction timestamp
                    "type": "1", //transaction type
                    "transactionSignature": "1", //transaction sign
                    "coinData": {
                        "from" : [
                            {
                                “fromAssetsChainId”：“”  
                                “fromAssetsId”：“”
                                “fromAddress”：“”
                                “amount”：“”
                                “nonce”：“”//Transaction sequence number and increasing
                            },{...}
                        ]
                        "to" : [
                            {
                                “toAssetsChainId”：“”
                                “toAssetsId”：“”
                                “toAddress”：“”
                                “amount”：“”
                                “locktime”：“”
                            },{...}
                        ]
                    }
                    "txData": XXXX, //Special transaction data HEX
        	    },
        	    {...}
        	], //transaction list
        }
    }
    ```

* Instructions of response parameters

    omit

#### 2.2.3 get block header according to the height

* Interface specification

1. According to the chain ID and height query DB, get the latest block header HASH
2. According to the HASH query DB to get the block header byte array
3. Deserialize to block header object

* Sample request

    ```
    {
      "cmd": "bl_getBlockHeaderByHeight",
      "minVersion":"1.1",
      "params": ["111","888"]
    }
    ```

* Instructions of request parameters

| index | parameter | required | type    | description |
| ----- | --------- | -------- | ------- | :---------: |
| 0     | chainId   | true     | Long  |   chain ID    |
| 1     | height   | true     | Long  |   block's height    |
    
* response sample

    Failed
    
    ```
    {
        "version": 1.2,
        "code": 1,
        "msg": "error message",
        "result": {}
    }
    ```
    
    Success
    
    ```
    {
        "version": 1.2,
        "code": 0,
        "result": {
            "chainId": "888",
            "hash": "xxxxxxx",
            "preHash": "xxxxxxx",
            "merkleHash": "1",
            "height": 1,
            "size": 1,
            "time": 1,
            "txCount": 1,
            "packingAddress": "1",
            "reward": 0,
            "fee": 0,
            "extend": xxxxxxx,HEX
            "scriptSig": "1"
        }
    }
    ```
    
* Instructions of response parameters
  
| parameter | type      | description                                |
| --------- | --------- | ------------------------------------------ |
| chainId      | Long    | chain ID                                |
| hash      | String    | block hash                                |
| preHash   | String    | pre block hash                              |
| merkleHash   | String    | block's MerkleHash                              |
| height   | Long | block's height                              |
| size   | Integer    | block's size                              |
| time   | Long    | block's packing timestamp                              |
| txCount   | Integer    | count of transactions                              |
| packingAddress   | String    | address of packing                              |
| reward   | Long    | Consensus reward                              |
| fee   | Long | procedure fee                             |
| extend   | String   | Extension field,HEX,contains roundIndex、roundStartTime、consensusMemberCount、packingIndexOfRound、stateRoot                              |
| scriptSig   | String    | Signature of block                              |

#### 2.2.4 get block according to the height

* Interface specification：

1. Get the block header according to the chain ID and height
2. Query the DB according to the block head height to get the transaction HASH list.
3. Obtain transaction data from the transaction management module according to the HASH list
4. Assemble into a block object

* Sample request

    ```
    {
      "cmd": "bl_getBlockByHeight",
      "minVersion":"1.1",
      "params": [“111”,"888"]
    }
    ```

* Instructions of request parameters

| index | parameter | required | type    | description |
| ----- | --------- | -------- | ------- | :---------: |
| 0     | chainId   | true     | Long  |   chain ID    |
| 1     | height   | true     | Long  |   block's height    |

* response sample 

    Failed

      ```
      {
          "version": 1.2,
          "code":1,
          "msg" :"xxxxxxxxxxxxxxxxxx",
          "result":{}
      }
      ```

    Success

    ```
    {
        "version": 1.2,
        "code": 0,
        "result": {
        	"blockHeader": {
                "chainId": "888",
                "hash": "xxxxxxx",
                "preHash": "xxxxxxx",
                "merkleHash": "1",
                "height": 1,
                "size": 1,
                "time": 1,
                "txCount": 1,
                "packingAddress": "1",
                "reward": 0,
                "fee": 0,
                "extend": xxxxxxx,HEX
                "scriptSig": "1"
        	}, //block header
        	"transactions": [
        	    {
                    "chainId": "888",//chain ID
                    "height": "1", //block's height
                    "hash": "1", //transaction's hash
                    "remark": "1", //transaction's remark
                    "size": "1", //transaction's size
                    "time": "1", //transaction's timestamp
                    "type": "1", //transaction's type
                    "transactionSignature": "1", //transaction's sign
                    "coinData": {
                        "from" : [
                            {
                                “fromAssetsChainId”：“”  
                                “fromAssetsId”：“”
                                “fromAddress”：“”
                                “amount”：“”
                                “nonce”：“”
                            },{...}
                        ]
                        "to" : [
                            {
                                “toAssetsChainId”：“”  
                                “toAssetsId”：“”
                                “toAddress”：“”
                                “amount”：“”
                                “nonce”：“”
                            },{...}
                        ]
                    }
                    "txData": XXXX, //HEX
        	    },
        	    {...}
        	], 
        }
    }
    ```

* Instructions of response parameters

    omit

#### 2.2.5 get block header according to the hash

* Interface specification

1. According to the chain ID, HASH query DB to get the block header byte array
2. Deserialize to block header object

* Sample request

    ```
    {
      "cmd": "bl_getBlockHeaderByHash",
      "minVersion":"1.1",
      "params": ["888","aaa"]
    }
    ```

* Instructions of request parameters

| index | parameter | required | type    | description |
| ----- | --------- | -------- | ------- | :---------: |
| 0     | chainId   | true     | Long  |   chain ID    |
| 1     | hash   | true     | String  |   block hash    |
    
* response sample

    Failed
    
    ```
    {
        "version": 1.2,
        "code": 1,
        "msg": "error message",
        "result": {}
    }
    ```
    
    Success
    
    ```
    {
        "version": 1.2,
        "code": 0,
        "result": {
            "chainId": "888",
            "hash": "xxxxxxx",
            "preHash": "xxxxxxx",
            "merkleHash": "1",
            "height": 1,
            "size": 1,
            "time": 1,
            "txCount": 1,
            "packingAddress": "1",
            "reward": 0,
            "fee": 0,
            "extend": xxxxxxx,HEX
            "scriptSig": "1"
        }
    }
    ```
    
* Instructions of response parameters
  
| parameter | type      | description                                |
| --------- | --------- | ------------------------------------------ |
| chainId      | Long    | chain ID                                |
| hash      | String    | block hash                                |
| preHash   | String    | pre block hash                              |
| merkleHash   | String    | block's MerkleHash                              |
| height   | Long | block's height                              |
| size   | Integer    | block's size                              |
| time   | Long    | block's packing timestamp                              |
| txCount   | Integer    | count of transactions                              |
| packingAddress   | String    | address of packing                              |
| reward   | Long    | Consensus reward                              |
| fee   | Long | procedure fee                             |
| extend   | String   | Extension field,HEX,contains roundIndex、roundStartTime、consensusMemberCount、packingIndexOfRound、stateRoot                              |
| scriptSig   | String    | Signature of block                              |

#### 2.2.6 get block according to the hash

* Interface specification：

1. Get the block header according to the chain ID and hash
2. Query the DB according to the block head height to get the transaction HASH list.
3. Obtain transaction data from the transaction management module according to the HASH list
4. Assemble into a block object

* Sample request

    ```
    {
      "cmd": "bl_getBlockByHash",
      "minVersion":"1.1",
      "params": ["888",“aaa”]
    }
    ```

* Instructions of request parameters

| index | parameter | required | type    | description |
| ----- | --------- | -------- | ------- | :---------: |
| 0     | chainId   | true     | Long  |   chain ID    |
| 1     | hash   | true     | String  |   block hash    |

* response sample 

    Failed

      ```
      {
          "version": 1.2,
          "code":1,
          "msg" :"xxxxxxxxxxxxxxxxxx",
          "result":{}
      }
      ```

    Success

    ```
    {
        "version": 1.2,
        "code": 0,
        "result": {
        	"blockHeader": {
        	    "chainId": "888",
                "hash": "xxxxxxx",
                "preHash": "xxxxxxx",
                "merkleHash": "1",
                "height": 1,
                "size": 1,
                "time": 1,
                "txCount": 1,
                "packingAddress": "1",
                "reward": 0,
                "fee": 0,
                "extend": xxxxxxx,HEX
                "scriptSig": "1"
        	}, //block header
        	"transactions": [
        	    {
        	        "chainId": "888",
                    "height": "1", //block's height
                    "hash": "1", //transaction's hash
                    "remark": "1", //transaction's remark
                    "size": "1", //transaction's size
                    "time": "1", //transaction's timestamp
                    "type": "1", //transaction's type
                    "transactionSignature": "1", //transaction's sign
                    "coinData": {
                        "from" : [
                            {
                                “fromAssetsChainId”：“”  
                                “fromAssetsId”：“”
                                “fromAddress”：“”
                                “amount”：“”
                                “nonce”：“”
                            },{...}
                        ]
                        "to" : [
                            {
                                “toAssetsChainId”：“”  
                                “toAssetsId”：“”
                                “toAddress”：“”
                                “amount”：“”
                                “nonce”：“”
                            },{...}
                        ]
                    }
                    "txData": XXXX, //HEX
        	    },
        	    {...}
        	], 
        }
    }
    ```

* Instructions of response parameters

    omit

#### 2.2.7 get sync status

* Interface specification

When the block synchronization on a ChainID is completed, the cached synchronization status identifier is updated. It is forbidden to initiate a transaction when the sync block is not completed.

* Sample request

    ```
    {
      "cmd": "bl_getSynchronizeInfo",
      "minVersion":"1.1",
      "params": ["888"]
    }
    ```

* Instructions of request parameters

| index | parameter | required | type    | description |
| ----- | --------- | -------- | ------- | :---------: |
| 0     | chainId   | true     | Long  |   chain ID    |
    
* response sample

    Failed
    
    ```
    {
        "version": 1.2,
        "code": 1,
        "msg": "error message",
        "result": {}
    }
    ```
    
    Success
    
    ```
    {
        "version": 1.2,
        "code": 0,
        "result": {"sync": "true"}
    }
    ```
    
* Instructions of response parameters
  
| parameter | type      | description                                |
| --------- | --------- | ------------------------------------------ |
| sync      | String    | Block the synchronization to be achieved   |

#### 2.2.8 get block header according to the height range

* Interface specification

1. Make queryHash = endHash
2. According to queryHash query DB get block headerbyte array
3. Deserialization for block header object blockHeader, added to the List as a return value
4. If the blockHeader. Hash!= startHash, make queryHash = blockHeader preHash, startHash, repeat step 2
5. Return to the List

* Sample request

    ```
    {
      "cmd": "bl_getBlockHeaderBetweenHeights",
      "minVersion":"1.1",
      "params": ["888",111","111"]
    }
    ```

* Instructions of request parameters

| index | parameter | required | type    | description |
| ----- | --------- | -------- | ------- | :---------: |
| 0     | chainId   | true     | Long  |   chain ID    |
| 1     | startHeight   | true     | Long  |   start Height    |
| 2     | endHeight   | true     | Long  |   end Height    |
    
* response sample

    Failed
    
    ```
    {
        "version": 1.2,
        "code": 1,
        "msg": "error message",
        "result": {}
    }
    ```
    
    Success
    
    ```
    {
        "version": 1.2,
        "code": 0,
        "result": {
            “list” : [
                {
               "chainId": "888",
               "hash": "xxxxxxx",
               "preHash": "xxxxxxx",
               "merkleHash": "1",
               "height": 1,
               "size": 1,
               "time": 1,
               "txCount": 1,
               "packingAddress": "1",
               "reward": 0,
               "fee": 0,
               "extend": xxxxxxx,HEX
               "scriptSig": "1"
               }
            ]

        }
    }
    ```
    
* Instructions of response parameters
  
| parameter | type      | description                                |
| --------- | --------- | ------------------------------------------ |
| chainId      | Long    | chain ID                                |
| hash      | String    | block hash                                |
| preHash   | String    | pre block hash                              |
| merkleHash   | String    | block's MerkleHash                              |
| height   | Long | block's height                              |
| size   | Integer    | block's size                              |
| time   | Long    | block's packing timestamp                              |
| txCount   | Integer    | count of transactions                              |
| packingAddress   | String    | address of packing                              |
| reward   | Long    | Consensus reward                              |
| fee   | Long | procedure fee                             |
| extend   | String   | Extension field,HEX,contains roundIndex、roundStartTime、consensusMemberCount、packingIndexOfRound、stateRoot                              |
| scriptSig   | String    | Signature of block                              |

#### 2.2.9 get block according to the height range

* Interface specification

1. make queryHash=endHash
2. According to the chain ID, queryHash query DB to get the block byte array
3. Deserialize to block object block, add to List as return value
4. If block.hash!=startHash, make queryHash=block.preHash, startHash, repeat step 2
5. Return to List

* Sample request

    ```
    {
      "cmd": "bl_getBlockBetweenHeights",
      "minVersion":"1.1",
      "params": ["888",111","111"]
    }
    ```

* Instructions of request parameters

| index | parameter | required | type    | description |
| ----- | --------- | -------- | ------- | :---------: |
| 0     | chainId   | true     | Long  |   chain ID    |
| 1     | startHeight   | true     | Long  |   start Height    |
| 2     | endHeight   | true     | Long  |   end Height    |
    
* response sample

    Failed
    
    ```
    {
        "version": 1.2,
        "code": 1,
        "msg": "error message",
        "result": {}
    }
    ```
    
    Success
    
    ```
    {
        "version": 1.2,
        "code": 0,
        "result": {
            “list” : [
                {
                    "blockHeader": {
                        "chainId": "888",
                        "hash": "xxxxxxx",
                        "preHash": "xxxxxxx",
                        "merkleHash": "1",
                        "height": 1,
                        "size": 1,
                        "time": 1,
                        "txCount": 1,
                        "packingAddress": "1",
                        "reward": 0,
                        "fee": 0,
                        "extend": xxxxxxx,HEX
                        "scriptSig": "1"
                    }, //block header
                    "transactions": [
                        {
                            "chainId": "888",
                            "height": "1", //block's height
                            "hash": "1", //transaction's hash
                            "remark": "1", //transaction's remark
                            "size": "1", //transaction's size
                            "time": "1", //transaction's timestamp
                            "type": "1", //transaction's type
                            "transactionSignature": "1", //transaction's sign
                            "coinData": {
                                "from" : [
                                    {
                                        “fromAssetsChainId”：“”  
                                        “fromAssetsId”：“”
                                        “fromAddress”：“”
                                        “amount”：“”
                                        “nonce”：“”
                                    },{...}
                                ]
                                "to" : [
                                    {
                                        “toAssetsChainId”：“”  
                                        “toAssetsId”：“”
                                        “toAddress”：“”
                                        “amount”：“”
                                        “nonce”：“”
                                    },{...}
                                ]
                            }
                            "txData": XXXX, //HEX
                        },
                        {...}
                    ], 
               }
            ]

        }
    }
    ```
    
* Instructions of response parameters
  
    omit

#### 2.2.10 set node's latest height, latest Hash

* Interface specification

After the local node consensus module is packaged, this interface is called to save the block data without verification.

* Sample request

    ```
    {
      "cmd": "bl_setNodesInfo",
      "minVersion":"1.1",
      "params": [
        {
           chainId：122,//chain ID
           nodeId:"20.20.30.10:9902"
           magicNumber：134124,//Magic number
           version：2,//Protocol version number
           blockHeight：6000,   //block's height
           blockHash："0020ba3f3f637ef53d025d",  //block Hash
           ip："200.25.36.41",//ip
           port：54,//
           state："connected",
           isOut："1", //0-Passive connection,1-active connection
           time："6449878789", //Recent connection time
        },{}
      ]
    }
    ```

* Instructions of request parameters

    omit
    
* response sample

    Failed
    
    ```
    {
        "version": 1.2,
        "code": 1,
        "msg": "error message",
        "result": {}
    }
    ```
    
    Success
    
    ```
    {
        "version": 1.2,
        "code": 0,
        "result": {"sync": "true"}
    }
    ```
    
* Instructions of response parameters
  
| parameter | type      | description                                |
| --------- | --------- | ------------------------------------------ |
| sync      | String    | Block information synchronization is completed    |

### 2.3 module's internal function

#### 2.3.1 module's boot

* Functional specifications：

  omit

* process description

![](./image/block-module/block-module-boot.png)

1. The loading block module configuration information
2. The registration of block module, message handler
3. Register block module service interface
4. Register block module
5. Start the synchronization of threads, the block monitoring thread, branching chain processing threads

* Dependent service

  Tool module、kernel module

#### 2.3.2 block's storage

* Functional specifications：

Description storage table division

   * main chain's storage

A complete block consists of a block header and a transaction, and the block header is stored separately from the transaction.
    Block header: (in the block management module)
        Key(block's height)-value(block header hash) block-header-index
        Key(block headerhash)-value(complete block header) block-header
    Trading: (put in the transaction management module)

   
   * fork chain's storage
Different branches of the bifurcation chain have different tables, table name plus chainID suffix
    Key (start block height + start block hash) - value (complete chains) chains
   ```
        ChainContainer
            private Chain chain;
                    private String id;
                    private String preChainId;
                    private BlockHeader startBlockHeader;
                    private BlockHeader endBlockHeader;
                    private List<BlockHeader> blockHeaderList;
                    private List<Block> blockList;
   ```

* process description

    omit

* Dependent service

  Database storage tool of tool modules

#### 2.3.2 block's Synchronization

* Functional specifications：

  omit

* process description

    * Block synchronization main flow
    
    ![](./image/block-module/block-synchronization.png)
    
    * Get a list of available nodes on the network
    
    ```
        1. Traverse the node and count the two MAPs, assuming that each node (the latest HASH+ latest height) is the key
        2. A key with the key as the number of statistics
        3. A key is used to record the list of nodes holding the key.
        4. Finally, the most frequently occurring key is obtained, and the current trusted latest height and latest hash, as well as the list of trusted nodes are obtained.
        
        for example:
        Now connect to 10 nodes at the same time. The latest block's height of 4 nodes (A, B, C, D) is 100, the latest block hash is aaa, and the latest block's height of 6 nodes (E, F, G, H, I, J) is 101. The latest block hash is bbb.
        Finally return (101, bbb, [E, F, G, H, I, J]).
    ```
    
    * Download block logic
    
    ![](./image/block-module/block-synchronization2.png)
    
    ```
            Before the official download of the block, it is necessary to determine whether the local and the network are forked, and whether it needs to be rolled back. In order to find the exact block download height.
            The following discussion is divided into:
            Take the result of the previous step (101, bbb, [E, F, G, H, I, J]), while LH(N) represents the hash of the local Nth block, and RH(N) represents the hash of the Nth block on the network. .
            1. Local height 100 < network height 101, LH (100) == RH (100), normal, behind the remote node, download block
            2. Local height 100 < network height 101, LH (100)! = RH (100), think local fork, roll back the local block, if LH (99) == RH (99)
            At the end of the rollback, download from 99 blocks. If LH(99)!=RH(99), continue to roll back and repeat the above logic. However, if you roll back 10 blocks at most, it will stop and wait for the next synchronization. This will avoid being attacked by malicious nodes and roll back normal blocks in large quantities.
            3. Local height 102> network height 101, LH (101) == RH (101), normal, leading than remote node, no need to download block
            4. Local height 102> network height 101, LH (101)! = RH (101), think local fork, first roll back to the height and remote consistency, repeat scene 2
            5. Local height 101 = network height 101, LH (101) == RH (101), normal, consistent with the remote node, no need to download the block
            6. Local height 101 = network height 101, LH (101)! = RH (101), think local fork, repeat scene 2
            
            In the scenario that needs to be rolled back, the number of available nodes (10) > configuration, the number of consistent available nodes (6) accounted for more than 80%, and avoiding too few nodes leads to frequent rollback. The above two conditions are not met, empty the connected nodes, and re-acquire the available nodes.
     
            When you actually download the block, give a chestnut:
            The current height is 100, the network height is 500, 12 nodes are available, 10 nodes are consistently available, and each node downloads 2 blocks at a time.
            Then calculate that you need to download 400 blocks, 400/(2*10)=20 rounds of downloading, and you can calculate the height range of each node download block per round.
            Pseudo code representation
                For (20 rounds){
                    For (10 nodes) {
                        Each node downloads the corresponding block and puts it in the shared queue for the block verification thread to process
                    }
                }
            Consider the case where the node is dropped during the download process. It is possible that 20 rounds cannot be downloaded, so the outer layer is added to the loop.
                While (not downloaded){
                    Recalculate the round, download the block's height interval for each node
                    For (20 rounds){
                        For (10 nodes) {
                            Each node downloads the corresponding block and puts it in the shared queue for the block verification thread to process
                        }
                    }
                }
    ```
    
    * Download blocks from a height interval from a node
    
    ![](./image/block-module/block-synchronization3.png)

* Dependent service

  Database storage tool of tool modules、RPC tool

#### 2.3.3 block's validation

* Functional specifications：

  Verify the correctness of the block's own data, verify during the download process, verify that there is no problem with the block data itself, and discard the block if the verification fails.

* process description

    * block's basic validation
    
    ![](./image/block-module/block-basic-validation.png)
    
    * block header's validation
    
    ![](./image/block-module/block-basic-validation1.png)
    
    * Merkelhash validation
    
    ![](./image/block-module/block-basic-validation2.png)

* Dependent service

  Database storage tool of tool modules

#### 2.3.4 fork block management

* Functional specifications：

  Verify the correctness of the block context. After the download is complete, verify that the block is connected to the main chain. 
  The verification failure indicates that the block is bifurcated and enters the forked chain processing logic.

* process description

    - Define a main chain (MasterChain), a fork chain set (forkChains)
    - Define the block to be verified as Block
    - Define the main chain height MG, to be verified block's heightBG
    - Define the latest block HASH of the main chain as MH, the HASH of the block to be verified is BH, and the PHHASH of the block to be verified is BPH
    
    - Discussion in six situations
    - 1.MG==BG, MH==BH, indicating that the latest main chain block is repeatedly received and discarded.
    - 2.MG==BG, MH!=BH, indicating network fork
        - Traverse the existing set of forked chains to determine if this block already exists
            - discard the block if it already exists
            - If it does not exist, create a new fork chain
              - Determine if you can connect to other fork chains (if you can connect, connect to other fork chains as a chain)
              - Recursive judgment
    - 3.MG==BG-1, MH==BPH, indicating that the block is continuous and saved to the main chain
    - 4.MG==BG-1, MH!=BPH, indicating network fork, processing the same as step 2
    - 5.MG<BG-1, indicating network fork, processing the same as step 2
    - 6.MG>BG, indicating network fork, processing the same as step 2
    
    The height difference is less than 1000 cached to disk, the disk space is limited in size, and the height is discarded. 
    If the cache space is full, the forked chain is cleared in the order of adding cache time.

  ![](./image/consensus-module/consensus-flow-4.png)

* Dependent service

  Database storage tool of tool modules

#### 2.3.5 fork chain management

* Functional specifications：

  Determine if the fork chain and the main chain need to be switched

* process description
  ​      
    - Check if there is an orphan chain that links to the main chain or the forked chain, if any
    - Take the longest one of the fork chains and compare the length of the main chain to determine whether you need to switch the main chain
        - If the length of the fork chain is longer than the length of the main chain by 3 (configured), you need to switch the main chain.
        - Find the bifurcation point of the main chain and the longest bifurcation chain
        - Verify the block in the forked chain if the verification continues by going down
        - Roll back the main chain block
        - Switch the fork chain as the main chain

![](./image/consensus-module/consensus-flow-3.png)

* Dependent service

  Database storage tool of tool modules

#### 2.3.6 forward block

* Functional specifications：

    omit

* process description

1. Use the blockHash to assemble the ForwardSmallBlockMessage and send it to the target node.
2. After receiving the ForwardSmallBlockMessage, the target node takes out the hash to determine whether it is duplicated. If it does not repeat, use the hash assembly GetSmallBlockMessage to send to the source node.
3. After the source node receives the GetSmallBlockMessage, it takes out the hash, queries the SmallBlock and assembles the SmallBlockMessage, and sends it to the target node.
4. Subsequent interaction process reference broadcast block

* Dependent service

  Database storage tool of tool modules

#### 2.3.7 braodcast block

* Functional specifications：

  omit

* process description

1. Get BlockHeader, TxList according to HASH, assemble into SmallBlock,
2. Put a SmallBlock into the memory. If it is not deleted actively, it will be automatically cleared when the cache is full or exists for more than 1000 seconds.
3. Local cache blockHash for filtering duplicate downloads
4. Assem the SmallBlockMessage and call the RPC module to send a message to the target node.
5. After receiving the message, the target node determines which transactions are not locally based on txHashList, and then assembles GetTxGroupRequest to the source node.
6. After receiving the information, the source node assembles the TxGroupMessage according to the hashlist and returns it to the target node.
7. All block data has been sent to the target node.
  
* Dependent service

  Database storage tool of tool modules
  
#### 2.3.8 Block monitoring

* Functional specifications：

  omit

* process description

![](./image/consensus-module/consensus-flow-6.png)

    - Start monitoring scheduled tasks, once every minute
    - Take the local latest block header
    - Verify that the network module needs to be restarted (if the latest local block has not been updated for 3 minutes, the network module needs to be disconnected and reconnected randomly)
    - to be perfected

* Dependent service

  Database storage tool of tool modules

## Chapter 3. Events

### 3.1 published event

#### 3.1.1 Synchronization completed

Description: The synchronization is completed. When the height of the area is the same as the height of the network, the event is released.

  Event_topic : "bl_blockSyncComplete",

```
Data:{
     chainId
     Height
     Hash
}
```

#### 3.1.2 save block event

Description：Each save a block, release the event

 event_topic : "evt_bl_saveBlock",

```
data:{
    chainId
    height
    hash
}
```

#### 3.1.3 rollback block event

Description：Each roll back a block, release the event

 event_topic : "evt_bl_rollbackBlock",

```
data:{
    chainId
    height
    hash
}
```

### 3.2 Subscribed event

    omit

## Chapter 4. Network Message

### 4.1 Network communication protocol

    omit

### 4.2 Message protocol

#### 4.2.1 Forward block message-ForwardSmallBlockMessage

* Message description：Used for the "forward block" function

* Message type（cmd）

  ForwardSmallBlock

* Message format（txData）

| Length | Fields  | Type      | Remark         |
| ------ | ------- | --------- | -------------- |
| 32     | chainID  | uint32      | chain ID           |
| 1     | digestAlgType  | byte      | digest algorithm identifier           |
| ?     | hashLength        | VarInt    | array's length           |
| ?     | hash        | byte[]    | hash           |

* Message validation

    omit

* Message processing logic

1. After receiving the message, the target node first determines whether the hash in the cache is duplicated according to the chainID.
2. If it is repeated, it indicates that the SmallBlock forwarded by another node has been received, and the message is discarded.
3. If there is no duplication, assemble GetSmallBlockMessage with hash and send it to the source node.

#### 4.2.2 Get small block message-GetSmallBlockMessage

* Message description：Used for the "forward block" function

* Message type（cmd）

  GetSmallBlock

* Message format（txData） 

| Length | Fields  | Type      | Remark         |
| ------ | ------- | --------- | -------------- |
| 32     | chainID  | uint32      | chain ID           |
| 1     | digestAlgType  | byte      | digest algorithm identifier           |
| ?     | hashLength      | VarInt    | array's length           |
| ?     | hash            | byte[]    | hash           |

* Message validation

    omit

* Message processing logic

1. Get the SmallBlock object according to the hash
2. Assem the SmallBlockMessage and send it to the source node

#### 4.2.3 Block broadcast message-SmallBlockMessage

* Message description：Used for "forwarding block" and "broadcast block" functions

* Message type（cmd）

  SmallBlock

* Message format（txData）

| Length | Fields  | Type      | Remark         |
| ------ | ------- | --------- | -------------- |
| 32     | chainID  | uint32      | chain ID           |
| 1     | digestAlgType  | byte      | digest algorithm identifier           |
| ?     | preHashLength   | VarInt    | preHash array's length           |
| ?     | preHash         | byte[]    | preHash           |
| 1     | digestAlgType  | byte      | digest algorithm identifier           |
| ?     | merkleHashLength| VarInt    | merkleHash array's length           |
| ?     | merkleHash      | byte[]    | merkleHash           |
| 48     | time          | Uint48    | time           |
| 32     | height      | Uint32    | block's height           |
| 32     | txCount      | Uint32    | count of transactions           |
| ?     | extendLength| VarInt    | extend array's length           |
| ?     | extend      | byte[]    | extend           |
| 32     | publicKeyLength      | Uint32    | public key array's length           |
| ?     | publicKey      | byte[]    | public key           |
| 1     | signAlgType      | byte    | Signature algorithm type           |
| ?     | signBytesLength| VarInt    | sign array's length           |
| ?     | signBytes      | byte[]    | sign bytes           |
| ?     | txHashListLength| VarInt    | transaction's hash list array's length           |
| 1     | digestAlgType  | byte      | digest algorithm identifier           |
| ?     | txHashLength| VarInt    | transaction's hash array's length           |
| ?     | txHash      | byte[]    | transaction's hash           |

* Message validation

    omit

* Message processing logic

1. Determine whether the block timestamp is greater than (current time +10s). If it is greater than this time, it is determined to be maliciously prematurely out of the block, ignoring the message.
2. Determine whether the message is repeated according to the block hash. If it is repeated, ignore the message (requires maintenance of a set to store the received block hash)
3. Query the DB according to the block hash to see if the block already exists in the local area. If it already exists, ignore the message.
4. Verify the block header, if the verification fails, ignore the message
5. Take txHashList, determine that tx is not local, assemble GetTxGroupMessage, and send it to the source node.

#### 4.2.4 Get block message based on height-GetBlocksByHeightMessage

* Message description：For the "sync block" function

* Message type（cmd）

  GetBlocksByHeight

* Message format（txData）

| Length | Fields  | Type      | Remark         |
| ------ | ------- | --------- | -------------- |
| 32     | chainID  | uint32      | chain ID           |
| 32     | startHeight  | uint32      | start Height           |
| 32     | endHeight  | uint32      | end Height           |

* Message validation

    omit

* Message processing logic

1. Height parameter verification
2. Return the response message ReactMessage
3. Find the Block from endHeight, assemble the BlockMessage, and send it to the target node.
4. Find the startHeight, assemble the CompleteMessage, and send it to the target node.

#### 4.2.5 Get block message-GetBlockMessage

* Message description：Used for "block synchronization"

* Message type（cmd）

  GetBlock

* Message format（txData）

| Length | Fields  | Type      | Remark         |
| ------ | ------- | --------- | -------------- |
| 32     | chainID  | uint32      | chain ID           |
| 1     | digestAlgType  | byte      | digest algorithm identifier           |
| ?     | HashLength   | VarInt    | Hash array's length           |
| ?     | Hash         | byte[]    | Hash           |

* Message validation

    omit

* Message processing logic

1. Return the response message ReactMessage
2. Find the Block according to the hash, assemble the BlockMessage, and send it to the target node.
3. Assemble the CompleteMessage and send it to the target node.

#### 4.2.6 Complete block message-BlockMessage

* Message description：Used for "block synchronization"

* Message type（cmd）

  Block

* Message format（txData）

| Length | Fields  | Type      | Remark         |
| ------ | ------- | --------- | -------------- |
| 32     | chainID  | uint32      | chain ID           |
| 1     | digestAlgType  | byte      | digest algorithm identifier           |
| ?     | preHashLength   | VarInt    | preHash array's length           |
| ?     | preHash         | byte[]    | preHash           |
| 1     | digestAlgType  | byte      | digest algorithm identifier           |
| ?     | merkleHashLength| VarInt    | merkleHash array's length           |
| ?     | merkleHash      | byte[]    | merkleHash           |
| 48     | time          | Uint48    | time           |
| 32     | height      | Uint32    | block's height           |
| 32     | txCount      | Uint32    | count of transactions           |
| ?     | extendLength| VarInt    | extend array's length           |
| ?     | extend      | byte[]    | extend           |
| 32     | publicKeyLength      | Uint32    | public key array's length           |
| ?     | publicKey      | byte[]    | public key           |
| 1     | signAlgType      | byte    | Signature algorithm type           |
| ?     | signBytesLength| VarInt    | sign array's length           |
| ?     | signBytes      | byte[]    | Signature of block           |
| ?     | txCount   | VarInt    | count of transactions           |
| 16     | type      | uint16    | transaction's type           |
| 48     | time      | uint48    | transaction's timestamp           |
| ?     | remarkLength| VarInt    | remark array's length           |
| ?     | remark      | byte[]    | remark bytes           |
| 32     | fromAssetsChainId      | Uint32    | 资产发行链的id           |
| 32     | fromAssetsId      | Uint32    | 资产id           |
| ?     | fromAddress      | VarChar    | 转出账户地址           |
| 32     | toAssetsChainId      | Uint32    | 资产发行链的id           |
| 32     | toAssetsId      | Uint32    | 资产id           |
| ?     | toAddress      | VarChar    | 转入账户地址           |
| 48     | amount      | Uint48    | 转出金额           |
| 32     | nonce      | Uint32    | 交易顺序号，递增           |
| ?     | txData      | T    | count of transactions据           |
| ?     | txSignLength| VarInt    | transaction's sign array's length           |
| ?     | txSign      | byte[]    | transaction's sign           |

* Message validation

    omit

* Message processing logic

1. Put into the cache queue
2. Wait for other blocks to be synchronized

#### 4.2.7 Data message not found-NotFoundMessage

* Message description：A generic message for an asynchronous request that marks the target node not finding the corresponding information.

* Message type（cmd）

  NotFound

* Message format（txData）

| Length | Fields  | Type      | Remark         |
| ------ | ------- | --------- | -------------- |
| 32     | chainID  | uint32      | chain ID           |
| 1     | msgType        | byte    | Data type not found           |
| 1     | digestAlgType  | byte      | digest algorithm identifier           |
| ?     | HashLength   | VarInt    | Hash array's length           |
| ?     | Hash         | byte[]    | Hash           |

* Message validation

    omit

* Message processing logic

    omit

#### 4.2.8 Response message-ReactMessage

* Message description：A generic message, used for asynchronous requests, to flag that the target node receives the request and is processing it.

* Message type（cmd）

  React

* Message format（txData）

| Length | Fields  | Type      | Remark         |
| ------ | ------- | --------- | -------------- |
| 32     | chainID  | uint32      | chain ID           |
| 1     | digestAlgType  | byte      | digest algorithm identifier           |
| ?     | HashLength   | VarInt    | Hash array's length           |
| ?     | Hash         | byte[]    | Hash           |

* Message validation

    omit

* Message processing logic

1. Find the asynchronous request of the source node cache according to the chainID and hash, and set the processing result flag to wait.

#### 4.2.9 Request completion message-CompleteMessage

* Message description：A generic message for asynchronous requests that marks the end of asynchronous request processing.

* Message type（cmd）

  Complete

* Message format（txData）

| Length | Fields  | Type      | Remark         |
| ------ | ------- | --------- | -------------- |
| 32     | chainID  | uint32      | chain ID           |
| 1     | digestAlgType  | byte      | digest algorithm identifier           |
| ?     | HashLength   | VarInt    | Hash array's length           |
| ?     | Hash         | byte[]    | Hash           |
| 1     | success  | byte      | flag           |

* Message validation

    omit

* Message processing logic

1. Find the asynchronous request of the source node cache according to the chainID and hash, and set the processing result flag to complete.

#### 4.2.10 Get the message of the transaction list-GetTxGroupRequest

* Message description：Used for "forwarding blocks"

* Message type（cmd）

  GetTxGroup

* Message format（txData）

| Length | Fields  | Type      | Remark         |
| ------ | ------- | --------- | -------------- |
| 32     | chainID  | uint32      | chain ID           |
| ?     | ArrayLength   | VarInt    | Hash list length           |
| 1     | digestAlgType  | byte      | digest algorithm identifier           |
| ?     | HashLength   | VarInt    | Hash array's length           |
| ?     | Hash         | byte[]    | Hash           |

* Message validation

    omit

* Message processing logic

1. After receiving the message, the target node takes out the hashList, traverses the hashList, obtains Tx according to txHash, assembles TxGroupMessage, and sends it to the source node.

#### 4.2.11 Transaction list message-TxGroupMessage

* Message description：Used for "forwarding blocks"

* Message type（cmd）

  TxGroup

* Message format（txData）

| Length | Fields  | Type      | Remark         |
| ------ | ------- | --------- | -------------- |
| 32     | chainID  | uint32      | chain ID           |
| 1     | digestAlgType  | byte      | digest algorithm identifier           |
| ?     | requestHashLength   | VarInt    | requestHash array's length           |
| ?     | requestHash         | byte[]    | requestHash           |
| ?     | txCount   | VarInt    | count of transactions           |
| 16     | type      | uint16    | transaction's type           |
| 48     | time      | uint48    | transaction's timestamp           |
| ?     | remarkLength| VarInt    | 备注 array's length           |
| ?     | remark      | byte[]    | 备注           |
| 32     | fromAssetsChainId      | Uint32    | 资产发行链的id           |
| 32     | fromAssetsId      | Uint32    | 资产id           |
| ?     | fromAddress      | VarChar    | 转出账户地址           |
| 32     | toAssetsChainId      | Uint32    | 资产发行链的id           |
| 32     | toAssetsId      | Uint32    | 资产id           |
| ?     | toAddress      | VarChar    | 转入账户地址           |
| 48     | amount      | Uint48    | 转出金额           |
| 32     | nonce      | Uint32    | Transaction sequence number, increment           |
| ?     | txData      | T    | count of transactions据           |
| ?     | txSignLength| VarInt    | transaction's sign array's length           |
| ?     | txSign      | byte[]    | transaction's sign           |

* Message validation

    omit

* Message processing logic

    omit

## Chapter 5. The module configuration

```
{
    {
        "name": "serverIp",
        "remark": "service ip",
        "changable": "true",
        "default": "127.0.0.1"
    },
    {
        "name": "serverPort",
        "remark": "service port",
        "changable": "true",
        "default": ""
    },
    {
        "name": "blockSize",
        "remark": "block size",
        "changable": "false",
        "default": "3m"
    },
    {
        "name": "resetTime",
        "remark": "When the block height is not updated for a long time, the available nodes are reacquired",
        "changable": "true",
        "default": "180"
    },
    {
        "name": "forkCount",
        "remark": "When the fork chain is higher than the main chain, switch",
        "changable": "false",
        "default": "3"
    },
    {
        "name": "cacheSize",
        "remark": "forked chain cache size",
        "changable": "true",
        "default": "50m"
    },
    {
        "name": "heightRange",
        "remark": "Cache to the height range of the forked chain",
        "changable": "false",
        "default": "1000"
    },
    {
        "name": "maxRollback",
        "remark": "How many blocks are rolled back at most each time",
        "changable": "true",
        "default": "20"
    },
    {
        "name": "consistencyNodePercent",
        "remark": "The lowest percentage of consistent nodes available, below this number of unsynchronized blocks",
        "changable": "false",
        "default": "80"
    },
    {
        "name": "minNodeAmount",
        "remark": "The minimum number of available nodes, lower than this number of unsynchronized blocks",
        "changable": "false",
        "default": "10"
    },
    {
        "name": "downloadNumber",
        "remark": "How many blocks are downloaded from one node each time during synchronization",
        "changable": "true",
        "default": "20"
    },
    {
        "name": "extendMaxSize",
        "remark": "block header extension field maximum value",
        "changable": "false",
        "default": "1024"
    }
}
```

## Chapter 6. Java's design

- Block Object design
> | `field name`          | `field type` | `instruction`     |
> | ------------------- | ---------- | ---------- |
> | blockHeader             | BlockHeader     | block header   |
> | transactions | List<Transaction>     | transaction's list |

- SmallBlock Object design
> | `field name`          | `field type` | `instruction`     |
> | ------------------- | ---------- | ---------- |
> | blockHeader             | BlockHeader     | block header   |
> | transactions | List<String>     | transaction's hash list |
> | subTxList | List<Transaction>     | Transactions that other nodes must not have (such as consensus reward transactions, red and yellow card transactions, etc. |

- BlockHeader Object design
> | `field name`          | `field type` | `instruction`     |
> | ------------------- | ---------- | ---------- |
> | chainId             | long     | chain ID   |
> | hash             | String     | block hash   |
> | preHash             | String     | pre block hash   |
> | merkleHash             | String     | block's MerkleHash   |
> | height             | int     | block's height   |
> | size             | short     | block's size   |
> | time             | long     | block's packing timestamp   |
> | txCount             | short     | count of transactions   |
> | packingAddress             | String     | address of packing   |
> | extend             | byte[]     | Extension field   |
> | blockSignature             | BlockSignature     | Signature of block   |

- BlockSignature Object design
> | `field name`          | `field type` | `instruction`     |
> | ------------------- | ---------- | ---------- |
> | signData             | String     | Signature of block   |
> | publicKey | byte[]     | public key |

## Chapter 7. additional content
