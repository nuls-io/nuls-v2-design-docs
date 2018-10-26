# Chain management module design document

[TOC]

## 1. Overall description 

### 1.1 Summary

#### 1.1.1 Why need chain management module?

* NULS2.0 supports multi chain and cross chain transactions, so a module is needed to manage chain information.

* identification

  * Satellite chain: The core chain of NULS2.0
  * Friends chain: Any other chain that links only to the satellite chain, any cross chain transaction is maintained by the satellite chain.
  * Cross chain: the transaction between friend chain A and friend chain B is defined as "cross chain".


#### 1.1.2 What should chain management do?

All maintenance operations for chain (friend chain) should be in the chain management module.

* Register chain
* Destroy chain
* Query chain information
* Register asset
* Destroy asset



#### 1.1.3 Positioning of chain management in system

In the NULS 2.0 ecosystem chain system, "chain management" belongs to the satellite chain and friend chain has a different module, satellite chain provides all the interfaces, in the friend chain is only contains  query chain interface.


Chain Management is a general module in the system. It not only calls the interfaces of other modules, but also other modules call the interfaces of Chain Management.



### 1.2 Architecture diagram



## 2. Function design

### 2.1 Functional architecture diagram

### 2.2 Module service

#### 2.2.1 Chain registration and storage

* Function Description

  Provide an entry to do chain registration.

* Process description

  ![](image/chainModule/chainRegister.png)




### 2.3 Module internal function



## 3. Interface design

### 3.1 Module interface

#### Get chain information

- Description
  Get a chain detail information

- Request example
  ```json
  {
      "cmd":"chainInfo",
      "minVersion":"1.1",
      "params":[1234]
  }
  ```
- Request parameter specification

  | index | parameter | required | type    | description |
  | ----- | :-------- | :------- | :------ | ----------- |
  | 0     | chainId   | true     | integer | 链标识      |

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

- Response field specification
  | parameter          | type                     | description                                    |
  | ------------------ | ------------------------ | ---------------------------------------------- |
  | hash               | integer                  | hashcode of the chain                          |
  | chainId            | integer                  | chain id                                       |
  | name               | string                   | chain name                                     |
  | addressType        | integer                  | Address type of account created on the chain   |
  | assets             | jsonArray【AssetObject】 | Array data: asset object                       |
  | magicNumber        | integer                  | magic number                                   |
  | seeds              | jsonArray                | ip: ip address of seed<br />port: port of seed |
  | supportInflowAsset | boolean                  | Do support asset inflow?                       |

  Asset Object

  | parameter   | type        | description                                                  |
  | ----------- | ----------- | ------------------------------------------------------------ |
  | assetId     | integer     | asset id                                                     |
  | symbol      | string      | asset unit                                                   |
  | name        | string      | asset name                                                   |
  | depositNuls | integer     | Total nuls of deposit                                        |
  | initTotal   | big integer | Total initial assets                                         |
  | minUnit     | byte        | The smallest unit<br /> (representing the number of places after the decimal point) |
  | flag        | boolean     | Are assets available?                                        |



### 3.2 Functional interface

#### Chain registration

- Interface description
  Register a new chain

- Request example

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

- Request parameter specification

  | index | parameter               | required | type                     | description                                  |
  | ----- | :---------------------- | :------- | :----------------------- | -------------------------------------------- |
  | 0     | chainId                 | true     | integer                  | chain id                                     |
  | 1     | chainName               | true     | string                   | chain name                                   |
  | 2     | addressType             | true     | string                   | Address type of account created on the chain |
  | 3     | assets                  | true     | jsonArray【AssetObject】 | Array data: asset object                     |
  | 4     | minAvailableNodeNum     | true     | integer                  | Minimum number of available nodes            |
  | 5     | singleNodeConMinNodeNum | true     | integer                  | Minimum number of single node connections    |
  | 6     | txConfirmBlockNum       | true     | integer                  | Transaction confirmation block numbers       |
  | 7     | supportInflowAsset      | true     | boolean                  | Do we support asset inflow?                  |

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




#### Chain query

-  Interface description 
   Query chain list 

- Request example

  ```json
  {   
      "cmd": "chainList",
      "minVersion": 1.0,
      "params":[ 
     		 1,20
      ]
  }
  ```

- Request parameter specification

  | index | parameter  | required | type    | description |
  | ----- | :--------- | :------- | :------ | ----------- |
  | 0     | pageNumber | true     | integer | page number |
  | 1     | pageSize   | true     | integer | page size   |

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

- Response field specification
  Returns a list of chain.





#### Chain destroy

-  Interface description 
   The creator can destroy the chain he created. 

- Request example

  ```json
  {   
      "cmd": "chainDestroy",
      "minVersion": 1.0,
      "params":[1234]
  }
  ```

- Request parameter specification

  | index | parameter | required | type    | description |
  | ----- | :-------- | :------- | :------ | ----------- |
  | 0     | chainId   | true     | integer | 链标识      |

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





#### Asset Register

-  Interface description 
   The creator can create new asset types in the chain he created. 

- Request example

  ```json
  {   
      "cmd":"assetAdd",
      "minVersion":1.0
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

- Request parameter specification

  | index | parameter   | required | type               | description      |
  | ----- | :---------- | :------- | :----------------- | ---------------- |
  | 0     | chainId     | true     | integer            | 链标识           |
  | 1     | addressType | true     | integer            | 资产中的地址类型 |
  | 2     | asset       | true     | object【资产对象】 | 新增的资产       |

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





#### Asset destroy

-  Interface description 
   The creator can destroy the assets in the chain he created. 

- Request example

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

- Request parameter specification

  | index | parameter | required | type    | description |
  | ----- | :-------- | :------- | :------ | ----------- |
  | 0     | chainId   | true     | integer | 链标识      |
  | 1     | assetId   | true     | integer | 资产标识    |

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

[^说明]: 业务流程中尽量避免使用事件的方式通信

### 4.1 Released events

[^说明]: 这里说明事件的topic，事件的格式协议（精确到字节），事件的发生情景。

[参考<事件总线>]: ./



#### Explanation: when chain registration is successful, publish the event.   

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





 ####  Explanation: when a chain destroy is successful, publish the event.    

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





#### Explanation: when a asset register is successful, publish the event.   

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





#### Explanation: when a asset destroy is successful, publish the event.  

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






## 5. protocol

### 5.1  Network communication protocol 

#### N/A




### 5.2  Transaction protocol

##### Chain registration

Compared with general transactions, there are only differences between types and txData.

```
  type: n // transaction type   Uint16
  txData:{

      chainId:  //uint16 ,chain id   uint16

      chainName     //varString     less than 30 bytes


      Asset information list: asset ID, name, initial total, minimum unit assets available.
	  
	  assetid： uint32
	  
      symbol：//varString，     less than 30 bytes

      name：//varString，      less than 30 bytes

      initTotal：//varint         4 bytes

      minUnit://byte     1 byte
      
      flag： Determine whether assets can be circulated.     1 byte
	

      addressType：1，//byte : 1, NULS address structure, 2 other, 1 bytes.

      minAvailableNodeNum //uint16,    2 bytes

      singleNodeConMinNodeNum //uint16,   2 bytes

      txConfirmBlockNum //uint16,     2 bytes

      supportInflowAsset //boolean,     1bytes

  }

```





##### - Validator

```
Whether or not chainId legality is repeated chainId is generated by callers to represent a series of "meaningful" numbers.

The fields are not empty and the values are in the correct range.

Mortgage verification device

Other basic validator
```

##### - processor

```
Processor storage chain information

Storage asset information

After the n (operation parameter) block is confirmed, the magic parameter of the chain is monitored.
```



##### Chain destroy

Compared with general transactions, there are only differences between types and txData.

```
  type: n // transaction type, 2bytes
  txData:{
      chainId:  //uint16 ,chain id  2bytes
  }
```



##### - validator

```
 ChainId legitimacy
Whether the chain can be operated by that address, and the permission is verified.
```



##### - processor

```
 Stop all cross-chain transactions, unlock the mortgage, and logically delete the chain data from the block chain after the n block 
```



##### Asset register

Compared with general transactions, there are only differences between types and txData. 

```
 type: n // transaction type. 2bytes

  txData:{

    chainId:  //uint16 ,chain id    2bytes

    symbol：//varString    less than 30 bytes

    name：//varString   less than 30 bytes 

    initTotal：//     32 bytes

    minUnit://byte

  }
```



##### - validator

```
ChainId legitimacy

The fields are not empty and the values are in the correct range

Mortgage verification device

Other basic validator
```



##### - processor

```
 Storage asset information 
```



##### Asset destroy

Compared with general transactions, there are only differences between types and txData.

```
  type: n //transaction type
  txData:{
      chainId:  
      assetsId：
  }
```



##### - validator

```
Asset legality

Whether the asset can be operated by the address, and the permission is verified.

Is it the last asset of the chain?
```



##### - processor

```
If the asset is not transferred out of the issuance chain, the asset transaction is stopped immediately, the mortgage is unlocked, and the asset data is logically deleted from the block chain

Otherwise, stop the asset transaction after the n block, unlock the mortgage, and logically delete the asset data from the block chain
```






## 6.  Module configuration 

## 7.  Java unique design 

## 8.  Supplementary content 

