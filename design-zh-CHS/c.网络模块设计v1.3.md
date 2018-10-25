# 网络模块设计文档

[TOC]

## 一、总体描述

### 1.1 模块概述

#### 1.1.1 为什么要有《网络模块》

网络模块保障了去中心化节点间的通讯，为NULS基础模块之一，提供最底层的网络通信、节点发现等服务。区块链的网络基础就是Peer to  Peer,即P2P。P2P网络中的所有参与者，可以是提供服务（server），也可以是资源使用者（client）。P2P网络的特点：去中心化、可扩展性、健壮性、高性价比、隐私保护、负载均衡。

#### 1.1.2 《网络模块》要做什么

网络模块是整个系统的基础模块，用来管理节点、节点间的连接及连接的状态、数据的发送与接收。网络模块不涉及复杂的业务逻辑。

* 接收到的网络消息，根据内核模块中的指令服务映射关系，推送消息相应的处理模块中。

* 开放接口供其他模块封装好的消息调用推送到指定的peer节点以及广播到指定的网络组中。


#### 1.1.3 《网络模块》在系统中的定位

* 网络模块是底层应用模块，任何需要网络通讯的模块都要通过网络模块来进行消息的收发。
* 网络模块依赖核心模块进行服务接口的治理。
* 网络模块按网络id（魔法参数） 来进行不同网络的构建。
* 网络模块在卫星链中的节点在进行跨链网络组建时，需要链管理模块提供跨链配置信息。
* 网络模块在子链中的节点在进行跨链网络组建时，需要跨链模块提供跨链配置信息。

### 1.2 架构图



![](./image/network-module/network-context.png)



## 二、功能设计

### 2.1 功能架构图

   网络模块在业务上功能有：节点管理，节点组管理，p2p网络连接管理，消息的收发管理。

   在内部基础功能含有:模块的状态管理（包含启动与关闭管理），对外的接口提供管理，

  线程任务管理，数据存储管理等。

![](.\image\network-module\network-functions.png)

- 节点管理 Node management 

  管理所有可连接的、已连接的节点信息、状态，提供节点操作接口

  - 节点发现
  - 节点存储

- 节点组管理 Node Group management

  管理不同的网络节点，将节点分为不同的集合，每个集合分别管理。每个集合内的节点连接的魔法参数都是相同的，并且和其他集合的魔法参数不同。

  每个NodeGroup都根据链登记的信息或者自身配置的网络信息进行初始化（魔法参数、节点数量限制等）

  每初始化一个NodeGroup，网络服务都多监听一个MagicNumber。

- 连接器管理 Connection management

  - 初始化连接
    - 卫星链节点：随机连接
    - 跨链节点：已固定的算法连接，目标是将跨链节点分散并全部连接
  - 连接管理：心跳维护
  - 连接断开

- 消息收发管理

  1>消息接收 Message receiver

  接收网络节点发送过来的消息，对消息进行简单的判断（判断cmd），根据消息cmd字段，将消息发送到关心的模块服务中。

[^注]: 服务接口信息（url）从kernel模块定时获取并缓存。



​      2>  消息发送 Message sender

​                  a> 对NodeGroup广播消息

​                  b> 对一个节点发送消息

- 模块状态管理

  ​       a>启动，关闭逻辑处理

​              b>对自身模块状态的维护与管理：管理模块的运行状态，内部功能状态等

​        

- 接口管理

  a>注册自身接口到kernel模块中

  b>同步模块信息与状态到kernel模块中 

  c>获取服务列表到本地模块

  d>开放对外接口调用

- 线程任务管理

  a>管理 心跳线程

  b>管理 节点发现/淘汰机制线程

  c>管理  接口信息同步线程



### 2.2 模块服务

#### 2.2.1 网络消息接收

* 功能说明：

    接收（外部）网络节点发送过来的消息，对消息进行简单的判断（判断魔法参数），根据消息头含有的command字段，将消息发送到关心的模块服务中。

* 流程描述

  ![](.\image\network-module\recMessage2.png)

* 设置运行参数接口

   - method : ***  //同消息头中的CMD指令，约束12字节

     接口说明：转发消息到外部模块

   - params

```
    0：chainId //链id
    1：nodeId //节点Id
    2：message //16进制网络序消息体
    ......
```

- returns 

```
    {
        "code": -1,
        "msg": "What happend",
        "version":"1.0",
        "result": {
        }
    }
```

```

```

  * 依赖服务

    解析消息头中的command参数，在调用远程接口处理时，依赖内核模块提供的远程的服务接口数据。

  #### 2.2.2网络消息发送

转发 其他或自身模块封装的消息，含广播消息以及指定节点发送消息。

##### 2.2.2.1、广播网络消息

功能说明：

   转发 其他或自身模块封装的消息，对外部模块提供转发调用的接口有以下2种情况：

  a> 对NodeGroup（指定某个网络）广播消息。

  b> 对NodeGroup（指定某个网络）广播消息，并排除某些节点。

- 流程描述



  ![](.\image\network-module\sendMsg1.png)

- 接口定义

  - 设置运行参数接口

    - method : nw_broadcast

      接口说明：外部模块可以通过该接口去广播消息

    - params

    ```
    0：chainId //链id
    1：excludeNodes //排除的节点Id列表，用逗号分隔  "nodeId1,nodeId2,nodeId3"
    2：message //16进制网络序消息体
    ......
    ```

    - returns 

    ```
    {
        "code": -1,
        "msg": "What happend",
        "jsonrpc":"1.0",
        "result": {
        }
    }
    ```


- 依赖服务

  无

##### 2.2.2.2、指定节点发送网络消息

功能说明：

转发 其他或自身模块封装的消息，可以指定某些节点（可以为1个节点）发送消息。

- 流程描述

  ![](.\image\network-module\sendMsg2.png)

- 接口定义

  - 设置运行参数接口

    - method : nw_sendPeersMsg

      接口说明：外部模块可以通过该接口去广播消息

    - params

    ```
    0：chainId //链id
    1：nodes //要发送的节点Id列表，用逗号分隔  "nodeId1,nodeId2,nodeId3"
    2：message //16进制网络序消息体,"FFFFFAAAAAA"
    ......
    ```

    - returns 

    ```
    {
        "code": -1,
        "msg": "What happend",
        "jsonrpc":"1.0",
        "result": {
        }
    }
    ```


- 依赖服务

  无

#### 2.2.3 创建节点组

- 功能说明：

  卫星链除了自身卫星网络，还存在n个跨链网络，友链上除了自身网络，还存在1个跨链网络。

  节点组就是用来管理不同网络信息的。网络模块通过节点组去隔离维护不同网络。

  节点组类型：1>自身网络 2>友链跨链网络 3>卫星链跨链网络

  网络模块是接收外部模块的调用，来创建节点组。跨链的基本网络配置信息主要通过2种途径获得:

  1>作为卫星链节点，由链管理模块进行注册登记后，系统产生交易验证确认后调用。

  2>作为友链节点，由跨链协议模块启动时，跨链协议模块从模块配置中获取跨链配置信息进行调用。

  PS：在实际使用中节点只会从一种途径获得信息，因为节点的角色只能是卫星链节点或友链节点。

* 流程描述

  ![](.\image\network-module\createNodeGroup.png)

* 接口定义

  - 设置运行参数接口

    - method : nw_createNodeGroup

      接口说明：接收外部模块的调用，创建节点组

    - params

    ```
    0：chainId //链id
    1：magicNumber //网络魔法参数
    2：maxOut //主动连接数量
    3：maxIn //被动连接数量
    4：minAvailableCount //最小连接数量
    5：seedIps  //种子节点
    6：isMoonNode //默认0，非卫星链节点，1是卫星链节点
    7：serverPort //节点作为server的端口号
    ```

    - returns 

    ```
    {
        "version":"1.0",
        "code": 0,
        "msg": "success",
        "result": {
        }
    }
    ```

 * 依赖服务

  依赖内核模块提供的远程的服务接口数据。



#### 2.2.4 注销节点组

- 功能说明：

  接收外部模块的调用，注销跨链节点组。

  1>作为卫星链节点，由链管理模块进行注销登记，系统产生交易验证确认后调用。

  2>作为友链节点，在跨链协议启动后n分钟未接受到注册跨链的请求，注销超时的节点组。

- 流程描述

   ![](.\image\network-module\deleteNodeGroup2.png)

- 接口定义

  - 设置运行参数接口

    - method : nw_delNodeGroup

      接口说明：接收外部模块的调用，删除节点组

    - params

    ```
    0：chainId //链id
    ```

    - returns 

    ```
    {
        "version":"1.0",
        "code": -1,
        "msg": "What happend",
        "result": {
        }
    }
    ```

- 依赖服务

  依赖内核模块提供的远程的服务接口数据。



#### 2.2.5 跨链种子节点提供

- 功能说明：

  种子节点是在网络初始化时候，用于提供peer连接信息的节点。链管理模块在进行链注册时，需要获取卫星链上种子节点信息用于初始化连接。

- 流程描述

   无

- 接口定义

  - 设置运行参数接口

    - method : nw_getSeeds

      接口说明：获取卫星链种子节点

    - params

    ```
    0：chainId //链id
    ```

    - returns 

    ```
    {
        "version":"1.0",
        "code": 0,
        "msg": "success",
        "result": {
         seedsIps:"101.132.33.140:8003,116.62.135.185:8003,47.90.243.131:8003"
        }
    }
    ```

- 依赖服务

  依赖内核模块提供的远程的服务接口数据。



#### 2.2.6 添加连接节点

- 功能说明：

  cmd指令下，对某个网络添加peer连接信息。

- 流程描述

  添加的节点会触发网络连接 流程。

- 接口定义

  - 设置运行参数接口

    - method : nw_addNode

      接口说明：添加网络peer节点

    - params

    ```
    0：chainId //链id
    1：magicNumber //魔法参数
    2：ip //peer Ip地址
    3：port //端口号
    ```

    - returns 

    ```
    {
        "version":"1.0",
        "code": 0,
        "msg": "success",
        "result": {
           "nodeId":"" //返回节点Id
        }
    }
    ```

- 依赖服务

   无



#### 2.2.7 删除连接节点

- 功能说明：

  cmd指令下，对某个网络删除peer连接信息。

- 流程描述

  删除节点会触发 网络节点的断开。

- 接口定义

  - 设置运行参数接口

    - method : nw_delNode

      接口说明：删除网络peer节点

    - params

    ```
    0：chainId //链id
    1：nodeId //节点id
    ```

    - returns 

    ```
    {
        "version":"1.0",
        "code": 0,
        "msg": "success",
        "result": {
        }
    }
    ```

- 依赖服务

   无

####  2.2.8重连指定网络

- 功能说明：

  cmd指令下，对指定的nodeGroup进行网络重新连接

- 流程描述

  接收指令后，对指定的nodeGroup下的所有peer进行断连接后，重新连接网络。

  （是否要删除nodeGroup系peer节点，并重新去发现peer？）

- 接口定义

  - 设置运行参数接口

    - method : nw_reconnect

      接口说明：获取卫星链种子节点

    - params

    ```
    0：chainId //链id
    ```

    - returns 

    ```
    {
        "version":"1.0",
        "code": 0,
        "msg": "success",
        "result": {
        }
    }
    ```

- 依赖服务

   无

#### 2.2.9 获取nodeGroup列表

- 功能说明：

  获取节点管理的所有网络列表。

- 流程描述

  无

- 接口定义

  - 设置运行参数接口

    - method : nw_getGroups

      接口说明：获取节点组信息

    - params

    ```
    0：startPage //起始页，填写为0时获取全部
    1: recordNum //每页记录数量
    ```

    - returns 

    ```
    {
        "version":"1.0",
        "code": 0,
        "msg": "success",
        "result": {
             list:[{
                chainId：1212, //链id
                magicNumber：324234,//魔法参数
                totalCount：2323, //总连接数
                inCount：22,   //被动连接数
                outCount：33,  //主动连接数
                isActive：1，//0未激活，1 已激活
                isCrossChain:1 //0不是跨链网络，1跨链网络
                },{}]
        }
    }
    ```

- 依赖服务

   无

#### 2.2.10 获取指定nodeGroup下的连接信息

- 功能说明：

  获取指定的网络id下所有节点的信息

- 流程描述

  无

- 接口定义

  - 设置运行参数接口

    - method : nw_getNodes

      接口说明：获取节点信息

    - params

    ```
    0：chainId //链id
    1：startPage //起始页，填写为0时获取全部
    2: recordNum //每页记录数量
    3: state //0所有的节点，1待连接的节点，2连接中的节点（待握手），3已连接的节点
    ```

    - returns 

    ```
    {
        "version":"1.0",
        "code": 0,
        "msg": "success",
        "result": {
              list:[{
                    chainId：122,//链id
                    magicNumber：134124,//魔法参数
                    version：2,//协议版本号
                    ip："200.25.36.41",//ip地址
                    port：54,//
                    state："已连接",
                    isOut："1", //0被动连接，1主动连接
                    time："6449878789", //最近连接时间
    	     },{}]
        }
    }
    ```

- 依赖服务

   无

### 2.3 模块内部功能

#### 2.3.1 模块启动

- 功能说明：

  模块启动时，进行配置信息的初始化，注册服务初始化，各个内部功能管理数据库信息的初始化等。

- 流程描述

​        ![](.\image\network-module\start.png)



- 依赖服务

  无

#### 2.3.2 模块关闭

- 功能说明：

   模块关闭时，进行连接的关闭，线程管理的关闭，各个资源的释放，并通知状态给核心接口。

- 流程描述

![](.\image\network-module\shutdown.png)



- 依赖服务

  无

#### 2.3.3 peer节点发现

- 功能说明：

  在网络模块启动后，进行peer节点的获取管理。

  节点获取的途径有：

  1>连接向种子节点，并请求地址获取。

  2>接收广播过来的节点消息。

  2>跨链网络的连接，比如作为卫星链上的节点与子链的连接，或者子链上的节点与卫星链的连接。

- 流程描述

![](.\image\network-module\discoverPeer.png)



- 依赖服务

​          无

#### 2.3.4 网络连接

- 功能说明：

   一个节点即作为client端，主动连接已知的peer节点，同时也是server端，等待peer节点的连接。

  一个连接能够正常工作，需要通过握手协议，即互相发送version协议消息，协议具体定义参看下面的

  “协议-网络通讯协议部分”。

- 流程描述

​        client在与server完成tcp连接后，需要通过业务version协议握手，只有握手成功的连接才能进行业务转发工作。连接中状态在持续X分钟后无法跃迁到已连接，则主动断开连接。

![](.\image\network-module\connection.png)



- 依赖服务


#### 2.3.5 心跳检测

- 功能说明：

  检测连接是否还保持连接。通过client与server的ping-pong消息来进行维持保活。涉及的ping-pong协议定义参看“协议-网络通讯协议部分”

- 流程描述

![](.\image\network-module\pingpong.png)



- 依赖服务



#### 2.3.6 连接数量验证

- 功能说明：

  在建立节点连接时，会进行连接数量，如果达到最大值，则主动断开连接。

  流程描述

​      ![](.\image\network-module\connet-validate.png)



- 依赖服务



#### 2.3.7 节点的外网IP存储

- 功能说明：

​        一个节点可能存在多个网卡，也可能是在某个局域网内，所以在建立连接时，并不知道自己的外网IP地址。

而节点要广播自己的地址供外网peers节点连接，则需要知道自身的外网IP地址。我们的设计中，节点的外网是通过version协议消息进行携带发出的。

- 流程描述

    在client接收到version消息时，可以知道自己的IP地址信息。

![](.\image\network-module\saveNodeIp.png)



- 依赖服务



#### 2.3.8 节点的外网可连接检测

- 功能说明：

​        一个节点在建立连接时，可以广播自身的外网IP+port，给其他节点。但若一个节点是在某局域网内，则其外网的IP地址是是无法直接去连接通的。因此为了检测节点的外网IP是否可用，可以通过自己的client连接自己的server来，如果连接成功，则IP可以用于广播，如果不成功则节点外网IP不能广播给其他节点。

- 流程描述

  自我连接可能成功，也可能失败，如果成功则说明外网IP是可达的，便可以在建立连接时广播传递给网络中其他节点，如果不可达，则连接无法建立不用处理。

![](.\image\network-module\connectSelf.png)



- 依赖服务


#### 2.3.9 peer节点的广播

- 功能说明：

  将自身节点广播给网络中其他节点，在设计中，我们将通过上面建立的自我连接，成功时，便可进行广播。

- 流程描述

![](.\image\network-module\connectSelf-recieve.png)



- 依赖服务



#### 2.3.10 跨链server端口的传递

- 功能说明：

   一个node可以存在2个角色，一个是维护自有的内部链网络。另一个是作为跨链角色维护跨链网络。

  因此在server定义中我们将这2个网络的端口监听进行区分，以便各自相对独立。自有链定义一个serverPort1，跨链部分定义一个serverPort2.

- 流程描述

  如下图，我们卫星链网络与友链网络产生一个跨链连接，当卫星链中有个节点2连接上节点1时，是通过内部服务Port1来建立的连接，而节点1是可以将节点2 发送给 友链的节点A与节点B来进行连接，则此时发送给友链的信息中 应该是serverPort2，因此serverPort2需要再卫星链的内部交互中进行传递。我们将该部分数据定义在version协议中进行传递。

​      ![](.\image\network-module\crossPortDeliver.png)

* 依赖服务



## 四、事件说明

### 4.1 发布的事件

[^说明]: 这里说明事件的topic，事件的格式协议（精确到字节），事件的发生情景。

[参考<事件总线>]: ./



#### 4.1.1 NodeGroup达到节点数量下限

说明：NodeGroup达到节点数量下限，发布该事件   

 event_topic : "evt_nw_inNodeLimit",

```
data:{
    chainId
    magicNumber
    nodeCount
    nodeLimit
    time
}
```

#### 4.1.2 NodeGroup少于节点数量下限

说明：NodeGroup少于节点数量下限，发布该事件   

 event_topic : "evt_nw_lessNodeLimit",

```
data:{
    chainId
    magicNumber
    nodeCount
    nodeLimit
    time
}
```

#### 4.1.3 节点握手成功

说明：节点握手成功，发布该事件   

 event_topic : "evt_nw_connectSuccess",

```
data:{
    chainId
    magicNumber
    nodeId
    time
    version
}
```

#### 4.1.4 节点断开连接

说明：节点断开连接，发布该事件   

 event_topic : "evt_nw_nodeDisconnect",

```
data:{
    chainId
    magicNumber
    nodeId
    time
    version
}
```





### 4.2 订阅的事件

​       暂无

[^说明]: 这里说明订阅哪些事件，订阅的原因，事件发生后的处理逻辑

*  


## 五、协议

### 5.1 网络通讯协议

[^说明]: 节点间通讯的具体协议，参考《网络模块》

#### version

用于建立连接(握手)

| Length | Fields      | Type     | Remark                                                       |
| ------ | ----------- | -------- | ------------------------------------------------------------ |
| 4      | version     | uint32   | 节点使用的协议版本标识                                       |
| 20     | addr_you    | byte[20] | 对方网络地址【IP+PORT1+PORT2】PORT2为跨链server端口   如：[10.32.12.25  8003  9003]  16byte+2byte+2byte |
| 20     | addr_me     | byte[20] | 本节点网络地址【IP+PORT1+PORT2】PORT2为跨链server端口   如：[20.32.12.25 7003 6003]    16byte+2byte+2byte |
| 6      | networkTime | uint48   | 网络时间                                                     |
| ??     | extend      | VarByte  | 扩展字段，不超过10个字节？                                   |

#### verack

用于应答version，无消息体

#### ping

用于维护连接，当一段时间未收到某个节点任何信息后，发送该消息，若能得到pong消息的回馈，则节点依然保持连接，否则关闭连接并删除节点

| Length | Fields     | Type   | Remark |
| ------ | ---------- | ------ | ------ |
| 4      | randomCode | uint32 | 随机数 |

#### pong

用于断开网络连接

| Length | Fields     | Type   | Remark |
| ------ | ---------- | ------ | ------ |
| 4      | randomCode | uint32 | 随机数 |

#### getaddr

用于获取网络中可用节点的连接信息，无消息体

#### addr

用于应答getaddr，或向网络中宣告自身的存在，节点接收到该消息后，判断节点是否已知，如果是未知节点，则向网络中传播该消息

| Length | Fields    | Type            | Remark                               |
| ------ | --------- | --------------- | ------------------------------------ |
| ??     | addr_list | network address | 每个节点18字节（16字节IP+2字节port） |



### 5.2 交易协议

​         暂无

## 六、模块配置

//自有网络模块

[network]
network.self.server.port=8003
network.self.magic=68866996
network.self.max.in=100
network.self.max.out=10
network.self.seed.ip=115.159.216.58:8003,115.159.69.140:8003,123.206.200.74:8003

//跨链网络模块

network.moon.server.port=8003
network.moon.max.in=100
network.moon.max.out=10
network.moon.seed.ip=215.159.216.58:8003,215.159.69.140:8003,223.206.200.74:8003



## 七、Java特有的设计

[^说明]: 核心对象类定义,存储数据结构，......

## 八、补充内容

[^说明]: 上面未涉及的必须的内容

