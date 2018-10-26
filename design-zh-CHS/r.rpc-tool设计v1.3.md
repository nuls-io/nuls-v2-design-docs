# RPC-TOOL设计文档

[TOC]

## 一、总体描述

### 1.1 概述

#### 1.1.1 为什么要有《RPC-TOOL》

[^说明]: 介绍的存在的原因

* NULS 2.0根据功能划分模块，所有模块隔离，可以独立运行。
* 模块间的交互通过RPC调用接口
* 模块只要实现了所需接口，可以用任何语言实现
* 各模块都会使用RPC，因此封装为同一jar文件



#### 1.1.2 《RPC-TOOL》要做什么

[^说明]: 要做些什么事情，达到什么目的，目标是让非技术人员了解要做什么事情

所有模块间的数据交互都通过《RPC-TOOL》进行

- 启动RPC Server
- 注册当前模块的所有CMD命令
- 解析接收的CMD命令
- 根据CMD调用对应的方法，返回结果
- 启动RPC Client
- 与kernel交互



#### 1.1.3 《RPC-TOOL》在系统中的定位

[^说明]: 在系统中的定位，是什么角色，依赖哪些做哪些事情，可以被依赖用于做哪些事情

RPC-TOOL是底层应用，任何模块都会依赖RPC-TOOL



### 1.2 架构图

[^说明]: 图形说明的层次结构、组件关系，并通过文字进行说明

## 二、功能设计

### 2.1 功能架构图

[^说明]: 说明的功能设计，可以有层级关系，可以通过图形的形式展示，并用文字进行说明。

### 2.2 服务

[^说明]: 这里说明该对外提供哪些服务，每个服务的功能说明、流程描述、接口定义、实现中依赖的外部服务


### 2.3 内部功能

[^说明]: 这里说明该内部有哪些功能，每个功能的说明、流程描述、实现中依赖的外部服务，参考上面外部服务格式



## 三、接口设计

### 3.1 接口

#### status

- 接口说明
  kernel会定时向各模块推送当前系统的状态，该模块用于接收

- 请求示例
  ```json
  {
      "cmd":"status",
      "minVersion":1,
      "params":[
          {
              "service":[
                  "a",
                  "b",
                  "c"
              ],
              "available":true,
              "modules":{                
                  "moduleABC":{
                      "name":"moduleABC",
                      "status":"READY",
                      "available":false,
                      "addr":"127.0.0.1",
                      "port":19722,
                      "rpcList":[
                          {
                              "cmd":"shutdown",
                              "version":1
                          },
                          {
                              "cmd":"cmd1",
                              "version":1
                          },
                          {
                              "cmd":"conf_reset",
                              "version":1
                          },
                          {
                              "cmd":"terminate",
                              "version":1
                          }
                      ],
                      "dependsModule":[
                          "m2",
                          "m3"
                      ]
                  }
              }
          }
      ]
  }
  ```

- 请求参数说明

  | index | parameter           | required | type | description  |
  | ----- | :------------------ | :------- | :--- | ------------ |
  | 0     | modules_information | true     | map  | 所有模块信息 |


  modules_information

| parameter | required | type              | description          |
| --------- | -------- | ----------------- | -------------------- |
| service   | true     | string[]          | 本模块需要依赖的模块 |
| available | true     | boolean           | 本模块是否可提供服务 |
| modules   | true     | map<name, module> | 所有模块的信息       |


  module

| parameter     | required | type     | description        |
| ------------- | -------- | -------- | ------------------ |
| name          | true     | string   | 模块名称           |
| status        | true     | string   | 模块状态           |
| available     | true     | boolean  | 模块是否可提供服务 |
| addr          | true     | string   | 连接的ip地址/域名  |
| port          | true     | int      | 连接的端口         |
| rpcList       | true     | list     | 对外提供的命令列表 |
| dependsModule | true     | string[] | 所依赖的模块       |



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



#### shutdown

- 接口说明
  kernel调用该接口关闭模块（等待当前业务全部处理完成）

- 请求示例

  ```json
  {
      "cmd":"shutdown",
      "minVersion":1,
      "params":[]
  }
  ```

- 请求参数说明

  无

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



#### terminate

- 接口说明
  kernel调用该接口关闭模块（立即终止）

- 请求示例

  ```json
  {
      "cmd":"terminate",
      "minVersion":1,
      "params":[]
  }
  ```

- 请求参数说明

  无

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



#### confGet

- 接口说明
  kernel获取模块配置项

- 请求示例

  ```json
  {
      "cmd":"confGet",
      "minVersion":1,
      "params":[]
  }
  ```

- 请求参数说明

  无

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
      "result":{
          "key1": "value1",
          "key2": "value2"
      }
  } 
  ```

- 返回字段说明
  无



#### confSet

- 接口说明
  kernel设置模块配置项

- 请求示例

  ```json
  {
      "cmd":"confSet",
      "minVersion":1,
      "params":[
          {
          "key1":"value1",
          "key2":"value2"
          }
      ]
  }
  ```

- 请求参数说明

  无

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



#### confReset

- 接口说明
  kernel恢复模板配置为初始值

- 请求示例

  ```json
  {
      "cmd":"confReset",
      "minVersion":1,
      "params":[]
  }
  ```

- 请求参数说明

  无

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




## 五、协议




## 六、配置

[^说明]: 本必须要有的配置项

## 七、Java特有的设计

[^说明]: 核心对象类定义,存储数据结构，......

### Server端
- 启动Server
    ```
    // 端口不是必须，如果不指定端口，则系统随机分配
    BaseRpcServer server = new GrizzlyServer(RpcConstant.KERNEL_PORT);
    
    // 扫描cmd命令所在的包
    server.scanPackage("io.nuls.rpc.mycmd");
    
    // 初始化
    server.init("moduleName", "dependsModule, type is List<String>");
    
    // 启动
    server.start();
    ```

- 自定义cmd
   ```
   // 继承BaseCmd
   public class SomeCmd extends BaseCmd
   
   // 自定义方法添加注解CmdInfo
   @CmdInfo(cmd = "cmd1", version = 1.0, preCompatible = true)
   public Object methodName(List params) {
       System.out.println("I'm version 1");
       return success();
   }
   
   注：rpc-tool会自动把客户端传输的Object[] 转换为 List<Object>
   ```

- 必须要有且只有一个cmd类实现KernelCmd接口
    ```
    // 实现接口
    public class SomeCmd extends BaseCmd implements KernelCmd 
    
    // 具体内容各模块不同，自己定义
    /**
     * 从kernel接收所有模块信息
     */
    public Object status(List params);

    /**
     * 关闭服务：在已有业务完成之后
     */
    public Object shutdown(List params);

    /**
     * 关闭服务：立即关闭，不管业务是否完成
     */
    public Object terminate(List params);

    /**
     * 提供本地配置信息
     */
    public Object confGet(List params);

    /**
     * 更新本地配置信息
     */
    public Object confSet(List params);

    /**
     * 重置本地配置信息
     */
    public Object confReset(List params);
    ```

### Client端
- 向kernel提供本模块信息
    ```
    RpcClient.versionToKernel();
    ```

- 调用rpc
    ```
    // cmd只对应一个接口
    String jsonStr = RpcClient.callSingleRpc("shutdown", params, minVersion);
    
    // cmd对应多个接口
    String jsonStr = RpcClient.callMultiplyRpc("shutdown", params, minVersion);
    
    备注：
    params为Object[]     
    ```

### 数据传输
- 请求参数
    ```
    {
      "cmd": "shutdown",
      "minVersion": 1.0,  //根据自己需要传最低版本号
      "params": [],
    }
    ```

- 返回结果（成功和失败都一样）
    ```
    {
      "code":0,
      "msg": "这个属性只有失败的时候才有，成功的时候没有",
      "version": "真正调用的版本号",
      "result": {}
    }    
    ```
    

### 其他
1. BaseCmd中有status方法的默认实现，因此如果大家没有特殊需求，只需要通过如下方法：
    ```
    @Override
    @CmdInfo(cmd = RpcConstant.STATUS, version = 1.0, preCompatible = true)
    public Object status(List params){
        return super.status(params);
    }
    ```
    
2. BaseCmd中有返回成功和失败的方法  
    成功：  
    ```
    protected Object success(double version) {
        return success(version, null);
    }

    protected Object success(double version, Object result) {
        Map<String, Object> map = new HashMap<>(16);
        map.put("code", 0);
        map.put("msg", SUCCESS);
        map.put("version", version);
        map.put("result", result);
        return map;
    }
    ```
    
    失败：  
    ```
    protected Object fail(String code, String msg, double version, Object result) {
        Map<String, Object> map = new HashMap<>(16);
        map.put("code", code);
        map.put("msg", msg);
        map.put("version", version);
        map.put("result", result);
        return map;
    }
    ```



## 八、补充内容

[^说明]: 上面未涉及的必须的内容

