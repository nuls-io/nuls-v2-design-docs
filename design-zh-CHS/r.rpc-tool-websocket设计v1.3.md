# Websocket-Tool设计文档——第四版

[TOC]




## 写在最前的话
```
如果想知道websocket的具体设计，请逐步阅读本文档

如果只想知道如何使用，请跳转到章节《7.1 如何使用》
```


## 一、总体描述 

### 1.1 概述

#### 1.1.1 为什么要有《Websocket-Tool》

[^说明]: 介绍的存在的原因

* NULS 2.0根据功能划分模块，所有模块隔离，可以独立运行。
* 模块间通过《Websocket-Tool》调用接口
* 模块只要实现了规定接口，可以用任何语言实现
* 模块可以分布式部署
* 《Websocket-Tool》会打包成jar包，供各模块引用



#### 1.1.2 《Websocket-Tool》要做什么

[^说明]: 要做些什么事情，达到什么目的，目标是让非技术人员了解要做什么事情

所有模块间的接口调用都通过《Websocket-Tool》进行

- 启动Websocket Server
- 注册当前模块的所有cmd命令
- 把当前模块信息汇报给kernel
- 接收从kernel推送的所有模块信息
- 启动Websocket Client
- 维护调用过程产生的数据
- 封装所有数据中间处理过程，各模块只需要关心
  - 输入
  - 输出
- 各模块通过尽可能简单的方式调用cmd命令



#### 1.1.3 《Websocket-Tool》在系统中的定位

[^说明]: 在系统中的定位，是什么角色，依赖哪些做哪些事情，可以被依赖用于做哪些事情

《Websocket-Tool》是底层框架，任何模块都会依赖

《Websocket-Tool》维护各模块基础信息，但是不涉及具体业务



## 二、功能设计

### 2.1 架构图

[^说明]: 说明的功能设计，可以有层级关系，可以通过图形的形式展示，并用文字进行说明。



## 三、接口设计



## 四、事件说明



## 五、协议

### 5.1 通信协议 – Json/WebSockets

微服务的行为不像标准的客户端 - 服务器基础设施，因为每个微服务同时是客户端和服务器，因此需要全双工通信协议，这允许实现特殊类型的发布 - 订阅模式; 在这个实现中，方法只能被调用一次，然后调用者可以通过两种不同的方式接收不断的更新：

- 基于事件：当方法在预定义数量的事件之后发送通知时
- 基于时间：当方法在预定义的秒数后发送通知时

WebSocket是一种成熟的选项，可以本机提供全双工连接，其他选项如标准Json-RPC不提供双向通道。
消息将使用JSon格式进行编码，因为它是最广泛用于消息交换的，并且易于调试。

### 5.2 消息结构

所有消息都有一个由5个字段组成的公共基础结构：

- MessageID：这是一个标识请求的字符串。 它的长度不应超过256个字符
- Timestamp：自纪元以来的秒数（1970年1月1日格林威治标准时间00:00:00）
- TimeZone：发起请求的时区，它应为介于-12和12之间的数字
- MessageType：消息类型，这些在第3节中指定
- MessageData：保存消息有效负载的Json对象

示例：

```json
{  
    "MessageID":"45sdj8jcf8899ekffEFefee",
    "Timestamp":"1542102459",
    "TimeZone":"-4",
    "MessageType":"NegotiateConnection",
    "MessageData":{
        "CompressionRate":"3",
        "CompressionAlgorithm":"zlib"
    }
}
```

### 5.3 Message Types

目前只定义了9种类型的消息：NegotiateConnection, NegotiateConnectionResponse, Request, Unsubscribe, Response, Ack, Notificatioin, RegisterCompoundMethod, UnregisterCompoundMethod



#### 5.3.1 NegotiateConnection

这应该是在与微服务建立连接时应该发送的第一个对象，只有在协商成功时，服务才可以处理其他请求，否则应该收到状态设置为0（失败）的NegotiateConnectionResponse对象并立即断开连接。

它由两个字段组成：

- CompressionAlgorithm（默认值：zlib）：一个String，表示如果CompressionRate大于0，将用于接收和发送消息的算法。默认为zlib，大多数开发语言中都有支持的库。
- CompressionRate：0到9之间的一个整数，用于建立应为此连接发送和接收消息的压缩级别。 0表示没有压缩，而9表示最大压缩

示例：

```json
{
    "messageId":"1",
    "timestamp":"1543133816985",
    "timezone":"9",
    "messageType":"NegotiateConnection",
    "messageData":{
        "protocolVersion":"1.0",
        "compressionAlgorithm":"zlib",
        "compressionRate":"0"
    }
}
```



#### 5.3.2 NegotiateConnectionResponse

仅在响应先前传入的NegotiateConnection消息时发送此类消息。 它由两个字段组成：

- NegotiationStatus：无符号的小整数值，如果协商失败则为0，如果成功则为1
- NegotiationComment：一个字符串值，用于描述拒绝连接时出现了什么问题。

示例：

```json
{
    "MessageID":"45sdj8jcf8899ekffEFefee",
    "Timestamp":"1542102459",
    "TimeZone":"-4",
    "MessageType":"NegotiateConnectionReponse",
    "MessageData":{
        "NegotiationStatus":"0",
        "NegotiationComment":"Incompatible protocol version"
    }
}
```



#### 5.3.3 Request

调用者服务必须发送一个Request对象来执行Nulstar网络内某些服务提供的方法。

如果在单个请求对象中包含两个或更多方法，则应按顺序执行方法，然后发送一个response包喊所有回执信息，如果其中一个请求失败，则整个操作被视为失败。

它由六个字段组成：

- RequestAck（默认值：0）：这是一个布尔值

  - 0：发出的请求只需要一条Response消息，如果它订阅了该函数，那么它可能会有很多响应消息
  - 1：发出的请求需要Ack和Response（译者注：有的请求可能需要一段时间处理，不会立刻返回Response，但是我要确保对方接收到了我的请求），如果它订阅了该函数，那么它可能会有很多响应消息

- SubscriptionEventCounter（默认值：0）：这是一个无符号整数，指定目标方法发送Response的值的`改变次数`。不管这个值是多少，总会立刻发送一个Response。如果是0，则不再继续发送，如果是2，则每改变两次就发送。以此类推。

- SubscriptionPeriod（默认值：0）：这是一个无符号整数，指定目标方法发送Response的时间间隔。不管这个值是多少，总会立刻发送一个Response。如果是0，则不再继续发送，如果是2，则每2秒都会检测是否发送。以此类推。

- SubscriptionRange：如果请求的事件返回的是一个数字，则定义何时返回这个数字。
  这是一个字符串，表示将触发响应的条件。 字符串是一对带符号的十进制数，第一个是下限，第二个是上限，如果不可用则为空。 如果该对分别以“（”或“）”开始或结束，则表示该数字不包括在内，如果该对分别以“[”或“]”开始或结束，则表示该数字包括在内。

  示例：假设我们只希望仅在余额等于或大于1000时收到通知。然后，getbalance请求应以“[1000，）”字符串作为SubscriptionRange参数发送

- ResponseMaxSize（默认值：0）：无符号整数，指定方法应返回的最大对象数，值为零（默认值）表示无限制。只针对返回结果为List有效，指定的是返回的记录个数。

- RequestMethods：一个数组，包含所请求的所有方法及其各自的参数

示例：

```json
{
    "messageId":"3",
    "timestamp":"1543133968578",
    "timezone":"9",
    "messageType":"Request",
    "messageData":{
        "requestAck":"0",
        "subscriptionEventCounter":"0",
        "subscriptionPeriod":"5",
        "subscriptionRange":"0",
        "responseMaxSize":"0",
        "requestMethods":{
            "getHeight":{
                "paramName":"value",
                "version":"1.0"
            }
        }
    }
}
```



#### 5.3.4 Unsubscribe

当服务不再希望从其订阅的方法接收响应时，它必须向目标服务发送Unsubscribe消息。

它由一个字段组成：

- UnsubscribeMethods：一个数组，包含调用者想要取消订阅的所有方法

示例：

```json
{
    "messageId":"4",
    "timestamp":"1543134296019",
    "timezone":"9",
    "messageType":"Unsubscribe",
    "messageData":{
        "unsubscribeMethods":[
            "getHeight"
        ]
    }
}
```



#### 5.3.5 Response

当目标服务完成处理请求时，应该发送响应以及操作结果。

它由六个字段组成：

- RequestID：这是引用的原始Request消息请求ID
- ResponseProcessingTime：目标服务处理请求所用的时间（以毫秒为单位）
- ResponseStatus：响应状态，如果成功则为1，否则为0
- ResponseComment：一个字符串，可以提供有关该过程结果的更多说明
- ResponseMaxSize：响应包含每个请求的最大对象数
- ResponseData：一个对象数组，包含已处理方法的结果，每个请求一个对象

示例：

```json
{
    "messageId":"9",
    "timestamp":"1543134299030",
    "timezone":"9",
    "messageType":"Response",
    "messageData":{
        "requestId":"5",
        "responseProcessingTime":"1",
        "responseStatus":"1",
        "responseComment":"Congratulations! Processing completed！",
        "responseMaxSize":"0",
        "responseData":{
            "getHeight":"getHeight->1.3"
        }
    }
}
```



#### 3.5.6 Ack

其唯一目的是通知呼叫者已成功接收请求。

它由一个字段组成：

- RequestID：这是引用的原始Request消息请求ID

示例：

```json
{
    "MessageID":"45sdj8jcf8899ekffEFefee",
    "Timestamp":"1542102459",
    "TimeZone":"-4",
    "MessageType":"Ack",
    "MessageData":{
        "RequestID":"sdj8jcf8899ekffEFefee"
    }
}
```



#### 3.5.7 Notification

当需要将某些事件通知给连接的组件而不期望响应消息时（例如，即将对服务执行升级时），将发送此消息类型。 通知将信息推送到其他组件，因此通知只应由Manager，Controller和Connector模块使用

它由四个字段组成：

- NotificationAck :(默认值：0）：这是一个布尔值
  - 0：发出的通知不期望任何类型的消息作为回执
  - 1：发出的通知需要一条Ack消息
- NotificationType：通知的类别，每个服务都可以定义自己的类型，因此不需要接收方处理此字段
- NotificationComment：字符串注释，提供有关通知原因的更多信息
- NotificationData：与通知相关的数据，接收方不需要处理此字段

示例：

```json
{
    "MessageID":"45sdj8jcf8899ekffEFefee",
    "Timestamp":"1542102459",
    "TimeZone":"-4",
    "MessageType":"Notification",
    "MessageData":{
        "NotificationAck":"1",
        "NotificationType":"SystemUpgrade",
        "NotificationComment":"A system upgrade is about to be performed!",
        "NotificationData":{
            "Date":"2018-11-11",
            "Time":"23:00:00",
            "NewVersion":"1.1.6"
        }
    }
}
```



#### 3.5.8 RegisterCompoundMethod

请求可以由多个方法组成，使用此消息类型，我们注册一个虚拟方法，该方法将按顺序执行其各个实际方法，如果其子方法之一失败，则虚方法返回失败。

某些子方法可能共享相同的参数名称，因此可以创建别名，如示例中所示。

它由三个字段组成：

- CompoundMethodName：这是标识虚方法的字符串
- CompoundMethodDescription：描述方法功能，在查询API以获取帮助时将提供描述
- CompoundMethods：这是一个数组，包含构成虚方法的各自参数别名的方法

示例：

```json
{
    "MessageID":"45sdj8jcf8899ekffEFefee",
    "Timestamp":"1542102459",
    "TimeZone":"-4",
    "MessageType":"RegisterCompoundMethod",
    "MessageData":{
        "CompoundMethodName":"GetMyInfo",
        "CompoundMethodDescription":"Get useful information.",
        "CompoundMethods":{
            "GetBalance":{
                "Address":"GetBalanceAddress"
            },
            "GetHeight":{

            }
        }
    }
}
```

在该示例中，正在注册一个名为GetMyInfo的虚方法，它由GetBalance和GetHeight方法组成，还为Address参数创建了一个名为GetBalanceAddress的别名。
GetMyInfo的请求可以作为标准方法发送。

（我确实没看懂这个的意义在哪，感觉就是单纯为了多这么个功能，有疑问请直接找Berzeck或者坚哥，哈哈）



#### 3.5.9 UnregisterCompoundMethod

此消息类型用于取消注册复合（虚拟）方法。

它由一个字段组成：

- UnregisterCompoundMethodName：这是标识虚方法的字符串。 如果它为空，则应取消注册调用者注册的所有虚拟方法

示例：

```json
{
    "MessageID":"45sdj8jcf8899ekffEFefee",
    "Timestamp":"1542102459",
    "TimeZone":"-4",
    "MessageType":"UnregisterCompoundMethod",
    "MessageData":{
        "UnregisterCompoundMethodName":"GetMyInfo"
    }
}
```



### 5.4 具体细节

- 传输过程的所有属性都是string类型

- 如果是boolean类型，"1"代表true，"0"代表false

- 当调用一个方法的时候，调用者需要知道：提供方法的角色，以及方法的使用方式

- 注册的时候，Role1有method1，Role2有method2，如何定义？
  答：不需要定义，这是写在文档中的。
  apiMethods = Role1Methods + Role2Methods。因此注册的时候不需要知道每个Role都包含什么方法，这些应该在文档中体现。

- 注册接口的字符串格式

  ```json
  {
      "messageId":"2",
      "timestamp":"1543204508986",
      "timezone":"9",
      "messageType":"Request",
      "messageData":{
          "requestAck":"0",
          "subscriptionEventCounter":"0",
          "subscriptionPeriod":"0",
          "subscriptionRange":"0",
          "responseMaxSize":"0",
          "requestMethods":{
              "registerAPI":{
                  "apiMethods":[
                      {
                          "methodName":"getHeight",
                          "methodDescription":"test getHeight 1.1",
                          "methodMinEvent":"0",
                          "methodMinPeriod":"0",
                          "methodScope":"private",
                          "parameters":[
                              {
                                  "parameterName":"aaa",
                                  "parameterType":"int",
                                  "parameterValidRange":"(1,100]",
                                  "parameterValidRegExp":""
                              },
                              {
                                  "parameterName":"bbb",
                                  "parameterType":"string",
                                  "parameterValidRange":"",
                                  "parameterValidRegExp":"^[A-Za-z0-9\\-]+$"
                              }
                          ]
                      },
                      {
                          "methodName":"getHeight",
                          "methodDescription":"test getHeight 2.0",
                          "methodMinEvent":"0",
                          "methodMinPeriod":"0",
                          "methodScope":"private",
                          "parameters":[
  
                          ]
                      }
                  ],
                  "dependencies":{
                      "Role_Ledger":"1.1"
                  },
                  "connectionInformation":{
                      "IP":"192.168.1.65",
                      "Port":"17733"
                  },
                  "moduleDomain":"nuls.io",
                  "moduleRoles":{
                      "cm":[
                          "1.1",
                          "1.2"
                      ]
                  },
                  "moduleVersion":"1.2",
                  "moduleAbbreviation":"cm",
                  "moduleName":"Chain Manager"
              }
          }
      }
  }
  ```

- Manager返回各模块连接信息的字符串格式

  ```json
  {
      "messageId":"8",
      "timestamp":"1543204714006",
      "timezone":"9",
      "messageType":"Response",
      "messageData":{
          "requestId":"2",
          "responseProcessingTime":"1",
          "responseStatus":"1",
          "responseComment":"Congratulations! Processing completed！",
          "responseMaxSize":"0",
          "responseData":{
              "registerAPI":{
                  "Dependencies":{
                      "test":{
                          "APIVersion":[
                              "1.0"
                          ],
                          "IP":"192.168.1.65",
                          "Port":"14694"
                      },
                      "ke":{
                          "APIVersion":null,
                          "IP":"192.168.1.65",
                          "Port":"8887"
                      },
                      "cm":{
                          "APIVersion":[
                              "1.1",
                              "1.2"
                          ],
                          "IP":"192.168.1.65",
                          "Port":"17733"
                      }
                  }
              }
          }
      }
  }
  ```





## 六、配置



## 七、Java特有的设计

[^说明]: 核心对象类定义,存储数据结构，......

### 7.1 设计

>  io.nuls.rpc
>
>  >  client
>  >
>  >  > `ClientProcessor`：处理服务器消息的线程
>  >  >
>  >  > `ClientRuntime`：客户端运行时需要的数据，方法
>  >  >
>  >  > `CmdDispatcher`：发送消息的入口类
>  >  >
>  >  > `WsClient`：与其他模块建立连接的对象
>  >
>  >  cmd
>  >
>  >  > cmd_package_1
>  >  >
>  >  > cmd_package_2
>  >  >
>  >  > `BaseCmd`：所有对外提供方法的类的父类，提供success, failed方法返回Response对象
>  >
>  >  info
>  >
>  >  > `Constants`：常量
>  >  >
>  >  > `HostInfo`：获取IP地址，随机获得端口
>  >
>  >  model
>  >
>  >  > message
>  >  >
>  >  > > `Message`：所有消息都应该用该对象进行传输
>  >  > >
>  >  > > `MessageType`：消息类型（包含以下9种）
>  >  > >
>  >  > > `Ack`：确认收到消息
>  >  > >
>  >  > > `NegotiateConnection`：握手
>  >  > >
>  >  > > `NegotiateConnectionResponse`：回复握手
>  >  > >
>  >  > > `Notification`：通知
>  >  > >
>  >  > > `Request`：请求调用远程方法
>  >  > >
>  >  > > `Response`：回复Request
>  >  > >
>  >  > > `Unsubscribe`：取消订阅
>  >  > >
>  >  > > `RegisterCompoundMethod`：订阅多个远程方法
>  >  > >
>  >  > > `UnregisterCompoundMethod`：取消订阅多个远程方法
>  >  >
>  >  > `CmdAnnotation`：注解类，有该注解的方法可以对外提供接口
>  >  >
>  >  > `Parameter`：注解类，用以描述对外提供接口的参数信息
>  >  >
>  >  > `Parameters`：注解类，Parameter的集合
>  >  >
>  >  > `CmdDetail`：对外提供的方法的具体信息
>  >  >
>  >  > `CmdParameter`：对外提供的方法的参数信息
>  >  >
>  >  > `ModuleE`：枚举类型，NULS2.0基础架构下的模块信息
>  >  >
>  >  > `RegisterApi`：一个模块对外提供的所有方法的合集
>  >
>  >  server
>  >
>  >  > `CmdHandler`：根据Request消息，调用正确的方法
>  >  >
>  >  > `ServerProcessor`：处理客户端消息的线程
>  >  >
>  >  > `ServerRuntime`：服务器运行时需要的参数，方法
>  >  >
>  >  > `WsServer`：与WsClient连接的服务器对象



### 7.2 如何使用  

Websocket-Tool会做成JAR包供各模块引用  



#### 7.2.1 测试专用：模拟kernel

非常重要！

各模块接口是在kernel中进行维护，但是kernel由社区成员开发，因此这一部分是内部测试的模拟代码，可以直接复制使用，无需额外操作。

（test/java/io.nuls.test.WsKernel）

```java
@Test
public void kernel() throws Exception {
    // url: "ws://127.0.0.1:8887"
    WsServer.mockKernel();
}
```



#### 7.2.2 自定义cmd

scope的值：public，private，admin

- public：暴露出去，第三方应用/平台也能调用的公开接口
- private：只有模块间内部才能调用的接口
- admin：专门为管理员设计的特定接口（管理员定义在在Berzeck的connector中，我们并不需要关心）

```java
/*
 * 该类所在的包需要通过7.1.3中的方法进行扫描
 */
public class MyCmd extends BaseCmd {

    @CmdAnnotation(cmd = "getHeight", version = 1.0, scope = "private", minEvent = 0, minPeriod = 0,
            description = "test getHeight 1.0")
    @Parameter(parameterName = "aaa", parameterType = "int", parameterValidRange = "(1,100]", parameterValidRegExp = "")
    @Parameter(parameterName = "bbb", parameterType = "string")
    public Object getHeight1(Map map) {
        Log.info("getHeight version 1");
        // success
        return success("Here is your real return value");
        // 预定义错误
        return failed(ErrorCode);
        // 非预定义错误
        return failed(String)
    }
}
```



#### 7.2.3 启动Server

```java
// Start server instance
WsServer.getInstance(ModuleE.CM)
    .moduleRoles(new String[]{"1.0", "2.4"})
    .moduleVersion("1.2")
    .dependencies(ModuleE.LG.abbr, "1.1")
    .dependencies(ModuleE.BL.abbr, "2.1")
    .scanPackage("io.nuls.rpc.cmd.test")
    .connect("ws://127.0.0.1:8887");

// Get information from kernel
CmdDispatcher.syncKernel();
```



#### 7.2.4 为kernel提供的接口

现阶段忽略！

```java

```



#### 7.2.5 调用cmd

```java
/*
  单元测试专用：单元测试时需要告知内核地址，以及同步接口列表
  如果不是单元测试，在模块中进行连调测试，下面两句话是不需要的
  */
WsServer.mockModule();
/*
  单元测试专用结束
  */


// Build params map
Map<String, Object> params = new HashMap<>();
// Version information ("1.1" or 1.1 is both available)
params.put(Constants.VERSION_KEY_STR, "1.0");
params.put("paramName", "value");

// 可以看成是一个同步方法，发送Request，获得Response
Response response = CmdDispatcher.requestAndResponse(ModuleE.CM.abbr, "getHeight", params);

// 发送Request，当有Response的时候会自动调用预设的方法，返回的messageId是为了取消订阅
String messageId = CmdDispatcher.requestAndInvoke(ModuleE.CM.abbr, "getHeight", params, "1", InvokeMethod.class, "invokeGetHeight2");

// 与requestAndInvoke一样，但是必须在收到Ack之后才会返回messageId
String messageId = CmdDispatcher.requestAndInvokeWithAck(ModuleE.CM.abbr, "getHeight", params, "1", InvokeMethod.class, "invokeGetHeight2");

// 取消订阅
CmdDispatcher.unsubscribe(messageId);
System.out.println("我已经取消了订阅");

```





## 八、补充内容

[^说明]: 上面未涉及的必须的内容

