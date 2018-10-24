#  事件总线模块设计

[TOC]

## 一、模块架构

### 1.1 模块概述

> 事件总线模块是专门用来接收及通知处理模块事件的功能性模块，管理着所有模块事件，并且提供事件的创建、订阅、接收、发送等功能，是模块间的事件中转站。

> [^注：]: 事件的订阅者与发送者是否需要验证

### 1.2 架构图

![event-bus-module](image/eventbus/event-bus-module.png)



### 1.3 模块上下文

![event-bus-content](image/eventbus/event-bus-content.png)


### 1.4 实现原理图

![event-bus-model](image/eventbus/event-bus-model.png)

## 二、功能设计

### 2.1 功能架构图

![event-bus-function](image/eventbus/event-bus-function.png)

- 微服务接口信息同步管理

  用于与kernel服务治理模块之间的服务接口同步

- 事件储存管理(eventBus)

  -    用于进行事件信息的创建，订阅等储存，并且在模块重启是进行信息的初始化。

- 事件订阅管理(subscribe)

  - 维护订阅事件的“配置表”：包括所有各个模块订阅的重要参数

- 事件转发管理(dispatcher)

    开放接口用于事件的接收，对接收事件按订阅进行转发，转发调用接口通过服务信息管理接口获得 。

- 功能接口管理(rpc)

    -   开放查询接口供外部查询


### 2.2 核心流程

* 事件处理时序

![event-bus-seq-flow](image/eventbus/event-bus-seq-flow.png)  

- 事件处理基本流程

![event-bus-main-flow](image/eventbus/event-bus-main-flow.png)

### 2.3 关键处理逻辑
> 在事件转发失败（比如网络原因）情况下进行的异常逻辑处理 ，按以下2种逻辑处理:

- 1、保留事件调用 按队列重复调用，直到转发成功。
- 2、尝试多次后直接丢弃,(暂定5次)每10秒后重试一次。

## 三、接口设计

### 3.1 模块接口
#### 3.1.1 事件主题订阅
> cmd: subscribe

##### 参数说明 (request body)

```json
{
  "jsonrpc": "1.0"，
  "method": "subscribe",
  "params":[
    "app.nuls.network.bandwidth",//topic 事件主题
    "moduleId", //moduleId订阅者模块id
  }
}
```
##### 返回值说明 (response content)

```json
{
  "jsonrpc": "1.0",
  "code": 0,//操作码
  "msg": "reponse message.",//失败时的信息
  "result": {
    "app_secret": "xxxxxxxxxxxx" // app_secret,暂时不需要，后期如果需要不是本机调用可能需要验证
  }
}
```

#### 3.1.2 事件取消订阅
> cmd: unsubscribe

##### 参数说明 (request body)

```json
{
"jsonrpc": "1.0"，
"method": "unsubscribe",
"params":[
  "app.nuls.network.bandwidth", //topic 事件主题
  "moduleId" //moduleId订阅者模块id
  ]
}
```

##### 返回值说明：(response content)

```json
{
  "jsonrpc": "1.0",
  "code": 0, //操作码
  "msg": "reponse message.",//失败时的信息
  "result": {
  }
}
```

#### 3.1.3 事件发送【自动创建topic】
> 在没人订阅情况下是否保留一定时间？

> cmd: send

##### 参数说明(request body)

```json
{
 "jsonrpc": "1.0"，
 "method": "send",
 "params":[
   "app.nuls.network.bandwidth",//topic 事件主题
   "moduleId", //moduleId订阅者模块id
   {data} // 需要发送的事件，jsonObj
 ]
}
```

##### 返回值说明(response content)

```json
{
  "jsonrpc": "1.0",
  "code": 0, //操作码
  "msg": "reponse message.",//失败时的信息
  "result": {
  }
}
```
#### 3.1.4 事件广播(推送push or dispatcher)
> 需要每个接口在订阅事件时提供接口，我在广播时调用即可,我这里是多线程去掉你们接口，你们需要返回正确的code,否则会有走重试机制

##### 参数说明(request body)

```json
{
 "jsonrpc": "1.0"，
 "method": "dispatcher",
 "params":[
   {} //data 需要发送的事件，payload
 ]
}
```

##### 返回值说明(response content)

```json
{
  "jsonrpc": "1.0",
  "code": 0, //一定要正确返回,不要需要告诉你业务逻辑是否出错，你只要接收到了就告诉我你成功接收到了即可。
  "msg": "reponse message.",//失败时的信息
  "result": {
  }
}
```
### 3.2 功能接口
> 功能接口是提供给界面和命令行工具使用的接口

#### 获取事件主题信息
>  cmd: topics

##### 参数说明(request body)

```json
{
 "jsonrpc": "1.0"，
 "method": "topics",
 "params":[]
}
```

##### 返回值说明(response content)

```json
{
  "jsonrpc": "1.0",
  "code": 0, //一定要正确返回,不要需要告诉你业务逻辑是否出错，你只要接收到了就告诉我你成功接收到了即可。
  "msg": "reponse message.",//失败时的信息
  "result": {
    "topics":[{
        topic: "",  //主题id
        createTime:"",	//创建时间
        moduleId:"",   //主题创建者（模块）Id
        subscribes:[//订阅者信息
            {
                moduleId:"", //订阅者
                subscribeTime:"" //订阅时间
            }
        ]}
    ]
  }
}
```

#### 获取事件主题信息(包含该主题上所有事件信息？) 每个事件其实我不关心的。

> cmd : get_topic
 
## 四、事件说明

该模块是最底层模块，不发送和订阅任何事件

## 五、协议

### 5.1 网络通讯协议

无

### 5.2 交易协议

无

## 六 模块依赖关系

- 内核模块
  - 模块注册
  - 模块注销
  - 模块状态上报（心跳）
  - 服务接口数据获取及定时更新

## 七、模块配置项
> 一般支持性配置，端口，重试次数，重试时间，默认执行器的线程池大小，网络调用超时配置等。
```yml
server:
  ip: 127.0.01   //本机ip，用于提供服务给其他模块,可以不填，默认自动获取
  port: 8080    //提供服务的端口,可以不填，默认自动获取
```

## 八、Java特有的设计

* 核心对象类定义

  待完善

* 存储数据结构

  待完善

#### 九、SDK提供给其他模块使用（已经废弃）
- 1 引入模块的依赖
- 2 注册订阅事件
- 3 发送事件
- 4 处理事件
- 5 取消注册订阅
- 6 网络连接检测

