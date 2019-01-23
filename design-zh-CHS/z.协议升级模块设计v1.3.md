# 协议升级模块设计文档

[TOC]

## 一、总体描述

### 1.1 模块概述

#### 1.1.1 为什么要有《协议升级》模块

​	不同的协议版本支持的交易类型、消息类型不同，为了管理区块链网络的版本，需要提供完善的版本管理功能。

#### 1.1.2 《协议升级》要做什么

- 解析区块头中的版本信息，进行动态统计、升级、回退
- 为其他模块提供版本支持的交易类型、消息类型、版本信息查询服务

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

  根据链ID查询DB得到主网版本信息

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
                "major": "1",
                "minor": "1",
                "percent": "80",
                "slice": "100",
                "waitCount": "240"
            }
        }
    }
    ```

* 返回字段说明
  
| parameter | type      | description                                |
| --------- | --------- | ------------------------------------------ |
| chainId      | Integer | 链ID                                |
| major | Integer | 主版本号                          |
| minor | Integer | 次版本号                          |
| percent | Integer | 有效比例                          |
| slice | Integer | 最小统计片断长度                    |
| waitCount | Integer    | 连续确认次数                        |

#### 2.2.2 获取当前本地版本信息

* 接口说明

  根据链ID查询DB得到本地版本信息

* 请求示例

    ```
    {
      "cmd": "currentLocalVersion",
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
            "versionInfo": {
                "major": "1",
                "minor": "1",
                "percent": "80",
                "slice": "100",
                "waitCount": "240"
            }
        }
    }
    ```

* 返回字段说明

    参考2.2.1

    #### 2.2.3 根据区块高度获取版本统计信息

    - 接口说明

      根据链ID查询DB得到本地版本信息

    - 请求示例

      ```
      {
        "cmd": "statisticsInfo",
        "minVersion":"1.1",
        "params": ["888", "888"]
      }
      ```

    - 请求参数说明

    | index | parameter | required | type | description |
    | ----- | --------- | -------- | ---- | :---------: |
    | 0     | chainId   | true     | Long |    链ID     |
    | 1     | height    | true     | Long |  区块高度   |

    - 返回示例 

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
              "statisticsInfo": {
                  "major": "1",
                  "minor": "1",
                  "percent": "80",
                  "slice": "100",
                  "count": "240"
              }
          }
      }
      ```

    - 返回字段说明

      参考2.2.1

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
* 依赖服务

  工具模块、内核模块、网络模块、交易管理模块

#### 2.3.2 版本升级

* 功能说明:

    每保存一个区块或者回滚一个区块时，都会读取区块头中的版本号，动态统计版本比例信息，决定是否进行

* 流程描述

    每100个区块为一个统计区间，统计区块头中版本号的分布比例

    - 版本占比大于某个阈值(必须大于50%)时，就能确定该统计区间内的版本号，当连续260个统计区间内的占比最大版本号保持连续时，主网执行协议升级。
    - 如果没有版本号占比大于阈值，沿用当前生效版本号作为当前区间的版本号
    - 如果中途有统计区间版本号波动，则重新开始统计

* 依赖服务

  工具模块的数据库工具

#### 2.3.3 版本回退

- 功能说明:

  版本回退的逻辑类似于升级

- 流程描述

  

- 依赖服务

  工具模块的数据库工具

#### 2.3.4 版本信息推送

- 功能说明:

  每个统计区间的最后一个区块接收完毕时，主动通知各模块当前生效的协议版本信息，主要是通知交易管理模块有效的交易类型及有效的交易验证器，通知网络模块有效的消息类型及消息处理器。

- 流程描述

  

- 依赖服务

  工具模块的数据库工具

## 三、事件说明

### 3.1 发布的事件

#### 3.1.1 版本升级

说明:协议版本升级，发布该事件

 event_topic : "versionUpadte",

```
data:{
    chainId
    major
    minor
    height
}
```

#### 3.1.2 版本回退

说明:协议版本回退，发布该事件

 event_topic : "versionRollback",

```
data:{
    chainId
    major
    minor
    height
}
```

### 3.2 订阅的事件

​	略

## 四、协议

### 4.1 网络通讯协议

​	略

### 4.2 消息协议

​	略

## 五、模块配置

```
[
  {
    "name": "supportTxTypes",
    "remark": "支持的交易类型",
    "readOnly": "true",
    "value": "1,2,3,4,5"
  },
  {
    "name": "supportMessageTypes",
    "remark": "支持的消息类型",
    "readOnly": "true",
    "value": "1,2,3,4,5"
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
