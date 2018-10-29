# 链管理模块设计文档

[TOC]

## 一、总体描述

### 1.1 模块概述

#### 1.1.1 为什么要有《链管理》模块

[^说明]: 介绍模块的存在的原因

在NULS 1.0中，只有一条链（NULS主网），因此不需要链管理模块。

在NULS 2.0中，NULS主网上可以注册其他友链信息，包括：

- NULS生态圈中的链：与NULS主网使用同一套代码衍生出来。
- 其他链：比特币、以太坊等

《链管理》模块用来管理所有加入NULS主网的友链的信息



名词解释：

- NULS主网：不同于NULS 1.0，是独立运行的另一条链，也称之为NULS 2.0。
  《链管理》是NULS主网的其中一个模块
- 友链：在NULS主网上注册的其他链



假设1：友链A，其拥有资产A

假设2：友链B，其拥有资产B

- 跨链交易：
  - 友链A把资产A转到友链B
  - 友链B内部转移资产A
  - 友链B把资产A转回到友链A
- 非跨链交易：
  - 友链A内部转移资产A
  - 友链B内部转移资产B

备注：不管是跨链交易，还是非跨链交易，都需要到NULS主链进行确认。




#### 1.1.2 《链管理》要做什么

[^说明]: 模块要做些什么事情，达到什么目的，目标是让非技术人员了解要做什么事情

《链管理》模块用来管理加入NULS主网的链的基本信息，包括：

* 注册一条新的友链
* 销毁已经存在的友链
* 查询友链信息
* 特定友链增加资产类型
* 特定友链销毁资产类型



#### 1.1.3 《链管理》在系统中的定位

[^说明]: 模块在系统中的定位，是什么角色，依赖哪些模块做哪些事情，可以被依赖用于做哪些事情

《链管理》强依赖的模块：

- 核心模块
- 网络模块
- 交易管理模块

《链管理》弱依赖的模块：

- 事件总线模块



依赖《链管理》的模块：

- 



### 1.2 架构图

[^说明]: 图形说明模块的层次结构、组件关系，并通过文字进行说明

## 二、功能设计

### 2.1 功能架构图

[^说明]: 说明模块的功能设计，可以有层级关系，可以通过图形的形式展示，并用文字进行说明。

![](image/chainModule/structure.png)

### 2.2 模块服务

[^说明]: 这里说明该模块对外提供哪些服务，每个服务的功能说明、流程描述、接口定义、实现中依赖的外部服务

#### 2.2.1 注册一条新的友链

* 功能说明：

  NULS主网会提供一个入口（网页），可以通过这个入口注册新的友链到NULS主网。

* 流程描述

  ![](image/chainModule/chainRegister.png)

* 依赖服务

  [^说明]: 文字描述依赖了哪些服务，做什么事情

  - 网络管理模块，广播交易
  - 事件总线模块，发布事件






#### 2.2.2 注销已经存在的友链

- 功能说明：

  NULS主网会提供一个入口（网页），可以通过这个入口注销已经存在的友链。

- 流程描述

  ![](E:/tony/nuls/nuls_2.0_docs/design-zh-CHS/image/chainModule/chainDestroy.png)

- 依赖服务

  [^说明]: 文字描述依赖了哪些服务，做什么事情

  - 网络管理模块，广播交易
  - 事件总线模块，发布事件





#### 2.2.3 查询友链信息

- 功能说明：

  根据链标识（chainId）查询链的具体信息

- 流程描述
  N/A

- 依赖服务

  [^说明]: 文字描述依赖了哪些服务，做什么事情

  N/A



#### 2.2.4 特定友链增加资产类型

- 功能说明：

  NULS主网会提供一个入口（网页），可以通过这个入口对特定友链增加资产类型。

- 流程描述

  ![](E:/tony/nuls/nuls_2.0_docs/design-zh-CHS/image/chainModule/assetRegister.png)

- 依赖服务

  [^说明]: 文字描述依赖了哪些服务，做什么事情

  - 网络管理模块，广播交易
  - 事件总线模块，发布事件



#### 2.2.5 特定友链删除资产类型

- 功能说明：

  NULS主网会提供一个入口（网页），可以通过这个入口对特定友链销毁资产类型。

- 流程描述

  ![](E:/tony/nuls/nuls_2.0_docs/design-zh-CHS/image/chainModule/assetDestroy.png)

- 依赖服务

  [^说明]: 文字描述依赖了哪些服务，做什么事情

  - 网络管理模块，广播交易
  - 事件总线模块，发布事件





## 三、接口设计

### 3.1 模块接口

#### 获取链信息

- 接口说明
  获取一条链详细信息

- 请求示例
  ```json
  {
      "cmd":"chainInfo",
      "minVersion":"1.1",
      "params":[1234]
  }
  ```

- 请求参数说明

  | index | parameter | required | type | description |
  | ----- | :-------- | :------- | :--- | ----------- |
  | 0     | chainId   | true     | int  | 链标识      |

- 返回示例  
  Failed

  ```json
  {
      "version": 1.2,
      "code":1,
      "msg" :"xxxxxxxxxxxxxxxxxx",
      "result":{}
  }
  ```

  Success
    ```json
    {
      "code":10000,
      "msg":"",
      "version":"",
      "result":{        "hash":"0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331",
          "chainId":1234,
          "name":"name",
          "addressType":1,
          "assets":[
              {
                  "assetId":1,
                  "symbol":"xxx",
                  "name":"xxx",
                  "depositNuls":20000,
                  "initTotal":1000000,
                  "minUnit":8,
                  "flag":true
              }
          ],
          "magicNumber":1025753999,
          "seeds":[
              {
                  "ip":"xxx.xxx.xxx.xxx",
                  "port":8001
              },
              {
                  "ip":"xxx.xxx.xxx.xxx",
                  "port":8001
              }
          ],
          "supportInflowAsset":true
      }
  }
    ```

- 返回字段说明  

  | parameter          | type                  | description                                  |
  | ------------------ | --------------------- | -------------------------------------------- |
  | hash               | int                   | 链的哈希值                                   |
  | chainId            | int                   | 链标识                                       |
  | name               | string                | 链名称                                       |
  | addressType        | int                   | 链上创建的账户的地址类型                     |
  | assets             | jsonArray【资产对象】 | 数组成员：资产对象                           |
  | magicNumber        | int                   | 魔法参数                                     |
  | seeds              | jsonArray             | 【ip->种子节点ip地址】【port->种子节点端口】 |
  | supportInflowAsset | boolean               | 是否支持资产流入                             |

    

  资产对象   

  | parameter   | type    | description                    |
  | ----------- | ------- | ------------------------------ |
  | assetId     | int     | 资产标识                       |
  | symbol      | string  | 资产单位                       |
  | name        | string  | 资产名称                       |
  | depositNuls | int     | 抵押的nuls总量                 |
  | initTotal   | long    | 资产发行总量                   |
  | minUnit     | byte    | 最小单位（代表小数点后多少位） |
  | flag        | boolean | 资产是否可用                   |




### 3.2 功能接口

#### 3.2.1 注册一条新的友链

- 接口说明
  注册一条新链

- 请求示例

  ```json
  {
      "cmd":"chainRegister",
      "minVersion":1,
      "params":[
          1234,
          "name",
          1,
          [
              {
                  "assetId":1,
                  "symbol":"xxx",
                  "name":"xxx",
                  "depositNuls":20000,
                  "initTotal":1000000,
                  "minUnit":8,
                  "flag":true
              }
          ],        
          1,
          1,
          "xxx",
          1
      ]
  }
  ```

- 请求参数说明

  | index | parameter               | required | type                  | description              |
  | ----- | :---------------------- | :------- | :-------------------- | ------------------------ |
  | 0     | chainId                 | true     | int                   | 链标识                   |
  | 1     | chainName               | true     | string                | 链名称                   |
  | 2     | addressType             | true     | string                | 链上创建的账户的地址类型 |
  | 3     | assets                  | true     | jsonArray【资产对象】 | 数组成员：资产对象       |
  | 4     | minAvailableNodeNum     | true     | int                   | 最小可用节点数量         |
  | 5     | singleNodeConMinNodeNum | true     | int                   | 单节点连接最小数量       |
  | 6     | txConfirmBlockNum       | true     | int                   | 交易确认块数             |
  | 7     | supportInflowAsset      | true     | boolean               | 是否支持资产流入         |

- 返回示例  
  Failed

  ```json
  {
      "version": 1.2,
      "code":1,
      "msg" :"xxxxxxxxxxxxxxxxxx",
      "result":{}
  }
  ```

  Success

  ```json
  {
   	"version": 1.2,
      "code":0,
      "result":{}
  }
  ```

- 返回字段说明  
  无





#### 3.2.2 注销已经存在的友链

- 接口说明
  创建者可以注销自己创建的链

- 请求示例

  ```json
  {   
      "cmd": "chainDestroy",
      "minVersion": 1.0,
      "params":[1234]
  }
  ```

- 请求参数说明

  | index | parameter | required | type | description |
  | ----- | :-------- | :------- | :--- | ----------- |
  | 0     | chainId   | true     | int  | 链标识      |

- 返回示例  
  Failed

  ```json
  {
      "version": 1.2,
      "code":1,
      "msg" :"xxxxxxxxxxxxxxxxxx",
      "result":{}
  }
  ```

  Success

  ```json
  {
   	"version": 1.2,
      "code":0,
      "result":{}
  }
  ```

- 返回字段说明  
  无






#### 3.2.3 查询友链信息

- 接口说明
  查询链列表

- 请求示例

  ```json
  {   
      "cmd": "chainList",
      "minVersion": 1.0,
      "params":[ 
     		 1,20
      ]
  }
  ```

- 请求参数说明

  | index | parameter  | required | type | description |
  | ----- | :--------- | :------- | :--- | ----------- |
  | 0     | pageNumber | true     | int  | 页数        |
  | 1     | pageSize   | true     | int  | 每页数量    |

- 返回示例  
  Failed

  ```json
  {
      "version": 1.2,
      "code":1,
      "msg" :"xxxxxxxxxxxxxxxxxx",
      "result":{}
  }
  ```

  Success

  ```json
  {
      "version":1.2,
      "code":0,
      "result":{
          "chainList":[
              {                "hash":"0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331",
                  "chainId":1234,
                  "name":"name",
                  "addressType":1,
                  "assets":[
                      {
                          "assetId":1,
                          "symbol":"xxx",
                          "name":"xxx",
                          "depositNuls":20000,
                          "initTotal":1000000,
                          "minUnit":8,
                          "flag":true
                      }
                  ],
                  "magicNumber":1025753999,
                  "seeds":[
                      {
                          "ip":"xxx.xxx.xxx.xxx",
                          "port":8001
                      },
                      {
                          "ip":"xxx.xxx.xxx.xxx",
                          "port":8001
                      }
                  ],
                  "supportInflowAsset":true
              },
              {
                  "hash":"0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331",
                  "chainId":1234,
                  "name":"name",
                  "addressType":1,
                  "assets":[
                      {
                          "assetId":1,
                          "symbol":"xxx",
                          "name":"xxx",
                          "depositNuls":20000,
                          "initTotal":1000000,
                          "minUnit":8,
                          "flag":true
                      }
                  ],
                  "magicNumber":1025753999,
                  "seeds":[
                      {
                          "ip":"xxx.xxx.xxx.xxx",
                          "port":8001
                      },
                      {
                          "ip":"xxx.xxx.xxx.xxx",
                          "port":8001
                      }
                  ],
                  "supportInflowAsset":true
              }
          ]
      }
  }
  ```

- 返回字段说明  
  同【获取链信息】，不同的是这里返回的是链列表，【获取链信息】返回指定链







#### 3.2.4 特定友链增加资产类型

- 接口说明
  创建者可以在自己创建的链种新增资产类型

- 请求示例

  ```json
  {   
      "cmd":"assetRegister",
      "minVersion":1.0,
      "params":[
          1234,
          1,
          [
              {
                  "assetId":1,
                  "symbol":"xxx",
                  "name":"xxx",
                  "depositNuls":20000,
                  "initTotal":1000000,
                  "minUnit":8,
                  "flag":true
              }
          ]
      ]
  }
  ```

- 请求参数说明

  | index | parameter   | required | type               | description      |
  | ----- | :---------- | :------- | :----------------- | ---------------- |
  | 0     | chainId     | true     | int                | 链标识           |
  | 1     | addressType | true     | int                | 资产中的地址类型 |
  | 2     | asset       | true     | object【资产对象】 | 新增的资产       |

- 返回示例  
  Failed

  ```json
  {
      "version": 1.2,
      "code":1,
      "msg" :"xxxxxxxxxxxxxxxxxx",
      "result":{}
  }
  ```

  Success

  ```json
  {
   	"version": 1.2,
      "code":0,
      "result":{}
  }
  ```

- 返回字段说明  
  无





#### 3.2.5 特定友链删除资产类型

- 接口说明
  创建者可以注销自己创建的链中的资产

- 请求示例

  ```json
  {
      "cmd":"assetDestroy",
      "minVersion":1,
      "params":[
          1234,
          88
      ]
  }
  ```

- 请求参数说明

  | index | parameter | required | type | description |
  | ----- | :-------- | :------- | :--- | ----------- |
  | 0     | chainId   | true     | int  | 链标识      |
  | 1     | assetId   | true     | int  | 资产标识    |

- 返回示例  
  Failed

  ```json
  {
      "version": 1.2,
      "code":1,
      "msg" :"xxxxxxxxxxxxxxxxxx",
      "result":{}
  }
  ```

  Success

  ```json
  {
   	"version": 1.2,
      "code":0,
      "result":{}
  }
  ```

- 返回字段说明  
  无



## 四、事件说明

[^说明]: 业务流程中尽量避免使用事件的方式通信

### 4.1 发布的事件

[^说明]: 这里说明事件的topic，事件的格式协议（精确到字节），事件的发生情景。

[参考<事件总线>]: ./



#### 4.1.1 注册一条新的友链  

 event_topic : "chain_register",

```
{
    "hash":"0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331",
        "chainId":1234,
        "name":"name",
        "addressType":1,
        "assets":[
            {
                "assetId":1,
                "symbol":"xxx",
                "name":"xxx",
                "depositNuls":20000,
                "initTotal":1000000,
                "minUnit":8,
                "flag":true
            }
        ],
        "magicNumber":1025753999,
        "seeds":[
            {
                "ip":"xxx.xxx.xxx.xxx",
                "port":8001
            },
            {
                "ip":"xxx.xxx.xxx.xxx",
                "port":8001
            }
        ],
        "supportInflowAsset":true
}
```





#### 4.1.2 注销已经存在的友链   

 event_topic : "chain_destroy",

```
{
    "hash":"0xe670ec64341771606e55d6b4ca35a1a6b75ee3d5145a99d05921026d1527331",
        "chainId":1234,
        "name":"name",
        "addressType":1,
        "assets":[
            {
                "assetId":1,
                "symbol":"xxx",
                "name":"xxx",
                "depositNuls":20000,
                "initTotal":1000000,
                "minUnit":8,
                "flag":true
            }
        ],
        "magicNumber":1025753999,
        "seeds":[
            {
                "ip":"xxx.xxx.xxx.xxx",
                "port":8001
            },
            {
                "ip":"xxx.xxx.xxx.xxx",
                "port":8001
            }
        ],
        "supportInflowAsset":true
}
```





#### 4.1.3 对特定友链增加资产类型   

 event_topic : "asset_register",

```
{
	"chainId":1234,
    "assetId":1,
    "symbol":"xxx",
    "name":"xxx",
    "depositNuls":20000,
    "initTotal":1000000,
    "minUnit":8,
    "flag":true
}
```





#### 4.1.4 特定友链删除资产类型   

 event_topic : "asset_destroy",

```
{
	"chainId":1234,
    "assetId":1,
    "symbol":"xxx",
    "name":"xxx",
    "depositNuls":20000,
    "initTotal":1000000,
    "minUnit":8,
    "flag":true
}
```






## 五、协议

### 5.1 网络通讯协议

[^说明]: 节点间通讯的具体协议，参考《网络模块》

#### 无




### 5.2 交易协议

##### 5.2.1 注册一条新的友链

与通用交易相比，只有类型和txData有区别，具体区别如下

```
  type: n // 交易的类型   Uint16
  txData:{
      chainId:  //uint16 ,链id   uint16
      链名称     //varString     不超过30个字节
      
      资产信息列表：资产id 标识、名称、初始总额、最小单位 资产是否可用	
          [{
              资产id： uint32	  
              标识：//varString，用于查询资产总额，区分不同资产     不超过30个字节
              名称：//varString，用于显示      不超过30个字节
              初始总额：//varint         4个字节
              最小单位://byte,代表小数点后多少位，     1个字节     
              资产是否可用： 判断资产是否可以流通     1个字节	
          }]
          
      地址类型：1，//byte : 1、NULS生态圈地址，2、其他    1个字节
      最小可用节点数量 //uint16,    2个字节
      单节点连接最小数量 //uint16,   2个字节
      交易确认块数 //uint16,     2个字节
      是否支持资产流入 //boolean,   1个字节
  }

```

说明：

type序列化2个字节

 txData:{

​	chhainId  2个字节

​	chainName   =》 前面 1个字节记录长度，比如length=9 表示读后面的 9个字节组成 chainName

​	Assets 资产列表需要序列化资产 对象 Asset   循环调用parse（byte[] b） 方法直到完成 

​			[

​				assetId   uint32

​				symbol     前面 1个字节记录长度，比如length=9 表示读后面的 9个字节组成 symbol

​				name         前面 1个字节记录长度，比如length=9 表示读后面的 9个字节组成 name         

​				initTotal      占用 32个字节

​				minUnit       占用1个字节

​				flag              占用1个字节

​			] 

​	assetType   占用1个字节

​	minAvailableNodeNum   占用2个字节

​	singleNodeConMinNodeNum   占用2个字节

​	txConfirmBlockNum    占用8个字节

​	supportInflowAsset 占用1个字节

}





| 参数                    | 必选  | 类型        | 说明                                                      |
| :---------------------- | :---- | :---------- | --------------------------------------------------------- |
| type                    | true  | uint16      | tx类型                                                    |
| chainId                 | ture  | uint16      | 链唯一id                                                  |
| chainName               | true  | varString   | 链名称           不超过30个字节                           |
| assets                  | true  | List<Asset> | 资产列表                                                  |
| assetId                 | true  | uint32      | 资产id             4个字节                                |
| symbol                  | true  | varString   | 资产标识         不超过30个字节                           |
| name                    | true  | varString   | 资产名字        不超过30个字节                            |
| initTotal               | true  | long        | 发行总量          32个字节                                |
| minUnit                 | true  | byte        | 最小单位://byte,代表小数点后多少位，比如1nuls=100000000na |
| flag                    | false | byte        | 资产是否可用  //0.不可用  1.可用                          |
| addressType             | true  | byte        | 资产类型  1、NULS地址结构，2、其他                        |
| minAvailableNodeNum     | true  | uint16      | 最小可用节点数量                                          |
| singleNodeConMinNodeNum | true  | uint16      | 单节点连接最小数量                                        |
| txConfirmBlockNum       | true  | Long        | 交易确认块数                                              |
| supportInflowAsset      | true  | byte        | 是否支持资产流入   1支持，0不支持                         |
| depositNuls             | false | Long        | 抵押金 （系统做）  比如nuls 抵押20000                     |
| fee                     | false | Na          | 手续费（系统做）                                          |
| signature               | false | byte[]      | 签名（系统做）                                            |

##### - 验证器

```
  chainId合法性 是否重复  chainId是由调用者生成表示有“意义”的一串数字

  各字段不为空、值在正确范围内

  抵押金验证器

  其他基本验证器
```

##### - 处理器

```
存储链信息

存储资产信息

在n（运行参数）块确认之后开始监听该链的魔法参数
```



##### 5.2.2 注销已经存在的友链  

协议   

与通用交易相比，只有类型和txData有区别，具体区别如下

```
  type: n // 交易的类型 序列化2个字节
  txData:{
      chainId:  //uint16 ,链id  序列化2个字节
  }
```



##### - 验证器

```
  chainId合法性
  该链是否可以被该地址操作，权限验证
```



##### - 处理器

```
在n（运行参数）块之后停止该链所有跨链交易、解锁抵押金、从区块链中逻辑删除该链数据
```



##### 5.2.3 对特定友链增加资产类型

与通用交易相比，只有类型和txData有区别，具体区别如下 

```
 type: n // 交易的类型    序列化2个字节

  txData:{

    chainId:  //uint16 ,链id    序列化2个字节

    标识：//varString，用于查询资产总额，区分不同资产    最多30个字节

    名称：//varString，用于显示   最大30个字节  

    总额：//     占用 32个字节

    最小单位://byte,代表小数点后多少位

  }
```

说明：

​	 type: n // 交易的类型    序列化2个字节

​	 txData:{

​	  	chainId:  //uint16 ,链id    序列化2个字节

​		symbol：varString 	 前面 1个字节记录长度，比如length=9 表示读后面的 9个字节组成 symbol

​		name         前面 1个字节记录长度，比如length=9 表示读后面的 9个字节组成 name         

​		initTotal      占用 32个字节

​		minUnit       占用1个字节

​	}



##### - 验证器

```
  chainId合法性

  各字段不为空、值在正确范围内

  抵押金验证器

  其他基本验证器

```



##### - 处理器

```
存储资产信息

```



##### 5.2.4 特定友链删除资产类型

与通用交易相比，只有类型和txData有区别，具体区别如下

```
  type: n //交易的类型
  txData:{
      chainId:  //uint16 ,链id
      assetsId：   // ,资产Id
  }
```

说明

type: n // 交易的类型    序列化2个字节

txData:{

​	  	chainId:  //uint16 ,链id    序列化2个字节	

​		assetsId：//	uint32       序列化4个字节	

}

##### - 验证器

```
  资产合法性  
  资产是否可以被该地址操作，权限验证
  是否是该链最后一种资产

```



##### - 处理器

```
若该资产没有转移到发行链之外，则立刻停止该资产交易、解锁抵押金、从区块链中逻辑删除该资产数据
否则，在n（运行参数）块之后停止该资产交易、解锁抵押金、从区块链中逻辑删除该资产数据

```






## 六、模块配置

[^说明]: 本模块必须要有的配置项

| Parameter               | Type     | Description          |
| ----------------------- | -------- | -------------------- |
| deposit                 | long     | 注册友链所需的抵押金 |
| minTxConfirmBlocks      | int      | 最小交易确认块数     |
| maxTxConfirmBlock       | int      | 最大交易确认块数     |
| maxSymbolLength         | int      | 最大货币符号长度     |
| dependsModule           | string[] | 依赖的模块           |
| minAvailableNodeNum     | int      | 最小可用节点数       |
| singleNodeConMinNodeNum | int      | 单节点连接最小数量   |



## 七、Java特有的设计

[^说明]: 核心对象类定义,存储数据结构，......

## 八、补充内容

[^说明]: 上面未涉及的必须的内容

