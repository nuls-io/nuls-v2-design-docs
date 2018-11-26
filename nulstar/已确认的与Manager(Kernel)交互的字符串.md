## Module向Manager发送握手信息

```json
{
    "MessageID":"1542345820430",
    "Timestamp":"542345820",
    "TimeZone":"-4",
    "MessageType":"NegotiateConnection",
    "MessageData":{
        "CompressionRate":"0",
        "CompressionAlgorithm":"zlib"
    }
}
```





## Manager向Module发送握手确认信息

```json
{
    "MessageID":"1542345821232",
    "Timestamp":"542345821",
    "TimeZone":"-4",
    "MessageType":"NegotiateConnectionReponse",
    "MessageData":{
        "NegotiationStatus":"1",
        "NegotiationComment":"Connection successful!"
    }
}
```





## Module向Manager注册方法

```json
{
    "messageId":2,
    "timestamp":1542782457264,
    "timezone":9,
    "messageType":"Request",
    "messageData":{
        "subscriptionEventCounter":0,
        "subscriptionPeriod":0,
        "subscriptionRange":"",
        "responseMaxSize":0,
        "requestMethods":{
            "registerAPI":{
                "apiMethods":[
                    {
                        "methodName":"getHeight",
                        "methodDescription":"test getHeight 1.0",
                        "methodMinEvent":0,
                        "methodMinPeriod":0,
                        "methodScope":"private",
                        "parameters":[
                            {
                                "parameterName":"aaa",
                                "parameterType":"int",
                                "parameterValidRange":"",
                                "parameterValidRegExp":""
                            },
                            {
                                "parameterName":"bbb",
                                "parameterType":"string",
                                "parameterValidRange":"",
                                "parameterValidRegExp":""
                            }
                        ],
                        "version":1
                    },
                    {
                        "methodName":"getHeight",
                        "methodDescription":"test getHeight 1",
                        "methodMinEvent":0,
                        "methodMinPeriod":0,
                        "methodScope":"private",
                        "parameters":[

                        ],
                        "version":2
                    },
                    {
                        "methodName":"getHeight",
                        "methodDescription":"test getHeight 1",
                        "methodMinEvent":1,
                        "methodMinPeriod":10,
                        "methodScope":"public",
                        "parameters":[

                        ],
                        "version":1.3
                    }
                ],
                "serviceSupportedAPIVersions":[

                ],
                "serviceDomain":null,
                "serviceName":null,
                "serviceRole":null,
                "serviceVersion":null,
                "abbr":"cm",
                "name":"Chain Manager",
                "address":"192.168.1.65",
                "port":14922
            }
        }
    }
}
```

