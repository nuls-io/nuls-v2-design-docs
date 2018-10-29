# RPC-TOOL design document

[TOC]

## 1. Overall description 

### 1.1 summary

#### 1.1.1 position of RPC-TOOL

* NULS 2 according to function partition module, all modules are isolated and can run independently.
*  Interaction between modules through RPC call interface 
* As long as the module implements the required interface, it can be implemented in any language
* Each module will use RPC, so it will be encapsulated into the same jar file



#### 1.1.2 what will RPC-TOOL did

Data interaction between modules is done through RPC-TOOL

- RPC Server start
- Register all CMD commands of the current module
- Parsing the received CMD command
- Returns the result by calling the corresponding method based on CMD
- RPC Client start
- Interacting with kernel



#### 1.1.3 position

[^说明]: 在系统中的定位，是什么角色，依赖哪些做哪些事情，可以被依赖用于做哪些事情

RPC-TOOL is the underlying application, and any module will depend on RPC-TOOL.



## 2.  functional design 



## 3.  interface design 

### 3.1 interface

#### status

- Interface description 
    Kernel will periodically push the status of the current system to each module, which is used for receiving

- Request example
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

- Request parameter specification

  | index | parameter           | required | type | description            |
  | ----- | :------------------ | :------- | :--- | ---------------------- |
  | 0     | modules_information | true     | map  | All module information |

  modules_information

  | parameter | required | type              | description            |
  | --------- | -------- | ----------------- | ---------------------- |
  | service   | true     | string[]          | The dependent modules  |
  | available | true     | boolean           | can start service?     |
  | modules   | true     | map<name, module> | All module information |

  module

  | parameter     | required | type     | description           |
  | ------------- | -------- | -------- | --------------------- |
  | name          | true     | string   | name                  |
  | status        | true     | string   | status                |
  | available     | true     | boolean  | can start service?    |
  | addr          | true     | string   | ip address/host name  |
  | port          | true     | int      | port                  |
  | rpcList       | true     | list     | cmd list              |
  | dependsModule | true     | string[] | The dependent modules |


- Response example
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

- Response field specification
  N/A



#### shutdown

- Interface description
  Kernel calls the interface to close the module (waiting for all the processing of the current business to complete)

- Request example

  ```json
  {
      "cmd":"shutdown",
      "minVersion":1,
      "params":[]
  }
  ```

- Request parameter specification

  N/A

- Response example
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

- Response field specification
  N/A



#### terminate

- Interface description
  Kernel calls the interface to close the module (immediately terminates).

- Request example

  ```json
  {
      "cmd":"terminate",
      "minVersion":1,
      "params":[]
  }
  ```

- Request parameter specification

  N/A

- Response example
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

- Response field specification
  N/A



#### confGet

- Interface description
   Kernel gets module configuration items 

- Request example

  ```json
  {
      "cmd":"confGet",
      "minVersion":1,
      "params":[]
  }
  ```

- Request parameter specification

  N/A

- Response example
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

- Response field specification
  N/A



#### confSet

- Interface description
  Kernel sets module configuration items

- Request example

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

- Request parameter specification

  N/A

- Response example
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

- Response field specification
  N/A



#### confReset

- Interface description
   Kernel restore template is configured as initial value. 

- Request example

  ```json
  {
      "cmd":"confReset",
      "minVersion":1,
      "params":[]
  }
  ```

- Request parameter specification

  N/A

- Response example
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

- Response field specification
  N/A





## 4. Event description




## 5. Protocol




## 6. Configuration



## 7. Java unique design

### Server
- Start server
    ```
    // Port is not necessary. If no port is specified, the system is allocated randomly
    BaseRpcServer server = new GrizzlyServer(RpcConstant.KERNEL_PORT);
    
    // Scan the package in which the CMD command is located
    server.scanPackage("io.nuls.rpc.mycmd");
    
    // Initialization
    server.init("moduleName", "dependsModule, type is List<String>");
    
    // start
    server.start();
    ```

-  Custom CMD 
   ```
   // extends BaseCmd
   public class SomeCmd extends BaseCmd
   
   //  Custom method to add annotation CmdInfo 
   @CmdInfo(cmd = "cmd1", version = 1.0, preCompatible = true)
   public Object methodName(List params) {
       System.out.println("I'm version 1");
       return success();
   }
   
   Note: rpc-tool automatically transfers the Object[] transferred from client to List<Object>. 
   ```

-  There must be and only one CMD class to implement the KernelCmd interface. 
    ```
    //  Implement interface 
    public class SomeCmd extends BaseCmd implements KernelCmd 
    
    /**
     *  Receive all module information from kernel 
     */
    public Object status(List params);
    
    /**
     * Shut down service: after the existing business is completed
     */
    public Object shutdown(List params);
    
    /**
     * Shut down service: close immediately, whether or not the business is completed.
     */
    public Object terminate(List params);
    
    /**
     *  Provide local configuration information 
     */
    public Object confGet(List params);
    
    /**
     *  Update local configuration information 
     */
    public Object confSet(List params);
    
    /**
     *  Reset local configuration information 
     */
    public Object confReset(List params);
    ```

### Client
-  Provide module information to kernel 
    ```
    RpcClient.versionToKernel();
    ```

- call rpc
    ```
    //  CMD corresponds to one interface. 
    String jsonStr = RpcClient.callSingleRpc("shutdown", params, minVersion);
    
    //  CMD corresponds to multiple interfaces. 
    String jsonStr = RpcClient.callMultiplyRpc("shutdown", params, minVersion);
    
    note:
    params is instance of Object[]     
    ```

###   Data exchange  
- Request 
    ```
    {
      "cmd": "shutdown",
      "minVersion": 1.0,  //根据自己需要传最低版本号
      "params": [],
    }
    ```

- Response(success and failure are the same)
    ```
    {
      "code":0,
      "msg": " This property is only available when failed",
      "version": "Actually called version",
      "result": {}
    }    
    ```
    

### Other
1.  BaseCmd has a default implementation of the status method, so if you don't have a specific requirement, you just need to do the following: 
    ```
    @Override
    @CmdInfo(cmd = RpcConstant.STATUS, version = 1.0, preCompatible = true)
    public Object status(List params){
        return super.status(params);
    }
    ```
    
2.  There are ways to return to success and failure in BaseCmd.   
    success:  

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

    fail:  
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



## 8. supplementary content

