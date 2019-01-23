# 协议升级模块设计文档

[TOC]

## 一、总体描述

### 1.1 模块概述

#### 1.1.1 为什么要有《协议升级》模块

​	不同的协议版本支持的交易类型、消息类型不同，为了管理区块链网络的版本，需要提供完善的版本管理功能。

#### 1.1.2 《协议升级》要做什么

- 解析区块头中的版本信息，进行动态升级、回退
- 为其他模块提供版本信息查询服务

#### 1.1.3 《协议升级》在系统中的定位

协议升级是底层模块之一，以下分功能讨论模块依赖情况

依赖

* 区块管理模块-初始化本地协议版本信息

被依赖

* 

### 1.2 架构图

​	补充图片

## 二、功能设计

### 2.1 功能架构图

​	补充图片

### 2.2 模块服务

#### 2.2.1 获取当前主网版本信息

* 接口说明

1. 根据链ID查询DB得到主网版本信息

* 请求示例

    ```
    {
      "cmd": "currentMainnetVersion",
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
            "versionInfo": {
                "major": "888",
                "minor": "888",
                "versionInfo": "xxxxxxx",
            }
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
| time   | Long    | 区块打包时间                              |
| txCount   | Integer    | 交易数                              |
| packingAddress   | String    | 打包地址                              |
| reward   | Long    | 共识奖励                              |
| fee   | Long | 手续费                             |
| extend   | String   | 扩展字段,HEX,包含roundIndex、roundStartTime、consensusMemberCount、packingIndexOfRound、stateRoot                              |

#### 2.2.2 获取当前本地版本信息

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
            "chainId": "888",
            "hash": "xxxxxxx",
            "preHash": "xxxxxxx",
            "merkleHash": "1",
        }
    }
    ```

* 返回字段说明

    略

### 2.3 模块内部功能

#### 2.3.1 模块启动

* 功能说明:

  启动协议升级模块

* 流程描述

  补充图片

- 1.RPC服务初始化
* 2.初始化通用数据库
* 3.加载配置信息
* 4.初始化各链数据库
* 5.等待依赖模块就绪
* 6.向网络模块注册消息处理类
* 7.启动同步区块线程、数据库大小监控线程、分叉链处理线程、孤儿链处理线程、孤儿链维护线程

* 依赖服务

  工具模块、内核模块、网络模块、交易管理模块

#### 2.3.2 版本升级

* 功能说明:

    存储主链上区块头数据以及分叉链、孤儿链的完整区块数据

    - 主链存储

      不同的链存到不同的表，表名加chainID后缀
              一个完整的区块由区块头和交易组成，区块头与交易分别进行存储。
      	区块头:(放在协议升级模块)
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

#### 2.3.3 版本回退

- 功能说明:

  为了避免过多垃圾数据占用硬盘空间，对分叉链和孤儿链进行定时清理

- 流程描述

  1. 按照配置的最大缓存区块数量进行清理，当分叉链+孤儿链缓存的区块数量大于阈值时，进行清理
  2. 按照分叉链或孤儿链的起始高度与主链的最新高度差进行清理
  3. 按照孤儿链的年龄进行清理，孤儿链的年龄初始值为0，每经过一次孤儿链维护，但该孤儿链的链首并没有新增合法区块时，该孤儿链年龄加一，当孤儿链年龄大于阈值时，进行清理

- 依赖服务

  工具模块的数据库工具

## 三、事件说明

### 3.1 发布的事件

#### 3.1.1 版本升级

说明:同步完成，本地区块高度与网络高度一致时，发布该事件

 event_topic : "bl_blockSyncComplete",

```
data:{
    chainId
    height
    hash
}
```

#### 3.1.2 版本回退

说明:每保存一个区块，发布该事件，初次同步时不发该事件

 event_topic : "bl_saveBlock",

```
data:{
    chainId
    height
    hash
}
```

### 3.2 订阅的事件

​	略

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
    "name": "extendMaxSize",
    "remark": "区块头扩展字段最大值",
    "readOnly": "false",
    "value": "1024"
  }
]
```

## 六、Java特有的设计

- BlockChainVersion对象设计
> | `字段名称`          | `字段类型` | `说明`     |
> | ------------------- | ---------- | ---------- |
> | blockHeader        | BlockHeader     | 区块头   |
> | transactions | List<Transaction>     | 交易列表 |

## 七、补充内容
