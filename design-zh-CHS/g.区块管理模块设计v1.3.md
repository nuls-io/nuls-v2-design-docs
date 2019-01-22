# 区块管理模块设计文档

[TOC]

## 一、总体描述

### 1.1 模块概述

#### 1.1.1 为什么要有《区块管理》模块

​	区块链上所有交易数据都保存在区块中，所以要有一个模块负责区块的存储与管理，以便其他模块对区块中数据进行验证、业务处理时可以获取到区块。

​	区块链程序初次启动时，需要同步网络上的最新区块到本地，这个过程一般耗时较长，且同步未完成时不能发起交易，所以适合由单独模块来完成该工作。

​	综上所述，为其他模块提供统一的区块数据服务是必要的，也能更好地把区块的管理与区块的具体业务进行分离，用到区块的模块不必关心区块的获取细节。

#### 1.1.2 《区块管理》要做什么

​	系统启动时，判断本地区块高度是否达到网络上大多数节点的最新高度，如果没有达到，要从网络上下载区块到本地，进行区块的验证，验证通过后，保存到本地数据库，这叫区块的同步。

​	区块同步完成后，系统开始正常运行，分下面两种情况讨论
- 如果是自身节点打包区块，共识模块会把打包好的区块交给区块管理模块，区块管理模块会负责进行区块验证、区块保存、区块广播，并且要响应网络上的其他节点发起的获取该区块的请求。
- 如果是其他共识节点打包区块，本地节点会收到网络上发来的转发区块消息，此时要从其他节点获取区块信息，进行验证并保存，保存完成后再次转发该区块，让区块在全网传播。

异常情况下，区块验证不通过，新区块无法与主链上最后一个区块相连，则把该区块视为分叉区块或孤儿区块，放入分叉链集合维护。当发现有一条分叉链A比主链B高度更高时，进行链切换，以分叉链A为最新的链
，原主链B回滚，并进入分叉链集合维护。

    对外提供区块头、区块查询服务

#### 1.1.3 《区块管理》在系统中的定位

区块管理是底层模块之一，以下分功能讨论模块依赖情况

依赖

* 区块同步-依赖网络模块的通讯接口，依赖工具模块的序列化工具
* 区块存储、回滚-依赖工具模块的数据库存储工具、共识模块、交易管理模块
* 区块转发-依赖网络模块的广播消息接口

被依赖

* 整个系统可以发起交易-区块同步
* 共识模块:区块详细验证、打包-区块查询、区块保存、区块广播、区块回滚

### 1.2 架构图

![](./image/block-module/block-module.png)

## 二、功能设计

### 2.1 功能架构图

![](image/block-module/block-functions.png)

1. 提供api，进行区块存储、查询、回滚的操作
2. 从网络上同步最新区块，进行初步验证、分叉验证，如果没有分叉，调用共识模块进行共识验证，调用交易模块进行交易验证，全部验证通过后保存到本地。
3. 区块同步、广播、转发消息的处理
4. 分叉区块的判断、存储
5. 孤儿区块的判断、存储
6. 分叉链维护、切换
7. 孤儿链维护、切换

### 2.2 模块服务

#### 2.2.1 获取本地最新区块头

* 接口说明

1. 根据链ID、缓存的最新区块高度查询DB得到最新区块头HASH
2. 根据HASH查询DB得到区块头byte数组
3. 反序列化为区块头对象

* 请求示例

    ```
    {
      "cmd": "bestBlockHeader",
      "minVersion":"1.1",
      "params": ["888"]
    }
    ```

* 请求参数说明

| index | parameter | required | type    | description |
| ----- | --------- | -------- | ------- | :---------: |
| 0     | chainId   | true     | Long  |   链ID    |

* 返回示例

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
    
* 返回字段说明
  
| parameter | type      | description                                |
| --------- | --------- | ------------------------------------------ |
| chainId      | Long    | 链ID                                |
| hash      | String    | 区块HASH                                |
| preHash   | String    | 上一区块HASH                              |
| merkleHash   | String    | 区块MerkleHash                              |
| height   | Long | 区块高度                              |
| size   | Integer    | 区块大小                              |
| time   | Long    | 区块打包时间                              |
| txCount   | Integer    | 交易数                              |
| packingAddress   | String    | 打包地址                              |
| reward   | Long    | 共识奖励                              |
| fee   | Long | 手续费                             |
| extend   | String   | 扩展字段,HEX,包含roundIndex、roundStartTime、consensusMemberCount、packingIndexOfRound、stateRoot                              |
| scriptSig   | String    | 区块签名                              |

#### 2.2.2 获取本地最新区块

* 接口说明:

1. 根据链ID获取本地最新区块头
2. 根据区块头高度查询DB得到交易HASH列表
3. 根据HASH列表从交易管理模块获取交易数据
4. 组装成block对象

* 请求示例

    ```
    {
      "cmd": "bestBlock",
      "minVersion":"1.1",
      "params": ["888"]
    }
    ```

* 请求参数说明

| index | parameter | required | type    | description |
| ----- | --------- | -------- | ------- | :---------: |
| 0     | chainId   | true     | Long  |   链ID    |

* 返回示例 

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
        	}, //区块头
        	"transactions": [
        	    {
                    "chainId": "888", //链Id
                    "height": "1", //区块高度
                    "hash": "1", //交易HASH
                    "remark": "1", //交易备注
                    "size": "1", //交易大小
                    "time": "1", //交易时间
                    "type": "1", //交易类型
                    "transactionSignature": "1", //交易签名
                    "coinData": {
                        "from" : [
                            {
                                "fromAssetsChainId":""//资产发行链的id  
                                "fromAssetsId":""//资产id
                                "fromAddress":""//转出账户地址
                                "amount":""//转出金额
                                "nonce":""//交易顺序号，递增
                            },{...}
                        ]
                        "to" : [
                            {
                                "toAssetsChainId":""//资产发行链的id  
                                "toAssetsId":""//资产id
                                "toAddress":""//转出账户地址
                                "amount":""//转出金额
                                "locktime":""
                            },{...}
                        ]
                    }
                    "txData": XXXX, //交易数据 HEX
        	    },
        	    {...}
        	] //交易列表
        }
    }
    ```

* 返回字段说明

    略

#### 2.2.3 根据高度获取区块头

* 接口说明

1. 根据链ID、高度查询DB得到最新区块头HASH
2. 根据HASH查询DB得到区块头byte数组
3. 反序列化为区块头对象

* 请求示例

    ```
    {
      "cmd": "getBlockHeaderByHeight",
      "minVersion":"1.1",
      "params": ["111","888"]
    }
    ```

* 请求参数说明

| index | parameter | required | type    | description |
| ----- | --------- | -------- | ------- | :---------: |
| 0     | chainId   | true     | Long  |   链ID    |
| 1     | height   | true     | Long  |   区块高度    |

* 返回示例

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
    
* 返回字段说明
  
| parameter | type      | description                                |
| --------- | --------- | ------------------------------------------ |
| chainId      | Long    | 链Id                                |
| hash      | String    | 区块HASH                                |
| preHash   | String    | 上一区块HASH                              |
| merkleHash   | String    | 区块MerkleHash                              |
| height   | Long | 区块高度                              |
| size   | Integer    | 区块大小                              |
| time   | Long    | 区块打包时间                              |
| txCount   | Integer    | 交易数                              |
| packingAddress   | String    | 打包地址                              |
| reward   | Long    | 共识奖励                              |
| fee   | Long | 手续费                             |
| extend   | String   | 扩展字段,HEX,包含roundIndex、roundStartTime、consensusMemberCount、packingIndexOfRound、stateRoot                              |
| scriptSig   | String    | 区块签名                              |

#### 2.2.4 根据高度获取区块

* 接口说明:

1. 根据链ID、高度获取区块头
2. 根据区块头高度查询DB得到交易HASH列表
3. 根据HASH列表从交易管理模块获取交易数据
4. 组装成block对象

* 请求示例

    ```
    {
      "cmd": "bl_getBlockByHeight",
      "minVersion":"1.1",
      "params": ["111","888"]
    }
    ```

* 请求参数说明

| index | parameter | required | type    | description |
| ----- | --------- | -------- | ------- | :---------: |
| 0     | chainId   | true     | Long  |   链ID    |
| 1     | height   | true     | Long  |   区块高度    |

* 返回示例 

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
        	}, //区块头
        	"transactions": [
        	    {
                    "chainId": "888",//链ID
                    "height": "1", //区块高度
                    "hash": "1", //交易HASH
                    "remark": "1", //交易备注
                    "size": "1", //交易大小
                    "time": "1", //交易时间
                    "type": "1", //交易类型
                    "transactionSignature": "1", //交易签名
                    "coinData": {
                        "from" : [
                            {
                                "fromAssetsChainId":""//资产发行链的id  
                                "fromAssetsId":""//资产id
                                "fromAddress":""//转出账户地址
                                "amount":""//转出金额
                                "nonce":""//交易顺序号，递增
                            },{...}
                        ]
                        "to" : [
                            {
                                "toAssetsChainId":""//资产发行链的id  
                                "toAssetsId":""//资产id
                                "toAddress":""//转出账户地址
                                "amount":""//转出金额
                                "nonce":""//交易顺序号，递增
                            },{...}
                        ]
                    }
                    "txData": XXXX, //交易数据 HEX
        	    },
        	    {...}
        	], //交易列表
        }
    }
    ```

* 返回字段说明

    略

#### 2.2.5 根据HASH获取区块头

* 接口说明

1. 根据链ID、HASH查询DB得到区块头byte数组
2. 反序列化为区块头对象

* 请求示例

    ```
    {
      "cmd": "bl_getBlockHeaderByHash",
      "minVersion":"1.1",
      "params": ["888","aaa"]
    }
    ```

* 请求参数说明

| index | parameter | required | type    | description |
| ----- | --------- | -------- | ------- | :---------: |
| 0     | chainId   | true     | Long  |   链ID    |
| 1     | hash   | true     | String  |   区块hash    |

* 返回示例

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
    
* 返回字段说明
  
| parameter | type      | description                                |
| --------- | --------- | ------------------------------------------ |
| chainId      | Long    | 链ID                                |
| hash      | String    | 区块HASH                                |
| preHash   | String    | 上一区块HASH                              |
| merkleHash   | String    | 区块MerkleHash                              |
| height   | Long | 区块高度                              |
| size   | Integer    | 区块大小                              |
| time   | Long    | 区块打包时间                              |
| txCount   | Integer    | 交易数                              |
| packingAddress   | String    | 打包地址                              |
| reward   | Long    | 共识奖励                              |
| fee   | Long | 手续费                             |
| extend   | String   | 扩展字段,HEX,包含roundIndex、roundStartTime、consensusMemberCount、packingIndexOfRound、stateRoot                              |
| scriptSig   | String    | 区块签名                              |

#### 2.2.6 根据HASH获取区块

* 接口说明:

1. 根据链ID、hash获取区块头
2. 根据区块头高度查询DB得到交易HASH列表
3. 根据HASH列表从交易管理模块获取交易数据
4. 组装成block对象

* 请求示例

    ```
    {
      "cmd": "bl_getBlockByHash",
      "minVersion":"1.1",
      "params": ["888","aaa"]
    }
    ```

* 请求参数说明

| index | parameter | required | type    | description |
| ----- | --------- | -------- | ------- | :---------: |
| 0     | chainId   | true     | Long  |   链ID    |
| 1     | hash   | true     | String  |   区块hash    |

* 返回示例 

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
        	}, //区块头
        	"transactions": [
        	    {
        	        "chainId": "888",
                    "height": "1", //区块高度
                    "hash": "1", //交易HASH
                    "remark": "1", //交易备注
                    "size": "1", //交易大小
                    "time": "1", //交易时间
                    "type": "1", //交易类型
                    "transactionSignature": "1", //交易签名
                    "coinData": {
                        "from" : [
                            {
                                "fromAssetsChainId":""//资产发行链的id  
                                "fromAssetsId":""//资产id
                                "fromAddress":""//转出账户地址
                                "amount":""//转出金额
                                "nonce":""//交易顺序号，递增
                            },{...}
                        ]
                        "to" : [
                            {
                                "toAssetsChainId":""//资产发行链的id  
                                "toAssetsId":""//资产id
                                "toAddress":""//转出账户地址
                                "amount":""//转出金额
                                "nonce":""//交易顺序号，递增
                            },{...}
                        ]
                    }
                    "txData": XXXX, //交易数据 HEX
        	    },
        	    {...}
        	], //交易列表
        }
    }
    ```

* 返回字段说明

    略

#### 2.2.7 获取某高度区间内区块头

* 接口说明

1. 令queryHash=endHash
2. 根据链ID、queryHash查询DB得到区块头byte数组
3. 反序列化为区块头对象blockHeader，添加到List中作为返回值
4. 如果blockHeader.hash!=startHash，令queryHash=blockHeader.preHash，重复第2步
5. 返回List

* 请求示例

    ```
    {
      "cmd": "bl_getBlockHeaderBetweenHeights",
      "minVersion":"1.1",
      "params": ["888",111","111"]
    }
    ```

* 请求参数说明

| index | parameter | required | type    | description |
| ----- | --------- | -------- | ------- | :---------: |
| 0     | chainId   | true     | Long  |   链ID    |
| 1     | startHeight   | true     | Long  |   起始高度    |
| 2     | endHeight   | true     | Long  |   结束高度    |

* 返回示例

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
            "list" : [
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
    
* 返回字段说明
  
| parameter | type      | description                                |
| --------- | --------- | ------------------------------------------ |
| chainId      | Long    | 链ID                                |
| hash      | String    | 区块HASH                                |
| preHash   | String    | 上一区块HASH                              |
| merkleHash   | String    | 区块MerkleHash                              |
| height   | Long | 区块高度                              |
| size   | Integer    | 区块大小                              |
| time   | Long    | 区块打包时间                              |
| txCount   | Integer    | 交易数                              |
| packingAddress   | String    | 打包地址                              |
| reward   | Long    | 共识奖励                              |
| fee   | Long | 手续费                             |
| extend   | String   | 扩展字段,HEX,包含roundIndex、roundStartTime、consensusMemberCount、packingIndexOfRound、stateRoot                              |
| scriptSig   | String    | 区块签名                              |

#### 2.2.8 获取某高度区间内区块

* 接口说明

1. 令queryHash=endHash
2. 根据链ID、queryHash查询DB得到区块byte数组
3. 反序列化为区块对象block，添加到List中作为返回值
4. 如果block.hash!=startHash，令queryHash=block.preHash，startHash，重复第2步
5. 返回List

* 请求示例

    ```
    {
      "cmd": "bl_getBlockBetweenHeights",
      "minVersion":"1.1",
      "params": ["888",111","111"]
    }
    ```

* 请求参数说明

| index | parameter | required | type    | description |
| ----- | --------- | -------- | ------- | :---------: |
| 0     | chainId   | true     | Long  |   链ID    |
| 1     | startHeight   | true     | Long  |   起始高度    |
| 2     | endHeight   | true     | Long  |   结束高度    |

* 返回示例

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
            "list" : [
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
                    }, //区块头
                    "transactions": [
                        {
                            "chainId": "888",
                            "height": "1", //区块高度
                            "hash": "1", //交易HASH
                            "remark": "1", //交易备注
                            "size": "1", //交易大小
                            "time": "1", //交易时间
                            "type": "1", //交易类型
                            "transactionSignature": "1", //交易签名
                            "coinData": {
                                "from" : [
                                    {
                                        "fromAssetsChainId":""//资产发行链的id  
                                        "fromAssetsId":""//资产id
                                        "fromAddress":""//转出账户地址
                                        "amount":""//转出金额
                                        "nonce":""//交易顺序号，递增
                                    },{...}
                                ]
                                "to" : [
                                    {
                                        "toAssetsChainId":""//资产发行链的id  
                                        "toAssetsId":""//资产id
                                        "toAddress":""//转出账户地址
                                        "amount":""//转出金额
                                        "nonce":""//交易顺序号，递增
                                    },{...}
                                ]
                            }
                            "txData": XXXX, //交易数据 HEX
                        },
                        {...}
                    ], //交易列表
               }
            ]

        }
    }
    ```
    
* 返回字段说明
  
    略

#### 2.2.9 接收最新打包区块

* 接口说明

本地节点共识模块打包后，调用此接口保存区块数据

* 请求示例

    ```
    {
      "cmd": "receivePackingBlock",
      "minVersion":"1.1",
      "params": [
      	blockhex//能用hex就用hex
      ]
    }
    ```

* 请求参数说明

    略

* 返回示例

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

* 返回字段说明

| parameter | type      | description                                |
| --------- | --------- | ------------------------------------------ |
| sync      | String    | 区块是否保存成功                          |

#### 2.2.10 运行一条链

* 接口说明

在链工厂发布一条链后，核心模块会调用区块管理模块的该接口，根据chainID初始化区块、分叉链数据库，开启chainID对应的一系列工作线程，并为运行新链做准备。

* 请求示例

    ```
    {
      "cmd": "bl_startChain",
      "minVersion":"1.1",
      "params": ["888"]
    }
    ```

* 请求参数说明

    略

* 返回示例

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
        "result": {"result": "true"}
    }
    ```

* 返回字段说明

| parameter | type      | description                                |
| --------- | --------- | ------------------------------------------ |
| result      | String    | 新链是否启动成功                          |

#### 2.2.11 停止一条链

* 接口说明

在链工厂停止一条链后，核心模块会调用区块管理模块的该接口，删除该链的缓存区块、分叉链数据，停止chainID对应的一系列工作线程。

* 请求示例

    ```
    {
      "cmd": "bl_stopChain",
      "minVersion":"1.1",
      "params": ["888"]
    }
    ```

* 请求参数说明

    略

* 返回示例

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
        "result": {"result": "true"}
    }
    ```

* 返回字段说明

| parameter | type      | description                                |
| --------- | --------- | ------------------------------------------ |
| result      | String    | 新链是否停止成功                          |

### 2.3 模块内部功能

#### 2.3.1 模块启动

* 功能说明:

  启动区块管理模块

* 流程描述

![](./image/block-module/block-module-boot.png)

- 1.RPC服务初始化
- 2.初始化通用数据库
- 3.加载配置信息
- 4.初始化各链数据库
- 5.等待依赖模块就绪
- 6.向网络模块注册消息处理类
- 7.启动同步区块线程、数据库大小监控线程、分叉链处理线程、孤儿链处理线程、孤儿链维护线程

* 依赖服务

  工具模块、内核模块、网络模块、交易管理模块

#### 2.3.2 区块存储

* 功能说明:

    存储主链上区块头数据以及分叉链、孤儿链的完整区块数据

    - 主链存储

      不同的链存到不同的表，表名加chainID后缀
              一个完整的区块由区块头和交易组成，区块头与交易分别进行存储。
      	区块头:(放在区块管理模块)
                    key(区块高度)-value(区块头hash)              		block-header-index
                    key(区块头hash)-value(完整的区块头)           	block-header
      	交易:(放在交易管理模块)

    - 分叉链、孤儿链存储

      内存中缓存所有分叉链与孤儿链对象(只记录起始高度、起始hash、结束高度、结束hash等关键信息)，在硬盘中缓存全量区块数据，如果需要分叉链切换、清理分叉链等操作，只需读取一次数据库即可
      	不同链的分叉链集合存在不同的表，表名加chainID后缀，每一个分叉链对象如下:
      		key(区块hash)-value(完整的区块数据)          	CachedBlock

* 流程描述

    略

* 依赖服务

  工具模块的数据库工具

#### 2.3.3 区块清理

- 功能说明:

  为了避免过多垃圾数据占用硬盘空间，对分叉链和孤儿链进行定时清理

- 流程描述

  1. 按照配置的最大缓存区块数量进行清理，当分叉链+孤儿链缓存的区块数量大于阈值时，进行清理
  2. 按照分叉链或孤儿链的起始高度与主链的最新高度差进行清理
  3. 按照孤儿链的年龄进行清理，孤儿链的年龄初始值为0，每经过一次孤儿链维护，但该孤儿链的链首并没有新增合法区块时，该孤儿链年龄加一，当孤儿链年龄大于阈值时，进行清理

- 依赖服务

  工具模块的数据库工具

#### 2.3.4 区块同步

* 功能说明:

  系统启动后，定时维护本地区块数据与网络上大部分节点保持一致。

  主要由一个总调度线程，三个子工作线程组成：

  ​	总调度线程：BlockSynchronizer，工作内容：统计网络上最新一致高度、检查本地区块是否需要回滚、初始化各种区块同步期间的参数、调度三个子线程

  ​	子工作线程1：BlockDownloader，工作内容：从起始高度开始，根据各下载节点的信用值分配下载任务，并启动后台下载任务BlockWorker

  ​	子工作线程1-1：BlockWorker，工作内容：组装HeightRangeMessage，发送给目标节点，并计算messageHash缓存起来，等待目标节点返回的CompleteMessage(携带的messageHash要对应)

  ​	子工作线程2：BlockCollector，工作内容：收集BlockDownloader下载到的区块，排序后放入共享队列供BlockConsumer消费

  ​	子工作线程3：BlockConsumer，工作内容：取出共享队列中的区块，依次保存

* 流程描述

    * 区块同步主流程

    ![](./image/block-module/block-synchronization.png)

    * 获取网络上可用节点列表

    ```
        1. 遍历节点，统计两个MAP，假定每个节点的(最新HASH+最新高度)是key
        2. 一个以key为主键统计次数
        3. 一个以key为主键记录持有该key的节点列表
        4. 最终统计出出现频率最大的key，就获取到当前可信的最新高度与最新hash，以及可信的节点列表
        
        举个栗子:
        现在同时连接到10个节点。其中4个节点(A,B,C,D)的最新区块高度是100，最新区块hash是aaa，其中6个节点(E,F,G,H,I,J)的最新区块高度是101，最新区块hash是bbb。
        最终返回(101，bbb,[E,F,G,H,I,J])。
    ```

    * 下载区块逻辑

    ![](./image/block-module/block-synchronization2.png)
    ```
        在正式下载区块前，要判断本地与网络是否发生分叉，是否需要回滚。以便找到准确的区块下载高度。
        以下分情况讨论:
        取上一步的结果(101，bbb,[E,F,G,H,I,J])，同时LH(N)代表本地第N块的hash，RH(N)代表网络上第N块的hash。
        1.本地高度100<网络高度101，LH(100)==RH(100)，正常，比远程节点落后，下载区块
        2.本地高度100<网络高度101，LH(100)!=RH(100)，认为本地分叉，回滚本地区块，如果LH(99)==RH(99)
        回滚结束，从99块开始下载，如果LH(99)!=RH(99)，继续回滚，重复上述逻辑。但最多回滚10个块就停止，等待下次同步，这样可以避免被恶意节点攻击，大量回滚正常区块。
        3.本地高度102>网络高度101，LH(101)==RH(101)，正常，比远程节点领先，无需下载区块
        4.本地高度102>网络高度101，LH(101)!=RH(101)，认为本地分叉，先一次性回滚到高度与远程一致，重复场景2
        5.本地高度101=网络高度101，LH(101)==RH(101)，正常，与远程节点一致，无需下载区块
        6.本地高度101=网络高度101，LH(101)!=RH(101)，认为本地分叉，重复场景2
        
        场景1、2需要额外从节点下载与本地高度一致的区块，进行hash判断
        上述需要回滚的场景，要满足可用节点数(10个)>配置，一致可用节点数(6个)占比超80%两个条件，避免节点太少导致频繁回滚。以上两个条件都不满足，清空已连接节点，重新获取可用节点。
     
        真正下载区块时，举个栗子:
        当前高度100，网络高度500，可用节点12个，一致可用节点10个，每个节点初始下载区块2个，下载时缓存队列最多允许缓存100个区块
        伪代码表示
        	空闲的下载节点队列：nodes(每个下载节点会有初始下载信用值，每下载成功一次信用值增加，信用最大值为初始值的2倍)
        	下载到的区块缓存队列：queue
            下载起始高度：startHeight = 101;
            下载结束高度：netLatestHeight = 500;
            while (startHeight <= netLatestHeight) {
                    while (queue.size() > 100) {
                        BlockDownloader wait！ cached queue size beyond config
                    }
                    获取一个可用节点
                    根据节点下载信用值计算本次下载区块数量为size
                    提交异步下载任务
                    startHeight += size;
            }
    如果某节点下载失败，则由其他节点代为下载
    考虑下载过程中，网络上其他节点还会继续生成新区块。下载结束后，需要判断本地最新区块高度与网上最新一致高度是否相同，如果相同，标志区块同步结束，如果不相同，则需要继续下载
    ```

    * 从节点下载某高度区间内的区块

    ![](./image/block-module/block-synchronization3.png)

* 依赖服务

  工具模块的数据库存储工具、RPC工具

#### 2.3.3 区块基础验证

* 功能说明:

  验证区块自身数据正确性,下载过程中验证，验证通过说明区块数据本身没有问题，验证失败则丢弃该区块

* 流程描述

    * 区块基本验证
    
    ![](./image/block-module/block-basic-validation.png)
    
    * 区块头验证
    
    ![](./image/block-module/block-basic-validation1.png)

* 依赖服务

  工具模块的数据库存储工具

#### 2.3.4 分叉块验证

* 功能说明:

  验证区块上下文正确性，下载完成后验证，验证通过说明该区块与主链相连，验证失败说明该区块分叉，进入分叉链处理逻辑

* 流程描述

    - 定义一条主链(MasterChain)，一个分叉链集合(forkChains)
    - 定义待验证区块为Block
    - 定义主链高度MG，待验证区块高度BG
    - 定义主链最新区块HASH为MH，待验证区块HASH为BH，待验证区块PREHASH为BPH
    
    - 分六种情况讨论
    - 1.MG==BG，MH==BH，说明重复收到最新主链区块，丢弃
    - 2.MG==BG，MH!=BH，说明网络分叉
        - 遍历已有分叉链集合，判断是否已存在此区块
            - 如果已存在，丢弃该区块
            - 如果不存在，新建一条分叉链
              - 判断是否能连接上其他的分叉链（若能连上，则连接到其他分叉链则视为一条链）
              - 递归判断
    - 3.MG==BG-1，MH==BPH，说明区块连续，保存到主链
    - 4.MG==BG-1，MH!=BPH，说明网络分叉，处理同第2步
    - 5.MG<BG-1，说明网络分叉，处理同第2步
    - 6.MG>BG，说明网络分叉，处理同第2步
    
    高度差1000以内缓存到磁盘，磁盘空间做大小限制，超出高度则丢弃，缓存空间满则按加入缓存时间顺序清理分叉链。
    如果是正常运行时，收到其他节点转发的区块，发现分叉了要通知共识模块给生成这个区块的节点红牌惩罚，系统启动后的同步过程中不做这个判断

  ![](./image/block-module/block-fork.png)

* 依赖服务

  工具模块的数据库存储工具

#### 2.3.5 分叉链管理

* 功能说明:

  判断分叉链与主链是否需要进行切换

* 流程描述
  - 检查是否有分叉链能链接上主链，如果有则链接
  - 取出最长的一条分叉链与主链长度对比判断是否需要切换主链
      - 如果分叉链长度比主链长度长3（配置）个区块以上则需要切换主链
      - 找到主链与最长分叉链的分叉点
      - 验证分叉链中的区块，如果验证通过继续往下执行
      - 回滚主链区块
      - 切换分叉链为主链

![](./image/block-module/block-fork-chain.png)

* 依赖服务

  工具模块的数据库存储工具

#### 2.3.5 孤儿链管理

* 功能说明:

    用来进行孤儿链与分叉链、主链的相连、分叉操作

* 流程描述

1. 使用blockHash组装HashMessage，发送给目标节点
2. 目标节点收到HashMessage后，取出hash判断是否重复，如果不重复，使用hash组装HashMessage发给源节点
3. 源节点收到GetSmallBlockMessage后，取出hash，查询SmallBlock并组装SmallBlockMessage，发给目标节点
4. 后续交互流程参考广播区块

* 依赖服务

  工具模块的数据库存储工具

#### 2.3.5 孤儿链维护

* 功能说明:

    定时尝试在孤儿链的链首请求增加区块

* 流程描述

    链与链之间的关系就两种，相连，或者分叉，
 1. 标记
 2. 复制、清除

* 依赖服务

  工具模块的数据库存储工具

#### 2.3.6 转发区块

* 功能说明:

    非出块节点保存完区块后走转发流程

* 流程描述

1. 使用blockHash组装HashMessage，发送给目标节点
2. 目标节点收到HashMessage后，取出hash判断是否重复，如果不重复，使用hash组装HashMessage发给源节点
3. 源节点收到GetSmallBlockMessage后，取出hash，查询SmallBlock并组装SmallBlockMessage，发给目标节点
4. 后续交互流程参考广播区块

* 依赖服务

  工具模块的数据库存储工具

#### 2.3.7 广播区块

* 功能说明:

  出块的节点走广播流程

* 流程描述

1. 收到共识模块的打包区块，保存成功后，根据Block组装SmallBlockMessage
2. 调用网络模块广播消息
3. 目标节点收到消息后根据txHashList判断哪些交易本地没有,再组装HashListMessage发给源节点
4. 源节点收到信息后按照hashlist组装TxGroupMessage,返回给目标节点
5. 至此完整区块数据已经发送给目标节点。

* 依赖服务

  工具模块的数据库存储工具

## 三、事件说明

### 3.1 发布的事件

#### 3.1.1 同步完成

说明:同步完成，本地区块高度与网络高度一致时，发布该事件

 event_topic : "bl_blockSyncComplete",

```
data:{
    chainId
    height
    hash
}
```

#### 3.1.2 保存区块

说明:每保存一个区块，发布该事件，初次同步时不发该事件

 event_topic : "bl_saveBlock",

```
data:{
    chainId
    height
    hash
}
```

#### 3.1.3 回滚区块

说明:每回滚一个区块，发布该事件   

 event_topic : "bl_rollbackBlock",

```
data:{
    chainId
    height
    hash
}
```

### 3.2 订阅的事件

#### 3.2.1 网络稳定

说明:网络稳定时，发布该事件   

 event_topic : "nt_networkDone",

    data:{
        chainId
    }

## 四、协议

### 4.1 网络通讯协议

    参见网络模块

### 4.2 消息协议

#### 4.2.1 摘要信息NulsDigestData

* 消息说明:基础消息，被别的业务消息引用

* 消息类型（cmd）

  略

* 消息的格式（messageBody）

| Length | Fields  | Type      | Remark         |
| ------ | ------- | --------- | -------------- |
| 1     | digestAlgType  | byte      | 摘要算法标识(0-SHA256, 1-SHA160)           |
| ?     | hashLength     | VarInt    | hash bytes length  |
| ?     | hash        | byte[]    | hash bytes           |

* 消息的验证

    略

* 消息的处理逻辑

    略

#### 4.2.2 交易信息Transaction

* 消息说明:基础消息，被别的业务消息引用

* 消息类型（cmd）

  略

* 消息的格式（messageBody）

| Length | Fields  | Type      | Remark         |
| ------ | ------- | --------- | -------------- |
| 16     | type  | Uint16      | 交易类型           |
| 48     | time   | Uint48    | 交易时间戳           |
| ?     | remark   | VarInt    | 交易备注           |
| ?     | remark      | byte[]    | 交易备注           |
| ?     | txData   | VarInt    | 交易数据           |
| ?     | txData      | byte[]    | 交易数据           |
| ?     | coinData   | VarInt    | 交易转账数据           |
| ?     | coinData      | byte[]    | 交易转账数据           |
| ?     | transactionSignature   | VarInt    | 交易签名    |
| ?     | transactionSignature      | byte[]    | 交易签名   |

* 消息的验证

    略

* 消息的处理逻辑

    略

#### 4.2.3 单个摘要消息HashMessage

* 消息说明:用于"转发区块","孤儿链维护"功能

* 消息类型（cmd）

  forward,getBlock,getsBlock

* 消息的格式（messageBody）

| Length | Fields  | Type      | Remark         |
| ------ | ------- | --------- | -------------- |
| ?     | hash  | NulsDigestData  | 结构参考4.2.1  |

* 消息的验证

    略

* 消息的处理逻辑

1. 目标节点收到消息后，先根据chainID判断缓存中hash是否重复
2. 如果重复，说明已经收到别的节点转发的SmallBlock，丢弃消息
3. 如果没有重复，用hash组装GetSmallBlockMessage，并发送给源节点

#### 4.2.4 摘要列表消息HashListMessage

* 消息说明:用于"转发区块"功能

* 消息类型（cmd）

  GetSmallBlock

* 消息的格式（messageBody） 

| Length | Fields  | Type      | Remark         |
| ------ | ------- | --------- | -------------- |
| 32     | chainID  | uint32      | 链ID           |
| 1     | digestAlgType  | byte      | 摘要算法标识           |
| ?     | hashLength      | VarInt    | 数组长度           |
| ?     | blockHash            | byte[]    | hash           |
| ?     | hashLength      | VarInt    | 数组长度           |
| 1     | digestAlgType  | byte      | 摘要算法标识           |
| ?     | hashLength        | VarInt    | 数组长度           |
| ?     | hash        | byte[]    | hash           |

* 消息的验证

    略

* 消息的处理逻辑

1. 根据chainID、hash获取SmallBlock对象
2. 组装SmallBlockMessage，并发送给源节点

#### 4.2.5 区块广播消息SmallBlockMessage

* 消息说明:用于"转发区块"、"广播区块"功能

* 消息类型（short）

  SmallBlock

* 消息的格式（messageBody）

| Length | Fields  | Type      | Remark         |
| ------ | ------- | --------- | -------------- |
| 32     | chainID  | uint32      | 链ID           |
| 1     | digestAlgType  | byte      | 摘要算法标识           |
| ?     | preHashLength   | VarInt    | preHash数组长度           |
| ?     | preHash         | byte[]    | preHash           |
| 1     | digestAlgType  | byte      | 摘要算法标识           |
| ?     | merkleHashLength| VarInt    | merkleHash数组长度           |
| ?     | merkleHash      | byte[]    | merkleHash           |
| 48     | time          | Uint48    | 时间           |
| 32     | height      | Uint32    | 区块高度           |
| 32     | txCount      | Uint32    | 交易数           |
| ?     | extendLength| VarInt    | extend数组长度           |
| ?     | extend      | byte[]    | extend           |
| 32     | publicKeyLength      | Uint32    | 公钥数组长度           |
| ?     | publicKey      | byte[]    | 公钥           |
| 1     | signAlgType      | byte    | 签名算法类型           |
| ?     | signBytesLength| VarInt    | 签名数组长度           |
| ?     | signBytes      | byte[]    | 签名           |
| ?     | txHashListLength| VarInt    | 交易hash列表数组长度           |
| 1     | digestAlgType  | byte      | 摘要算法标识           |
| ?     | txHashLength| VarInt    | 交易hash数组长度           |
| ?     | txHash      | byte[]    | 交易hash           |

* 消息的验证

    略

* 消息的处理逻辑

1. 判断区块时间戳是否大于(当前时间+10s)，如果大于这个时间，则判定为恶意提前出块，忽略该消息
2. 根据chainID、区块hash判断消息是否重复，如果重复，则忽略该消息(这里要求维护一个集合,按照chainID分类储存收到的区块hash)
3. 根据chainID、区块hash在DB中查询本地是否已经有该区块，如果已经有了，则忽略该消息
4. 验证区块头，验证失败，则忽略该消息
5. 取txHashList，判断那些tx本地没有，组装HashListMessage，发给源节点，获取没有的那些交易信息
6. 如果交易都有，组放入缓存队列，等待验证线程验证后存储

#### 4.2.6 高度区间消息HeightRangeMessage

* 消息说明:用于"同步区块"功能

* 消息类型（cmd）

  GetBlocksByHeight

* 消息的格式（messageBody）

| Length | Fields  | Type      | Remark         |
| ------ | ------- | --------- | -------------- |
| 32     | chainID  | uint32      | 链ID           |
| 32     | startHeight  | uint32      | 起始高度           |
| 32     | endHeight  | uint32      | 结束高度           |

* 消息的验证

    略

* 消息的处理逻辑

1. chainID、高度参数验证
2. 返回响应消息ReactMessage
3. 从endHeight开始查找Block,组装BlockMessage，发给目标节点
4. 查找到startHeight为止，组装CompleteMessage，发给目标节点

#### 4.2.7 完整的区块消息BlockMessage

* 消息说明:用于"区块同步"

* 消息类型（cmd）

  Block

* 消息的格式（messageBody）

| Length | Fields  | Type      | Remark         |
| ------ | ------- | --------- | -------------- |
| 32     | chainID  | uint32      | 链ID           |
| 1     | digestAlgType  | byte      | 摘要算法标识           |
| ?     | preHashLength   | VarInt    | preHash数组长度           |
| ?     | preHash         | byte[]    | preHash           |
| 1     | digestAlgType  | byte      | 摘要算法标识           |
| ?     | merkleHashLength| VarInt    | merkleHash数组长度           |
| ?     | merkleHash      | byte[]    | merkleHash           |
| 48     | time          | Uint48    | 时间           |
| 32     | height      | Uint32    | 区块高度           |
| 32     | txCount      | Uint32    | 交易数           |
| ?     | extendLength| VarInt    | extend数组长度           |
| ?     | extend      | byte[]    | extend           |
| 32     | publicKeyLength      | Uint32    | 公钥数组长度           |
| ?     | publicKey      | byte[]    | 公钥           |
| 1     | signAlgType      | byte    | 签名算法类型           |
| ?     | signBytesLength| VarInt    | 签名数组长度           |
| ?     | signBytes      | byte[]    | 区块签名           |
| 16     | type      | uint16    | 交易类型           |
| 48     | time      | uint48    | 交易时间           |
| ?     | remarkLength| VarInt    | 备注数组长度           |
| ?     | remark      | byte[]    | 备注           |
| 32     | fromCount      | Uint32    | 转出记录数           |
| 32     | fromAssetsChainId      | Uint32    | 资产发行链的id           |
| 32     | fromAssetsId      | Uint32    | 资产id           |
| ?     | fromAddress      | VarChar    | 转出账户地址           |
| 48     | amount      | Uint48    | 转出金额           |
| 32     | nonce      | Uint32    | 交易顺序号，递增           |
| 32     | toCount      | Uint32    | 转入记录数           |
| 32     | toAssetsChainId      | Uint32    | 资产发行链的id           |
| 32     | toAssetsId      | Uint32    | 资产id           |
| ?     | toAddress      | VarChar    | 转入账户地址           |
| 48     | amount      | Uint48    | 转入金额           |
| 32     | lockTIme      | Uint32    | 锁定时间           |
| ?     | txData      | T    | 交易数据           |
| ?     | txSignLength| VarInt    | 交易签名数组长度           |
| ?     | txSign      | byte[]    | 交易签名           |

* 消息的验证

    略

* 消息的处理逻辑

1. 放入缓存队列
2. 等待其他区块同步中

#### 4.2.8 请求完成消息CompleteMessage

* 消息说明:通用消息，用于异步请求，标志异步请求处理结束。

* 消息类型（cmd）

  Complete

* 消息的格式（messageBody）

| Length | Fields  | Type      | Remark         |
| ------ | ------- | --------- | -------------- |
| 32     | chainID  | uint32      | 链ID           |
| 1     | digestAlgType  | byte      | 摘要算法标识           |
| ?     | HashLength   | VarInt    | Hash数组长度           |
| ?     | Hash         | byte[]    | Hash           |
| 1     | success  | byte      | 成功标志           |

* 消息的验证

    略

* 消息的处理逻辑

1. 根据chainID、hash查找源节点缓存的异步请求，把处理结果标志设置为完成。

#### 4.2.9 交易列表的消息TxGroupMessage

* 消息说明:用于"转发区块"

* 消息类型（cmd）

  TxGroup

* 消息的格式（messageBody）

| Length | Fields  | Type      | Remark         |
| ------ | ------- | --------- | -------------- |
| 32     | chainID  | uint32      | 链ID           |
| 1     | digestAlgType  | byte      | 摘要算法标识           |
| ?     | requestHashLength   | VarInt    | requestHash数组长度           |
| ?     | requestHash         | byte[]    | requestHash           |
| ?     | txCount   | VarInt    | 交易数           |
| 16     | type      | uint16    | 交易类型           |
| 48     | time      | uint48    | 交易时间           |
| ?     | remarkLength| VarInt    | 备注数组长度           |
| ?     | remark      | byte[]    | 备注           |
| 32     | fromCount      | Uint32    | 转出记录数           |
| 32     | fromAssetsChainId      | Uint32    | 资产发行链的id           |
| 32     | fromAssetsId      | Uint32    | 资产id           |
| ?     | fromAddress      | VarChar    | 转出账户地址           |
| 48     | amount      | Uint48    | 转出金额           |
| 32     | nonce      | Uint32    | 交易顺序号，递增           |
| 32     | toCount      | Uint32    | 转入记录数           |
| 32     | toAssetsChainId      | Uint32    | 资产发行链的id           |
| 32     | toAssetsId      | Uint32    | 资产id           |
| ?     | toAddress      | VarChar    | 转入账户地址           |
| 48     | amount      | Uint48    | 转入金额           |
| 32     | lockTIme      | Uint32    | 锁定时间           |
| ?     | txData      | T    | 交易数据           |
| ?     | txSignLength| VarInt    | 交易签名数组长度           |
| ?     | txSign      | byte[]    | 交易签名           |

* 消息的验证

    略

* 消息的处理逻辑

    略

## 五、模块配置

```
[
  {
    "name": "logLevel",
    "remark": "日志级别",
    "readOnly": "false",
    "value": "DEBUG"
  },
  {
    "name": "orphanChainMaxAge",
    "remark": "孤儿链最大年龄",
    "readOnly": "true",
    "value": "10"
  },
  {
    "name": "validBlockInterval",
    "remark": "为阻止恶意节点提前出块,设置此参数,区块时间戳大于当前时间多少就丢弃该区块",
    "readOnly": "true",
    "value": "60000"
  },
  {
    "name": "blockCache",
    "remark": "同步区块时最多缓存多少个区块",
    "readOnly": "true",
    "value": "10000"
  },
  {
    "name": "smallBlockCache",
    "remark": "系统正常运行时最多缓存多少个从别的节点接收到的小区块",
    "readOnly": "true",
    "value": "100"
  },
  {
    "name": "chainSwtichThreshold",
    "remark": "分叉链切换为主链的高度差阈值",
    "readOnly": "true",
    "value": "1"
  },
  {
    "name": "chainName",
    "remark": "链名称",
    "readOnly": "true",
    "value": "nuls2.0"
  },
  {
    "name": "chainId",
    "remark": "链ID",
    "readOnly": "true",
    "value": "1"
  },
  {
    "name": "blockMaxSize",
    "remark": "区块大小最大值",
    "readOnly": "false",
    "value": "2097152"
  },
  {
    "name": "resetTime",
    "remark": "持续多长时间区块高度没有更新时,就重新获取可用节点",
    "readOnly": "true",
    "value": "180"
  },
  {
    "name": "cacheSize",
    "remark": "分叉链、孤儿链缓存区块最大数量",
    "readOnly": "true",
    "value": "100"
  },
  {
    "name": "heightRange",
    "remark": "缓存到分叉链的高度区间",
    "readOnly": "false",
    "value": "1000"
  },
  {
    "name": "maxRollback",
    "remark": "每次最多回滚多少区块",
    "readOnly": "true",
    "value": "10"
  },
  {
    "name": "consistencyNodePercent",
    "remark": "一致可用节点最低比例,低于此数不同步区块",
    "readOnly": "false",
    "value": "60"
  },
  {
    "name": "minNodeAmount",
    "remark": "最小可用节点个数,低于此数不同步区块",
    "readOnly": "false",
    "value": "3"
  },
  {
    "name": "downloadNumber",
    "remark": "同步时,每次从一个节点下载多少区块",
    "readOnly": "true",
    "value": "20"
  },
  {
    "name": "extendMaxSize",
    "remark": "区块头扩展字段最大值",
    "readOnly": "false",
    "value": "1024"
  }
]
```

## 六、Java特有的设计

- Block对象设计
> | `字段名称`          | `字段类型` | `说明`     |
> | ------------------- | ---------- | ---------- |
> | blockHeader        | BlockHeader     | 区块头   |
> | transactions | List<Transaction>     | 交易列表 |

- SmallBlock对象设计
> | `字段名称`          | `字段类型` | `说明`     |
> | ------------------- | ---------- | ---------- |
> | blockHeader             | BlockHeader     | 区块头   |
> | transactions | List<String>     | 所有交易HASH列表 |
> | subTxList | List<Transaction>     | 其他节点一定没有的交易(如共识奖励交易、红黄牌交易等) |

- BlockHeader对象设计
> | `字段名称`          | `字段类型` | `说明`     |
> | ------------------- | ---------- | ---------- |
> | hash             | String     | 区块HASH   |
> | preHash             | String     | 上一区块HASH   |
> | merkleHash             | String     | 区块MerkleHash   |
> | height             | int     | 区块高度   |
> | time             | long     | 区块打包时间   |
> | txCount             | short     | 交易数   |
> | extend             | byte[]     | 扩展字段   |
> | blockSignature             | BlockSignature     | 区块签名   |

- BlockSignature对象设计
> | `字段名称`          | `字段类型` | `说明`     |
> | ------------------- | ---------- | ---------- |
> | signData            | String     | 区块签名   |
> | publicKey           | byte[]     | 公钥 |

- Chain对象设计
> | `字段名称`          | `字段类型` | `说明`     |
> | ------------------- | ---------- | ---------- |
> | parent              | Chain     | 父链   |
> | sons                | SortedSet<Chain>     | 子链集合 |
> | chainId             | int     | 链ID   |
> | previousHash        | NulsDigestData     | 链上起始区块的previousHash   |
> | startHeight         | long     | 链起始高度   |
> | startHashCode       | int     | 链起始hash转换int(排序用)   |
> | endHeight           | long     | 链结束高度   |
> | hashList            | LinkedList     | 链上区块hash列表   |
> | type                | ChainTypeEnum     | 链类型   |
> | age                 | AtomicInteger     | 链年龄(孤儿链清理使用)   |

## 七、补充内容
