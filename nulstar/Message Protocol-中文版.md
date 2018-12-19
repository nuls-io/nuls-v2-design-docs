# Nulstar MESSAGE PROTOCOL

##### V 1.1



## 1] 通信协议 – Json/WebSockets

微服务的行为不像标准的客户端 - 服务器基础设施，因为每个微服务同时是客户端和服务器，因此需要全双工通信协议，这允许实现特殊类型的发布 - 订阅模式; 在这个实现中，方法只能被调用一次，然后调用者可以通过两种不同的方式接收不断的更新：

- 基于事件：当方法在预定义数量的事件之后发送通知时
- 基于时间：当方法在预定义的秒数后发送通知时

WebSocket是一种成熟的选项，可以本机提供全双工连接，其他选项如标准Json-RPC不提供双向通道。
消息将使用JSon格式进行编码，因为它是最广泛用于消息交换的，并且易于调试。



## 2] 消息结构

所有消息都有一个由六个字段组成的公共基础结构：

- ProtocolVersion：表示调用者需要服务理解的协议版本，它由两个数字（主要和次要）组成，遵循语义规则，这意味着如果主要数字不同，则拒绝连接，如果次要数量不同则 可以建立成功的连接
- MessageID：这是一个标识请求的字符串。 它的长度不应超过256个字符
- Timestamp：自纪元以来的秒数（1970年1月1日格林威治标准时间00:00:00）
- TimeZone：发起请求的时区，它应为介于-12和12之间的数字
- MessageType：消息类型，这些在第3节中指定
- MessageData：保存消息有效负载的Json对象

示例：

```json
{
    "ProtocolVersion":"1.0",
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



## 3] Message Types

目前只定义了9种类型的消息：NegotiateConnection, NegotiateConnectionResponse, Request, Unsubscribe, Response, Ack, Notificatioin, RegisterCompoundMethod, UnregisterCompoundMethod。

#### 3.1] NegotiateConnection

这应该是在与微服务建立连接时应该发送的第一个对象，只有在协商成功时，服务才可以处理其他请求，否则应该收到状态设置为0（失败）的NegotiateConnectionResponse对象并立即断开连接。

它由两个字段组成：

- CompressionAlgorithm（默认值：zlib）：一个String，表示如果CompressionRate大于0，将用于接收和发送消息的算法。默认为zlib，大多数开发语言中都有支持的库。
- CompressionRate：0到9之间的一个整数，用于建立应为此连接发送和接收消息的压缩级别。 0表示没有压缩，而9表示最大压缩

示例：

```json
{
    "ProtocolVersion":"1.0",
    "MessageID":"45sdj8jcf8899ekffEFefee",
    "Timestamp":"1542102459",
    "TimeZone":"-4",
    "MessageType":"NegotiateConnection",
    "MessageData":{
        "CompressionAlgorithm":"zlib",
        "CompressionRate":"3"
    }
}
```



#### 3.2] NegotiateConnectionResponse

仅在响应先前传入的NegotiateConnection消息时发送此类消息。 它由两个字段组成：

- NegotiationStatus：无符号的小整数值，如果协商失败则为0，如果成功则为1
- NegotiationComment：一个字符串值，用于描述拒绝连接时出现了什么问题。

示例：

```json
{
    "ProtocolVersion":"1.0",
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



#### 3.3] Request

调用者服务必须发送一个Request对象来执行Nulstar网络内某些服务提供的方法。

如果在单个请求对象中包含两个或更多方法，则应按顺序执行方法，然后发送一个response包喊所有回执信息，如果其中一个请求失败，则整个操作被视为失败。

它由六个字段组成：

- RequestAck（默认值：false）：这是一个布尔值

  - false：发出的请求只需要一条Response消息，如果它订阅了该函数，那么它可能会有很多响应消息
  - true：发出的请求需要Ack和Response（译者注：有的请求可能需要一段时间处理，不会立刻返回Response，但是我要确保对方接收到了我的请求），如果它订阅了该函数，那么它可能会有很多响应消息

- SubscriptionEventCounter（默认值：0）：这是一个无符号整数，指定目标方法发送Response的区块间隔。不管这个值是多少，总会立刻发送一个Response。如果是0，则不再继续发送，如果是2，则每2个块都会检测是否发送。以此类推。

- SubscriptionPeriod（默认值：0）：这是一个无符号整数，指定目标方法发送Response的时间间隔。不管这个值是多少，总会立刻发送一个Response。如果是0，则不再继续发送，如果是2，则每2秒都会检测是否发送。以此类推。

- SubscriptionRange：如果请求的事件返回的是一个数字，则定义何时返回这个数字。
  这是一个字符串，表示将触发响应的条件。 字符串是一对带符号的十进制数，第一个是下限，第二个是上限，如果不可用则为空。 如果该对分别以“（”或“）”开始或结束，则表示该数字不包括在内，如果该对分别以“[”或“]”开始或结束，则表示该数字包括在内。

  示例：假设我们只希望仅在余额等于或大于1000时收到通知。然后，getbalance请求应以“[1000，）”字符串作为SubscriptionRange参数发送

- ResponseMaxSize（默认值：0）：无符号整数，指定方法应返回的最大对象数，值为零（默认值）表示无限制
- RequestMethods：一个数组，包含所请求的所有方法及其各自的参数

示例：

```json
{
    "ProtocolVersion":"1.0",
    "MessageID":"45sdj8jcf8899ekffEFefee",
    "Timestamp":"1542102459",
    "TimeZone":"-4",
    "MessageType":"Request",
    "MessageData":{
        "RequestAck":"0",
        "SubscriptionEventCounter":"3",
        "SubscriptionPeriod":"0",
        "SubscriptionRange":"0",
        "ResponseMaxSize":"0",
        "RequestMethods":{
            "GetBalance":{
                "Address":"N234rFr4Rtgg5ref4$45tgg5f43335emcnd"
            },
            "GetHeight":{

            }
        }
    }
}
```



#### 3.4] Unsubscribe

当服务不再希望从其订阅的方法接收响应时，它必须向目标服务发送Unsubscribe消息。

它由一个字段组成：

- UnsubscribeMethods：一个数组，包含调用者想要取消订阅的所有方法

示例：

```json
{
    "ProtocolVersion":"1.0",
    "MessageID":"45sdj8jcf8899ekffEFefee",
    "Timestamp":"1542102459",
    "TimeZone":"-4",
    "MessageType":"Unsubscribe",
    "MessageData":{
        "UnsubscribeMethods":[
            "GetBalance",
            "GetHeight"
        ]
    }
}
```



#### 3.5] Response

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
    "ProtocolVersion":"1.0",
    "MessageID":"767345sdfgsd99qwe",
    "Timestamp":"1542102459",
    "TimeZone":"-4",
    "MessageType":"Response",
    "MessageData":{
        "RequestID":"45sdj8jcf8899ekffEFefee",
        "ResponseProcessingTime":"13",
        "ResponseStatus":"1",
        "ResponseComment":"Congratulations! Processing completed！",
        "ResponseMaxSize":"0",
        "ResponseData":{
            "getBalance":{
                "Balance":"25000"
            },
            "getHeight":{
                "Height":"45454655454"
            }
        }
    }
}
```





#### 3.6] Ack

当RequestType为2或3时，将发送此消息类型（请参阅第3.3节）。 其唯一目的是通知呼叫者已成功接收请求。

它由一个字段组成：

- RequestID：这是引用的原始Request消息请求ID

示例：

```json
{
    "ProtocolVersion":"1.0",
    "MessageID":"45sdj8jcf8899ekffEFefee",
    "Timestamp":"1542102459",
    "TimeZone":"-4",
    "MessageType":"Ack",
    "MessageData":{
        "RequestID":"sdj8jcf8899ekffEFefee"
    }
}
```





#### 3.7] Notification

当需要将某些事件通知给连接的组件而不期望响应消息时（例如，即将对服务执行升级时），将发送此消息类型。 通知将信息推送到其他组件，因此通知只应由Manager，Controller和Connector模块使用

它由四个字段组成：

- NotificationAck :(默认值：false）：这是一个布尔值
  - false：发出的通知不期望任何类型的消息作为回执
  - true：发出的通知需要一条Ack消息
- NotificationType：通知的类别，每个服务都可以定义自己的类型，因此不需要接收方处理此字段
- NotificationComment：字符串注释，提供有关通知原因的更多信息
- NotificationData：与通知相关的数据，接收方不需要处理此字段

示例：

```json
{
    "ProtocolVersion":"1.0",
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





#### 3.8] RegisterCompoundMethod

如3.3中所述，请求可以由多个方法组成，使用此消息类型，我们注册一个虚拟方法，该方法将按顺序执行其各个实际方法，如果其子方法之一失败，则虚方法返回失败。

某些子方法可能共享相同的参数名称，因此可以创建别名，如示例中所示。

它由三个字段组成：

- CompoundMethodName：这是标识虚方法的字符串
- CompoundMethodDescription：描述方法功能，在查询API以获取帮助时将提供描述
- CompoundMethods：这是一个数组，包含构成虚方法的各自参数别名的方法

示例：

```json
{
    "ProtocolVersion":"1.0",
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



#### 3.9] UnregisterCompoundMethod

此消息类型用于取消注册复合（虚拟）方法。

它由一个字段组成：

- UnregisterCompoundMethodName：这是标识虚方法的字符串。 如果它为空，则应取消注册调用者注册的所有虚拟方法

示例：

```json
{
    "ProtocolVersion":"1.0",
    "MessageID":"45sdj8jcf8899ekffEFefee",
    "Timestamp":"1542102459",
    "TimeZone":"-4",
    "MessageType":"UnregisterCompoundMethod",
    "MessageData":{
        "UnregisterCompoundMethodName":"GetMyInfo"
    }
}
```

