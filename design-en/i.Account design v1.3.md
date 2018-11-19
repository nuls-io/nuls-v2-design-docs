# Account module design document

[TOC]

## 1、General description

### 1.1 Module overview

#### 1.1.1 Why  have a Account Module

- Used to manage account generation, security and storage, and access to information.
- User saves account address, public and private key pair, and verification data signature.

#### 1.1.2 What to do with Account Module

The Account Module is a basic module that provides information about the various functions of the account.It mainly supports the functions of account generation, security and storage, and information acquisition. Other modules can use various functions of the account and obtain account information according to the interface provided by the account module. Users or other applications can be based on RPC. The interface makes the account more practical and personalized. The account is the basic module and the carrier of the user data.

- Account generation

  Create an account, import an account.

- Account security and storage

  Backup of account, setting account password, modifying account password, removing account.

- Access to account information

  Query singel account information, query multiple account information, query account address, query account alias.

- Other usability and personalization features

  Set account alias, set account remark, verify account encryption, sign, verify account address format, verify account password is correct, etc.

#### 1.1.3 Positioning of the account module in the system

![](./image/account-module/account-context.png)

The account is the underlying module that is dependent on the ledger, consensus, transaction, kernel, event bus, and community governance modules.

1、Account module depends on the ledger module

	The ledger module needs to handle local transactions and relies on account information.
	
	The account module needs to create an alias transaction, which requires the ledger module to pay the fee.
	
	Account balance inquiry, need to rely on ledger module.

2、Account module depends on kernel module

```
Report module information and share data operations.
```

3、Account module depends on network module

```
Receive and send data through a network module.
```

4、Account module depends on event bus module

```
Create an account, delete an account,change a password event,send a message through the event bus module.
The account module is not strongly dependent on the event bus module,because even if the event fails to send,it does not affect the normal business process.
```

5、Consensus module depends on account module

```
Consensus requires account information for packaging blocks.
```

6、Transaction management module  depends on account module

```
The transaction management module needs to verify the transaction,relying on the address function in the account to verify whether the address is legal.
```

7、Community Governance Module depends on account Module

```
Community governance requires account signature
```

### 1.2 Architecture diagram

![](./image/account-module/account-module.png)



1.API: Provides an interface layer externally, providing operations such as creating, backing up, and setting aliases for accounts.

2.Business logic layer: the function of defining accounts, account addresses, and aliases.

3.Data persistence layer: save account, alias data.

## 2、Function design

### 2.1 Functional architecture diagram

![](./image/account-module/account-functions.png)

### 2.2 Module service

#### 2.2.1 create account

- Function Description

  Create a user's account, including the account's address, public key, private key creation, account information saved to the database, and create an account to notify other nodes through the event.

- Process description

  ![](./image/account-module/account-create-address.png)

  Create Address: Create a satellite chain standard account address

  ```
  1.Generate a random public-private key pair
  2.Get chainId and account type
  3.Calculate hash160 based on public key
  4.Stitching byte arrays to form addresses
  4.1.If it is the NULS system address: address=type+hash160
  4.2、If non-NULS system address (bitcoin): address=original address length + original address
  5、Generate address string: address byte[] + check digit, then perform base58 calculation to generate string
  5.1.If it is the NULS system check digit: xor=XOR(addressType+pkh)
  5.2.If non-NULS system check digit: xor=XOR(length+address)
  6. base58 calculation generates the address string:
      NULS system address: Base58 (type+hash160+xor)+Hex(chainId)
      Non-NULS system address: Base58 (length+address+xor)+Hex(chainId)
  7.Encrypt the private key according to the password and delete the plaintext of the private key
  8.Store account information
  9.Add account to cache
  10.Send Create Account Event
  ```

- ac_createAccount interface

  - Interface Description

    This interface creates one or more accounts.

  - Request example

  ```
  {
      "cmd":"ac_createAccount",
      "minVersion":1.0,
      "params":[
          1234,
          10,
          "123456"
      ]
  }
  ```

  - Request parameter description

    | index | parameter | required | type    | description                                             |
    | ----- | --------- | -------- | ------- | ------------------------------------------------------- |
    | 0     | chainId   | true     | Short   | Chain ID, indicating which chain the account belongs to |
    | 1     | count     | false    | Integer | To create an account number, constraints:1-100.         |
    | 2     | password  | false    | String  | Account initial password, Can be empty                  |

  - Return example

  ```
  {
      "code": 0,
      "msg": "success",
      "version":1.0,
      "result": {
      	"list":["address","",""]
      }
  }
  ```

  - Return field description

  | parameter | type      | description          |
  | :-------- | :-------- | :------------------- |
  | code      | Integer   | Return result status |
  | msg       | String    | Failure message      |
  | result    | jsonObj   | Data return object   |
  | list      | jsonArray | Account address list |

- Dependent service

  Event Bus: Send Create Account Event

#### 2.2.2 Create an offline account

- Function Description

  Create a user's offline account, including the account's address, public key, private key creation, account information is not saved to the database

- Process description

  ![](./image/account-module/account-create-offline.png)

  Create Address: Create a satellite chain standard account address

  ```
  1.Generate a random public-private key pair
  2.Get chainId and account type
  3.Calculate hash160 based on public key
  4.Stitching byte arrays to form addresses
  4.1.If it is the NULS system address: address=type+hash160
  4.2、If non-NULS system address (bitcoin): address=original address length + original address
  5、Generate address string: address byte[] + check digit, then perform base58 calculation to generate string
  5.1.If it is the NULS system check digit: xor=XOR(addressType+pkh)
  5.2.If non-NULS system check digit: xor=XOR(length+address)
  6. base58 calculation generates the address string:
      NULS system address: Base58 (type+hash160+xor)+Hex(chainId)
      Non-NULS system address: Base58 (length+address+xor)+Hex(chainId)
  7.Encrypt the private key according to the password and delete the plaintext of the private key
  8.Return account information, not saved to the database
  ```

- ac_createOfflineAccount interface

  - Interface Description

    This interface creates one or more offline accounts.

  - Request example

    ```
    {
        "cmd":"ac_createOfflineAccount",
        "minVersion":1.0,
        "params":[
            1234,
            10,
            "123456"
        ]
    }
    ```

  - Request parameter description

    | index | parameter | required | type    | description                                             |
    | ----- | --------- | -------- | ------- | ------------------------------------------------------- |
    | 0     | chainId   | true     | Short   | Chain ID, indicating which chain the account belongs to |
    | 1     | count     | false    | Integer | To create an account number, constraints:1-100.         |
    | 2     | password  | false    | String  | Account initial password, Can be empty                  |

  - Return example

    ```
    {
        "code": 0,
        "msg": "success",
        "version":1.0,
        "result": {
        	"list":["address","",""]
        }
    }
    ```

  - Return field description

    | parameter | type      | description          |
    | :-------- | :-------- | :------------------- |
    | code      | Integer   | Return result status |
    | msg       | String    | Failure message      |
    | result    | jsonObj   | Data return object   |
    | list      | jsonArray | Account address list |


#### 2.2.3 Create a multi-sign account

  - Function Description

    Create a multi-signature account, including the address of the account, the creation of the script, and save the multi-signed account information to the database.

  - Process description

    ```
    1. Verify the signature public key list, and verify that the minimum number of signatures is correct.
    2. create a multi-signature script.
    3. create a multi-signature type account address according to the multi-sign script.
    4. save the multi-signature account.
    5. return multi-signed account information.
    ```

  - ac_createMultiSigAccount interface

    - Interface Description

      This interface creates a multi-signature account.

    - Request example

      ```
      {
        "cmd": "createMultiAccount",
        "minVersion":1.0,
        "params": [
              1234,
              ["pubKey1","pubKey2"],
              2
          ]
      }
      ```

    - Request parameter description

      | index | parameter | required | type      | description                                                  |
      | ----- | --------- | -------- | --------- | ------------------------------------------------------------ |
      | 0     | chainId   | true     | Short     | Chain ID, indicating which chain the account belongs to      |
      | 1     | pubKeys   | true     | jsonArray | Public key list that needs to be signed                      |
      | 2     | minSigns  | true     | String    | Minimum number of signatures, at least a few public key verifications are required |

    - Return example

      ```
      {
          "code": 0,
          "msg": "success",
          "version":1.0,
          "result": {
              "address":"",
              "minSigns":"",
              "pubKeys":[{
                  "pubKey":"",
                  "address":""
                  },{}
              ]
          }
      }
      ```

    - Return field description

      | parameter | type    | description                |
      | :-------- | :------ | :------------------------- |
      | code      | Integer | Return result status       |
      | msg       | String  | Failure message            |
      | result    | jsonObj | Data return object         |
      | address   | String  | Multi-sign account address |
      | minSigns  | Integer | Minimum signature number   |
      | pubKeys   | jsonObj | Public key list            |
      | --pubKey  | String  | Public key                 |
      | --address | String  | Account address            |

  - Dependent service

    no

#### 2.2.4 Remove account

- Function Description

  Remove the user's local account, including deleting the address of the local account, and notifying the other nodes through the event

- Process description

  ![](./image/account-module/account-remove-address.png)

  ```
  1. Verify that the account address format is correct.
  2. Verify that the account exists
  3. Verify that the account is encrypted. If the account is encrypted and the account is unlocked, you need to verify the password.
  3.1. Obtain an unencrypted private key according to the account's encrypted private key and password.
  3.2, get the public key according to the unencrypted private key
  3.3. Compare the decrypted public key with the queried public key
  4, delete the data
  4.1. Delete local account information
  4.2. Delete account cache information
  5. send remove account events
  ```

- ac_removeAccount interface

  - Interface Description

    This interface is used to remove the account.

  - Request example

    ```
    {
      "cmd": "ac_removeAccount",
      "minVersion":1.0,
      "params": [
            1234,
            "AAax8wqxALqjyhrL8Wv1tQiqswAshAnX",
            "123456"
        ]
    }
    ```

  - Request parameter description

    | index | parameter | required | type    | description               |
    | ----- | --------- | -------- | ------- | ------------------------- |
    | 0     | chainId   | true     | Short   | Chain ID                  |
    | 1     | address   | true     | Integer | Account address to delete |
    | 2     | password  | false    | String  | account password          |

  - Return example

    ```
    {
        "code": 0,
        "msg": "success",
        "version":1.0,
        "result": {
        	"value":true
        }
    }
    ```

  - Return field description

    | parameter | type    | description          |
    | :-------- | :------ | :------------------- |
    | code      | Integer | Return result status |
    | msg       | String  | Failure message      |
    | result    | jsonObj | Data return object   |
    | value     | boolean | 删除是否成功         |

- Dependent service

  Event Bus Module: Send Remove Account Event.


#### 2.2.5 Import account - private key

- Function Description

  Import accounts based on private keys, generate accounts based on private keys, and import account book data.

- Process description

  ![](./image/account-module/account-import-prikey.png)

  Import account information based on private key

  ```
  1. Generate a public-private key pair based on the private key.
  2. get the chainId and account type.
  3. calculate hash160 according to Public key.
  4.Stitching byte arrays to form addresses
  4.1.If it is the NULS system address: address=type+hash160
  4.2、If non-NULS system address (bitcoin): original address length + original address
  5、Generate address string: address byte[] + check digit, then perform base58 calculation to generate string
  5.1.If it is the NULS system check digit: xor=XOR(addressType+pkh)
  5.2.If non-NULS system check digit: xor=XOR(length+address)
  6. base58 calculation generates the address string:
      NULS system address: Base58 (type+hash160+xor)+Base58(chainId)
      Non-NULS system address: Base58 (length+address+xor)+Base58(chainId)
  7. Encrypt the private key according to the password and delete the plaintext of the private key.
  8. storage account information.
  9. add the account to the cache.
  10. If you send an import account event: If the account already exists, the new event will not be released, only the update will be made.
  ```

- ac_importAccountByPriKey interface

  - Interface Description

    The interface is imported into the account based on the account private key.

  - Request example

    ```
    {
      "cmd": "ac_importAccountByPriKey",
      "minVersion":1.0,
      "params": [
            1234,
            "00c22ad91a170fc49df53b79791f702879eb0604235787eee2c303463bf6e41111",
            "123456",
            true
        ]
    }
    ```

  - Request parameter description

    | index | parameter | required | type    | description                                  |
    | ----- | --------- | -------- | ------- | -------------------------------------------- |
    | 0     | chainId   | true     | Short   | Chain ID                                     |
    | 1     | priKey    | true     | String  | Account private key                          |
    | 2     | password  | false    | String  | Account password                             |
    | 3     | overwrite | true     | Boolean | Whether to overwrite when the account exists |

  - Return example

    ```
    {
        "code": 0,
        "msg": "success",
        "version":1.0,
        "result": {
        	"address":"NseMUi1q9TefkXUcaysAuvFjj4NbTEST"
        }
    }
    ```

  - Return field description

    | parameter | type    | description          |
    | :-------- | :------ | :------------------- |
    | code      | Integer | Return result status |
    | msg       | String  | Failure message      |
    | result    | jsonObj | Data return object   |
    | address   | String  | Account address      |

- Dependent service

  Event Bus Module: Send Import Account Event when the account does not exist.

  Account Book: Import Account leger(confirmed transaction)

#### 2.2.6 Import account-keystore

- Function Description

  According to the keystore import account, according to the keystore parsing and decrypting to get the private key and generate an account, and import the account book data

- Process description

    ![](./image/account-module/account-import-keystore.png)

    ```
    1. Verify that the keystore and password match.
    2. decrypt the private key in the keystore according to the password.
    3. Generate public and private based on the private key.
    4. get the chainId and account type.
    5. spliced byte array to form an address.
    6. generate an address string.
    7. Verify that the address string and the address in the keystore are consistent.
    8. storage account information.
    9. send import account event: the account already exists, do not post new events, only do the coverage update.
    ```

- ac_importAccountByKeystore interface

    - Interface Description

      This interface is used to import accounts keystore

    - Request example

      ```
      {
        "cmd": "ac_importAccountByKeystore",
        "minVersion":1.0,
        "params": [
              1234,
              "HEX",
              "123456",
              true
          ]
      }
      ```

    - Request parameter description

      | index | parameter | required | type    | description                                  |
      | ----- | --------- | -------- | ------- | -------------------------------------------- |
      | 0     | chainId   | true     | Short   | Chain ID                                     |
      | 1     | keyStore  | true     | String  | Imported keyStore hex code                   |
      | 2     | password  | false    | String  | Account password                             |
      | 3     | overwrite | true     | Boolean | Whether to overwrite when the account exists |

    - Return example

      ```
      {
          "code": 0,
          "msg": "success",
          "version":1.0,
          "result": {
          	"address":"ABCMUi1q9TefkXUcaysAuvFjj4NbTEST"
          }
      }
      ```

    - Return field description

      | parameter | type    | description          |
      | :-------- | :------ | :------------------- |
      | code      | Integer | Return result status |
      | msg       | String  | Failure message      |
      | result    | jsonObj | Data return object   |
      | address   | String  | Account address      |

- Dependent service

    Event Bus Module: Send Import Account Event when the account does not exist

    Account Book: Import Account leger(confirmed transaction)

#### 2.2.7 Import multi-signed accounts

- Function Description

  Import a multi-signed account related to the local address, including the address of the account, the creation of the script, and save the multi-signed account information to the database.

- Process description

  ```
  1. Verify that the multi-sign address, the signature public key list, and the minimum number of verified signatures are correct.
  2. create a multi-signature script.
  3. Create an account address of multiple signature types according to the multi-signed script.
  4. Determine whether the imported multi-sign address is the same as the address generated by the script. If it is not the same, it prompts an import error.
  5. save multi-signed account information, including: address, public key list, minimum number of verification signatures.
  6. return to multi-signal address.
  
  ```

- ac_importMultiSigAccount interface

  - Interface Description

    This interface is used to import multi-signature accounts.

  - Request example

    ```
    {
      "cmd": "ac_importMultiSigAccount",
      "minVersion":1.0,
      "params": [
            1234,
            "ABCMUi1q9TefkXUcaysAuvFjj4NbTEST",
            ["pubKey1","pubKey2"],
            2
        ]
    }
    ```

  - Request parameter description

    | index | parameter | required | type      | description                                                  |
    | ----- | --------- | -------- | --------- | ------------------------------------------------------------ |
    | 0     | chainId   | true     | Short     | Chain ID                                                     |
    | 1     | address   | true     | String    | Multi-sign account address                                   |
    | 2     | pubkeys   | true     | jsonArray | Public key list that needs to be signed                      |
    | 3     | minSigns  | true     | Integer   | Minimum signature number, At least a few Public key verifications are required |

  - Return example

    ```
    {
        "code": 0,
        "msg": "success",
        "version":1.0,
        "result": {
        	"address":"NseMUi1q9TefkXUcaysAuvFjj4NbTEST"
        }
    }
    ```

  - Return field description

    | parameter | type    | description          |
    | :-------- | :------ | :------------------- |
    | code      | Integer | Return result status |
    | msg       | String  | Failure message      |
    | result    | jsonObj | Data return object   |
    | address   | String  | Multi-sign address   |

  

#### 2.2.8 Export Account private key

- Function Description

  Export Account private key hex code

- Process description

    ![](./image/account-module/account-export-prikey.png)

    Export Account private key

    ```
    1. Verify that the account exists and verify that the password is correct.
    2. decrypt the private key, generate a Hex string
    ```

- ac_exportAccountPriKey interface

    - Interface Description

      This interface is used to export the Account private key.

    - Request example

      ```
      {
        "cmd": "ac_importMultiSigAccount",
        "minVersion":1.0,
        "params": [
              1234,
              "ABCMUi1q9TefkXUcaysAuvFjj4NbTEST",
              "123456"
          ]
      }
      ```

    - Request parameter description

      | index | parameter | required | type   | description      |
      | ----- | --------- | -------- | ------ | ---------------- |
      | 0     | chainId   | true     | Short  | Chain ID         |
      | 1     | address   | true     | String | Account address  |
      | 2     | password  | true     | String | Account password |

    - Return example

      ```
      {
          "code": 0,
          "msg": "success",
          "version":1.0,
          "result": {
          	"address":"NseMUi1q9TefkXUcaysAuvFjj4NbTEST",
          	"priKey":"1cb336b834494fb7eef070cf9c3e60a5a49e762ca1f81cb2592593047235f308"
          }
      }
      ```

    - Return field description

      | parameter | type    | description          |
      | :-------- | :------ | :------------------- |
      | code      | Integer | Return result status |
      | msg       | String  | Failure message      |
      | result    | jsonObj | Data return object   |
      | address   | String  | Account address      |
      | priKey    | String  | Private key hex      |

#### 2.2.9 Export account KeyStore

- Function Description

  Export account keystore

- Process description

    ![](./image/account-module/account-export-keystore.png)

    Export account keystore
    ```
    1. Verify that the account exists and verify that the password is correct.
    2. generate a keystore file.
    ```
- ac_exportAccountKeyStore interface

    - Interface Description

      This interface is used to export the account keystore.

    - Request example

      ```
      {
        "cmd": "ac_exportAccountKeyStore",
        "minVersion":1.0,
        "params": [
              1234,
              "ABCMUi1q9TefkXUcaysAuvFjj4NbTEST",
              "123456"
          ]
      }
      ```

    - Request parameter description

      | index | parameter | required | type   | description      |
      | ----- | --------- | -------- | ------ | ---------------- |
      | 0     | chainId   | true     | Short  | Chain ID         |
      | 1     | address   | true     | String | Account address  |
      | 2     | password  | true     | String | Account password |

    - Return example

          {
              "code": 0,
              "msg": "success",
              "version":1.0,
              "result": {
              	"address":"NseMUi1q9TefkXUcaysAuvFjj4NbTEST",
                  "encryptedPrivateKey":"",
                  "pubKey":"1cb336b834494fb7eef070cf9c3e60a5a49e762ca1f81cb2592593047235f308"
              }
          }

    - Return field description

      | parameter           | type    | description           |
      | :------------------ | :------ | :-------------------- |
      | code                | Integer | Return result status  |
      | msg                 | String  | Failure message       |
      | result              | jsonObj | Data return object    |
      | address             | String  | Account address       |
      | encryptedPrivateKey | String  | Encrypted private key |
      | pubKey              | String  | Public key hex        |

#### 2.2.10 Query all accounts

- Function Description

  Query all accounts

- Process description 

    ```
    1. query all account information
    ```
- ac_getAccountList interface

    - Interface Description

      This interface is used to query all accounts.

    - Request example

      ```
      {
          "cmd":"ac_getAccountList",
          "minVersion":1.0,
          "params":[
              1234
          ]
      }
      ```

    - Request parameter description

      | index | parameter | required | type  | description |
      | ----- | --------- | -------- | ----- | ----------- |
      | 0     | chainId   | true     | Short | Chain ID    |

    - Return example

          {
              "code": 0,
              "msg": "success",
              "version":1.0,
              "result": {
              	"list":[{
                   	"address":"",
                      "alias":"",
                      "pubkeyHex":"",
                      "encryptedPrikeyHex":""
                      },{}]
              }
          }

    - Return field description

      | parameter   | type | description |
      | :------------------ | :------- | :----------- |
      | code                | Integer  | Return result status |
      | msg                 | String   | Failure message |
      | result              | jsonObj  | Data return object |
      | list               | List     | Account list collection |
      | address            | String   | Account address |
      | alias              | String   | alias            |
      | pubkeyHex          | String   | Public key hex code |
      | encryptedPrikeyHex | String   | Encrypted private key hex code |
#### 2.2.11 Get an account based on address

- Function Description

  Get an account based on address

- Process description

  ```
  1. Verify that the address exists.
  2. get an account based on the address
  ```

- ac_getAccountByAddress interface

    - Interface Description

      This interface is used to get an account based on the address.

    - Request example

      ```
      {
        "cmd": "ac_getAccountByAddress",
        "minVersion":1.0,
        "params": [
              1234,
              "ABCMUi1q9TefkXUcaysAuvFjj4NbTEST"
          ]
      }
      ```

    - Request parameter description

      | index | parameter | required | type   | description     |
      | ----- | --------- | -------- | ------ | --------------- |
      | 0     | chainId   | true     | Short  | Chain ID        |
      | 1     | address   | true     | String | Account address |

    - Return example 

      ```
      {
          "code": 0,
          "msg": "success",
          "version":1.0,
          "result": {
            "address":"",
            "alias":"",
            "pubkeyHex":"",
            "encryptedPrikeyHex":""
          }
      }
      ```

    - Return field description

      | parameter   | type | description |
      | :------------------ | :------- | :----------- |
      | code                | Integer  | Return result status |
      | msg                 | String   | Failure message |
      | result              | jsonObj  | Data return object |
      | address            | String   | Account address |
      | alias              | String   | alias            |
      | pubkeyHex          | String   | Public key hex code |
      | encryptedPrikeyHex | String   | Encrypted private key hex code |

#### 2.2.12 Query the list of Account addresses

- Function Description

  Query the list of Account addresses

- Process description

    ```
    1. Check whether the paging parameter is legal. The number of pages and the size of the page cannot be less than 0, and must be an integer.
    2. Query all accounts.
    3. Filter accounts that meet the paging conditions.
    4. Return only the address list of the account.
    ```
- ac_getAddressList interface

    - Interface Description

      This interface is used to query the list of Account addresses.

    - Request example

      ```
      {
        "cmd": "ac_getAddressList",
        "minVersion":1.0,
        "params": [
              1234,
              0,
              10
          ]
      }
      ```

    - Request parameter description

      | index | parameter  | required | type    | description |
      | ----- | ---------- | -------- | ------- | ----------- |
      | 0     | chainId    | true     | Short   | Chain ID    |
      | 1     | pageNumber | true     | Integer | pageNumber  |
      | 2     | pageSize   | true     | Integer | pageSize    |

    - Return example

          {
              "code": 0,
              "msg": "success",
              "version":1.0,
              "result": {
              	"list":["","",""]
              }
          }

    - Return field description

      | parameter   | type | description |
      | :------------------ | :------- | :----------- |
      | code                | Integer  | Return result status |
      | msg                 | String   | Failure message |
      | result              | jsonObj  | Data return object |
      | list     | jsonArray | Address list collection |

#### 2.2.13 Get an address based on alias

- Function Description

  Get an address based on alias

- Process description

    ```
    1. query whether the existence of alias
    2. return the Account address used to set the alias, and use Base58 to encode the Account address
    ```
- ac_getAddressByAlias interface

    - Interface Description

      This interface is used to get the address based on alias.

    - Request example

      ```
      {
        "cmd": "ac_getAddressList",
        "minVersion":1.0,
        "params": [
              1234,
              "abc"
          ]
      }
      ```

    - Request parameter description

      | index | parameter | required | type   | description |
      | ----- | --------- | -------- | ------ | ----------- |
      | 0     | chainId   | true     | Short  | Chain ID    |
      | 1     | alias     | true     | String | alias       |

    - Return example

      ```
      {
          "code": 0,
          "msg": "success",
          "version":1.0,
          "result": {
          	"address":"NseMUi1q9TefkXUcaysAuvFjj4NbTEST",
          }
      }
      ```

    - Return field description

      | parameter | type | description |
      | :------- | :------- | :-------------- |
      | code     | Integer  | Return result status |
      | msg      | String   | Failure message |
      | result   | jsonObj  | Data return object |
      | address  | String   | Account address, Base58 code |
#### 2.2.14 Query Account private key

- Function Description

  Query Account private key

- Process description

    ```
    1. Check whether the address is correct, use Base58 decoding, and check the Chain ID, address type, and check digit respectively.
    2. Verify that the account exists.
    3. If the account is over-densified (with password) and is not unlocked, decrypt it by AES and verify that the password is correct, and obtain the unencrypted private key.
    4. use hexadecimal encoding, and return to Account private key.
    ```
- ac_getPriKeyByAddress interface

    - Interface Description

      This interface is used to query the Account private key.

    - Request example

      ```
      {
        "cmd": "ac_getPriKeyByAddress",
        "minVersion":1.0,
        "params": [
              1234,
              "NseMUi1q9TefkXUcaysAuvFjj4NbTEST",
              "123456"
          ]
      }
      ```

    - Request parameter description

      | index | parameter | required | type   | description      |
      | ----- | --------- | -------- | ------ | ---------------- |
      | 0     | chainId   | true     | Short  | Chain ID         |
      | 1     | address   | true     | String | address          |
      | 2     | password  | false    | String | Account password |

    - Return example

          {
              "code": 0,
              "msg": "success",
              "version":1.0,
              "result": {
                 "priKey":"1cb336b834494fb7eef070cf9c3e60a5a49e762ca1f81cb2592593047235f308"
              }
          }

    - Return field description

      | parameter | type | description |
      | :------- | :------- | :----------- |
      | code     | Integer  | Return result status |
      | msg      | String   | Failure message |
      | result   | jsonObj  | Data return object |
      | priKey   | String   | Private key hex |
#### 2.2.15Query all Account private keys

- Function Description

  Query all Account private keys

- Process description

    ```
    1. Verify that the password format is correct and the password can be empty.
    2. get all local accounts.
    3. The encryption information of the local account must be the same. If the parameter password is not empty, the passwords of all accounts must be the same. If the parameter password is empty, all accounts cannot be set. Otherwise, the error is displayed.
    4. If the account is encrypted, the unencrypted private key is reversed by the password, otherwise the private key is obtained without encryption.
    5. Add all private keys to the collection and return.
    ```
- ac_getAllPriKey interface

    - Interface Description

      This interface is used to query all Account private keys.

    - Request example

      ```
      {
        "cmd": "ac_getAllPriKey",
        "minVersion":1.0,
        "params": [
              1234,
              "123456"
          ]
      }
      ```

    - Request parameter description

      | index | parameter | required | type   | description      |
      | ----- | --------- | -------- | ------ | ---------------- |
      | 0     | chainId   | true     | Short  | Chain ID         |
      | 2     | password  | false    | String | Account password |

    - Return example

          {
              "code": 0,
              "msg": "success",
              "version":1.0,
              "result": {
                "list":["",""]
              }
          }

    - Return field description

      | parameter   | type | description |
      | :------------------ | :------- | :----------- |
      | code                | Integer  | Return result status |
      | msg                 | String   | Failure message |
      | result              | jsonObj  | Data return object |
      | list     | String   | Private key collection |
#### 2.2.16 set password

- Function Description

  set password

- Process description

  ```
  1. Verify that the account exists.
  2. Verify that the password has been set.
  3. set Account password.
  ```

- ac_setPassword interface

    - Interface Description

      This interface is used to set the Account password.

    - Request example

      ```
      {
        "cmd": "ac_setPassword",
        "minVersion":1.0,
        "params": [
              1234,
              "NseMUi1q9TefkXUcaysAuvFjj4NbTEST",
              "123456"
          ]
      }
      ```

    - Request parameter description

      | index | parameter | required | type   | description      |
      | ----- | --------- | -------- | ------ | ---------------- |
      | 0     | chainId   | true     | Short  | Chain ID         |
      | 1     | address   | true     | String | Account address  |
      | 2     | password  | true     | String | Account password |

    - Return example

          {
              "code": 0,
              "msg": "success",
              "version":1.0,
              "result": {
              	"value":true
              }
          }

    - Return field description

      | parameter   | type | description |
      | :------------------ | :------- | :----------- |
      | code                | Integer  | Return result status |
      | msg                 | String   | Failure message |
      | result              | jsonObj  | Data return object |
      | value               | boolean  | Data return object, password setting is successful |
#### 2.2.17 Set offline Account password

- Function Description

  Set offline Account password

- Process description

  ```
  1. Verify that the address is correct.
  2. Verify that the private key is correct.
  3. create according to the private key.
  4. set offline Account password.
  ```

- ac_setOfflineAccountPassword interface

    - Interface Description

      This interface is used to set the offline Account password.

    - Request example

      ```
      {
        "cmd": "ac_setOfflineAccountPassword",
        "minVersion":1.0,
        "params": [
              1234,
              "NseMUi1q9TefkXUcaysAuvFjj4NbTEST",
              "00c22ad91a170fc49df53b79791f702879eb0604235787eee2c303463bf6e41111",
              "123456"
          ]
      }
      ```

    - Request parameter description

      | index | parameter | required | type   | description         |
      | ----- | --------- | -------- | ------ | ------------------- |
      | 0     | chainId   | true     | Short  | Chain ID            |
      | 1     | address   | true     | String | Account address     |
      | 2     | priKey    | true     | String | Account private key |
      | 3     | password  | true     | String | Account password    |

    - Return example

      ```
      {
          "code": 0,
          "msg": "success",
          "version":1.0,
          "result": {
              "encryptedPriKey":""
          }
      }
      ```

    - Return field description

      | parameter | type | description |
      | :-------------- | :------- | :----------- |
      | code            | Integer  | Return result status |
      | msg             | String   | Failure message |
      | result          | jsonObj  | Data return object |
      | encryptedPriKey | String   | Encrypted private key |
#### 2.2.18 change Password

- Request Body

- Function Description

  change Password

- Process description

  ```
  1. verify the correctness of the old password.
  2. Update the private key ciphertext.
  3. send a password change event.
  ```

- ac_updatePassword interface

    - Interface Description

      This interface is used to modify the Account password.

    - Request example

      ```
      {
        "cmd": "ac_updatePassword",
        "minVersion":1.0,
        "params": [
              1234,
              "NseMUi1q9TefkXUcaysAuvFjj4NbTEST",
              "123456",
              "111111"
          ]
      }
      ```

    - Request parameter description

      | index | parameter   | required | type   | description          |
      | ----- | ----------- | -------- | ------ | -------------------- |
      | 0     | chainId     | true     | Short  | Chain ID             |
      | 1     | address     | true     | String | Account address      |
      | 2     | password    | true     | String | Account password     |
      | 3     | newPassword | true     | String | Account new password |

    - Return example

          {
              "code": 0,
              "msg": "success",
              "version":1.0,
              "result": {
                  "value":true
              }
          }

    - Return field description

      | parameter   | type | description |
      | :------------------ | :------- | :----------- |
      | code                | Integer  | Return result status |
      | msg                 | String   | Failure message |
      | result              | jsonObj  | Data return object |
      | value               | boolean  | Whether the password modification is successful |
#### 2.2.19 Modify offline Account password

- Function Description

  Modify offline Account password

- Process description

  ```
  1. Verify that the address is correct.
  2. Generate an offline account based on address, private key, and new password.
  ```

- ac_updateOfflineAccountPassword interface

    - Interface Description

      This interface is used to modify the offline Account password.

    - Request example

      ```
      {
        "cmd": "ac_updateOfflineAccountPassword",
        "minVersion":1.0,
        "params": [
              1234,
              "NseMUi1q9TefkXUcaysAuvFjj4NbTEST",
              "00c22ad91a170fc49df53b79791f702879eb0604235787eee2c303463bf6e41111",
              "123456",
              "111111"
          ]
      }
      
      ```

    - Request parameter description

      | index | parameter   | required | type   | description          |
      | ----- | ----------- | -------- | ------ | -------------------- |
      | 0     | chainId     | true     | Short  | Chain ID             |
      | 1     | address     | true     | String | Account address      |
      | 2     | priKey      | true     | String | Private key          |
      | 3     | password    | true     | String | Account password     |
      | 4     | newPassword | true     | String | Account new password |

    - Return example

          {
              "code": 0,
              "msg": "success",
              "version":1.0,
              "result": {
                  "encryptedPriKey":""
              }
          }

    - Return field description

      | parameter | type | description |
      | :-------------- | :------- | :----------- |
      | code            | Integer  | Return result status |
      | msg             | String   | Failure message |
      | result          | jsonObj  | Data return object |
      | encryptedPriKey | String   | Encrypted private key |
#### 2.2.20 verify password

- Function Description

  verify password

- Process description

  ```
  1. Verify that the password is correct.
  2. return verification results.
  ```

- ac_validationPassword interface

    - Interface Description

      This interface is used to verify the password.

    - Request example

      ```
      {
        "cmd": "ac_validationPassword",
        "minVersion":1.0,
        "params": [
              1234,
              "NseMUi1q9TefkXUcaysAuvFjj4NbTEST",
              "123456"
          ]
      }
      ```

    - Request parameter description

      | index | parameter | required | type   | description      |
      | ----- | --------- | -------- | ------ | ---------------- |
      | 0     | chainId   | true     | Short  | Chain ID         |
      | 1     | address   | true     | String | Account address  |
      | 2     | password  | true     | String | Account password |

    - Return example

          {
              "code": 0,
              "msg": "success",
              "version":1.0,
              "result": {
                  "value":true
              }
          }

    - Return field description

      | parameter | type | description |
      | :------- | :------- | :----------- |
      | code     | Integer  | Return result status |
      | msg      | String   | Failure message |
      | result   | jsonObj  | Data return object |
      | value    | boolean  | Is the password correct |
#### 2.2.21 Verify that the account is encrypted

- Function Description

  Verify that the account is encrypted

- Process description

  ```
  1. Verify that the account exists.
  2. Verify that the account is encrypted.
  3. return verification results.
  ```

- ac_isEncrypted interface

    - Interface Description

      This interface is used to verify that the account is encrypted.

    - Request example

      ```
      {
        "cmd": "ac_isEncrypted",
        "minVersion":1.0,
        "params": [
              1234,
              "NseMUi1q9TefkXUcaysAuvFjj4NbTEST"
          ]
      }
      ```

    - Request parameter description

      | index | parameter | required | type   | description     |
      | ----- | --------- | -------- | ------ | --------------- |
      | 0     | chainId   | true     | Short  | Chain ID        |
      | 1     | address   | true     | String | Account address |

    - Return example

      ```
      {
          "code": 0,
          "msg": "success",
          "version":1.0,
          "result": {
          	"value":true
          }
      }
      ```

    - Return field description

      | parameter | type | description |
      | :------- | :------- | :----------- |
      | code     | Integer  | Return result status |
      | msg      | String   | Failure message |
      | result   | jsonObj  | Data return object |
      | value    | boolean  | Whether the account is encrypted |
#### 2.2.22 Set up an account alias

- Function Description

  Set up an account alias

- Process description

    ```
    1. Verify that alias is legal
    2. generate settings alias transaction
    3. call ledger to fill transaction fee information
    4. broadcast transactions
    5. After the transaction is confirmed, the alias will be saved to the database, and address and alias will be stored as keys respectively. That is, the alias data will store two data, mainly for the convenience of querying according to address and alias.
    ```

- ac_setAlias interface

    - Interface Description

      This interface is used to set the account alias.

    - Request example

      ```
      {
        "cmd": "ac_setAlias",
        "minVersion":1.0,
        "params": [
              1234,
              "NseMUi1q9TefkXUcaysAuvFjj4NbTEST",
              "123456",
              "abc"
          ]
      }
      ```

    - Request parameter description

      | index | parameter | required | type   | description      |
      | ----- | --------- | -------- | ------ | ---------------- |
      | 0     | chainId   | true     | Short  | Chain ID         |
      | 1     | address   | true     | String | Account address  |
      | 2     | password  | false    | String | Account password |
      | 3     | alias     | true     | String | alias            |

    - Return example

          {
              "code": 0,
              "msg": "success",
              "version":1.0,
              "result": {
                "txHash":"1cb336b834494fb7eef070cf9c3e60a5a49e762ca1f81cb2592593047235f308"
              }
          }

    - Return field description

      | parameter | type | description |
      | :------- | :------- | :----------- |
      | code     | Integer  | Return result status |
      | msg      | String   | Failure message |
      | result   | jsonObj  | Data return object |
      | txHash   | String   | Alias transaction hash |

- Dependent service

    Account module: setting an alias requires a fee
#### 2.2.23 Get set alias fee

- Function Description

  Get set alias fee

- Process description

  ```
  1. Verify that the account exists and verify that the alias is correct.
  2. calculate the fees required for the alias setting.
  ```

- ac_getAliasFee interface

    - Interface Description

      This interface is used to get the set alias fee.

    - Request example

      ```
      {
        "cmd": "ac_getAliasFee",
        "minVersion":1.0,
        "params": [
              1234,
              "NseMUi1q9TefkXUcaysAuvFjj4NbTEST",
              "abc"
          ]
      }
      ```

    - Request parameter description

      | index | parameter | required | type   | description     |
      | ----- | --------- | -------- | ------ | --------------- |
      | 0     | chainId   | true     | Short  | Chain ID        |
      | 1     | address   | true     | String | Account address |
      | 2     | alias     | true     | String | alias           |

    - Return example

          {
              "code": 0,
              "msg": "success",
              "version":1.0,
              "result": {
                  "fee":"100",
                  "maxAmount":"10000"
              }
          }

    - Return field description

      | parameter   | type | description |
      | :------------------ | :------- | :----------- |
      | code                | Integer  | Return result status |
      | msg                 | String   | Failure message |
      | result              | jsonObj  | Data return object |
      | fee                 | String   | Alias transaction fee |
      | maxAmount           | String   | Maximum transaction fee |
#### 2.3.24 Query alias based on address

- Function Description

  Query alias based on address

- Process description

  ```
  1. Verify that the account exists.
  2. query the account corresponding alias from the database
  ```

- ac_getAliasByAddress interface

    - Interface Description

      This interface is used to query aliases based on the address.

    - Request example

      ```
      {
        "cmd": "ac_getAliasByAddress",
        "minVersion":1.0,
        "params": [
              1234,
              "NseMUi1q9TefkXUcaysAuvFjj4NbTEST"
          ]
      }
      ```

    - Request parameter description

      | index | parameter | required | type   | description     |
      | ----- | --------- | -------- | ------ | --------------- |
      | 0     | chainId   | true     | Short  | Chain ID        |
      | 1     | address   | true     | String | Account address |

    - Return example

          {
              "code": 0,
              "msg": "success",
              "version":1.0,
              "result": {
                  "alias":""
              }
          }

    - Return field description

      | parameter | type | description |
      | :------- | :------- | :----------- |
      | code     | Integer  | Return result status |
      | msg      | String   | Failure message |
      | result   | jsonObj  | Data return object |
      | alias    | String   | Account alias |
#### 2.2.25 Verify that alias is available

- Function Description

  Verify that alias is available

- Process description

  ```
  1. Query whether the alias already exists. If it exists, it is not available. Otherwise, it is available.
  ```

- ac_isAliasUsable interface

    - Interface Description

      This interface is used to verify if alias is available.

    - Request example

      ```
      {
        "cmd": "ac_isAliasUsable",
        "minVersion":1.0,
        "params": [
              1234,
              "abc"
          ]
      }
      ```

    - Request parameter description

      | index | parameter | required | type   | description   |
      | ----- | --------- | -------- | ------ | ------------- |
      | 0     | chainId   | true     | Short  | Chain ID      |
      | 1     | alias     | true     | String | Account alias |

    - Return example

          {
              "code": 0,
              "msg": "success",
              "version":1.0,
              "result": {
                  "value":true
              }
          }

    - Return field description

      | parameter   | type | description |
      | :------------------ | :------- | :----------- |
      | code                | Integer  | Return result status |
      | msg                 | String   | Failure message |
      | result              | jsonObj  | Data return object |
      | value               | boolean  | Is alias available |
#### 2.2.26 Set account remark

- Function Description

  Set account remark

- Process description

  ```
  1. Verify that the account exists.
  2. modify the remarks information and save
  ```

- ac_setRemark interface

    - Interface Description

      This interface is used to set account remark.

    - Request example

      ```
      {
        "cmd": "ac_setRemark",
        "minVersion":1.0,
        "params": [
              1234,
              "NseMUi1q9TefkXUcaysAuvFjj4NbTEST"
              "remark1"
          ]
      }
      ```

    - Request parameter description

      | index | parameter | required | type   | description     |
      | ----- | --------- | -------- | ------ | --------------- |
      | 0     | chainId   | true     | Short  | Chain ID        |
      | 1     | address   | true     | String | Account address |
      | 2     | remark    | true     | String | Account remark  |

    - Return example

          {
              "code": 0,
              "msg": "success",
              "version":1.0,
              "result": {
                  "value":true
              }
          }

    - Return field description

      | parameter | type | description |
      | :------- | :------- | :----------- |
      | code     | Integer  | Return result status |
      | msg      | String   | Failure message |
      | result   | jsonObj  | Data return object |
      | value    | boolean  | Whether the setting is successful |
#### 2.2.27 Set up multi-sign account alias

- Function Description

  Set up multi-sign account alias

- Process description

  ```
  1. Verify that the Account address, alias, Account password, and signature address parameters are legal.
  2. Query whether the Account address and the signature address exist.
  3. generate a set of multi-sign account alias trading.
  4, call ledger to fill transaction fee information.
  5. Sign the transaction using a signed account.
  6. Save the unconfirmed transaction to the local account when the signed number is equal to the minimum number of signatures.
  7. Broadcast the transaction.
  8. return the transaction hash.
  ```

- ac_setMultiSigAlias interface

  - Interface Description

      This interface is used to set the multi-sign account alias.

  - Request example

      ```
      {
        "cmd": "ac_setMultiSigAlias",
        "minVersion":1.0,
        "params": [
              1234,
              "NseMUi1q9TefkXUcaysAuvFjj4NbTEST",
              "DCQMUi1q9TefkXUcaysAuvFjj4NbTEST",
              "123456",
              "abc"
          ]
      }
      ```

  - Request parameter description

    | index | parameter   | required | type   | description                |
    | ----- | ----------- | -------- | ------ | -------------------------- |
    | 0     | chainId     | true     | Short  | Chain ID                   |
    | 1     | address     | true     | String | Multi-sign account address |
    | 2     | signAddress | true     | String | Signature address          |
    | 3     | password    | false    | String | Account password           |
    | 4     | alias       | true     | String | alias                      |

  - Return example

    ```
    {
        "code": 0,
        "msg": "success",
        "version":1.0,
        "result": {
           "txHash":"1cb336b834494fb7eef070cf9c3e60a5a49e762ca1f81cb2592593047235f308"
        }
    }
    
    ```

  - Return field description

    | parameter | type    | description                           |
    | :-------- | :------ | :------------------------------------ |
    | code      | Integer | Return result status                  |
    | msg       | String  | Failure message                       |
    | result    | jsonObj | Data return object                    |
    | txHash    | String  | Multi-signalias alias transactionhash |

- Dependent service

  Account module: setting a multi-signal alias requires a fee.

#### 2.2.28 Remove multi-signed account

- Function Description

  Remove multi-signed account

- Process description

  ```
  1. Verify that the Account address is correct.
  2. delete the multi-signed account in the database.
  3. return to delete is successful.
  ```

- ac_removeMultiSigAccount interface

  - Interface Description

      This interface is used to remove multi-signed accounts.

  - Request example

      ```
      {
        "cmd": "ac_setMutilSigAlias",
        "minVersion":1.0,
        "params": [
              1234,
              "DCQMUi1q9TefkXUcaysAuvFjj4NbTEST",
          ]
      }
      ```

  - Request parameter description

    | index | parameter | required | type   | description                |
    | ----- | --------- | -------- | ------ | -------------------------- |
    | 0     | chainId   | true     | Short  | Chain ID                   |
    | 1     | address   | true     | String | Multi-sign account address |

  - Return example

    ```
    {
        "code": 0,
        "msg": "success",
        "version":1.0,
        "result": {
           "value":true
        }
    }
    
    ```

  - Return field description

    | parameter | type    | description                        |
    | :-------- | :------ | :--------------------------------- |
    | code      | Integer | Return result status               |
    | msg       | String  | Failure message                    |
    | result    | jsonObj | Data return object                 |
    | value     | boolean | Whether the removal was successful |


#### 2.2.29 Account all transaction verification

- Function Description

  account module all transaction verification interface, currently only alias transaction

- Process description

  ```
  1. Is the transaction list empty
  2. loop through all transaction lists, processing for alias transactions
  3. Check if the same alias is set in the current transaction list.
  4. Check whether there is an account duplicate setting alias in the current transaction list.
  5. If there is no conflict in the transaction list, the verification is passed.
  ```

- ac_accountTxValidate interface

  - Interface Description

    This interface is used to batch verify all transactions in the account module.

  - Request example

    ```
    {
      "cmd": "ac_accountTxValidate",
      "minVersion":1.0,
      "params": [chianId, ["txHex","txHex","txHex", ...]]
    }
    
    ```

  - Request parameter description

    | index | parameter | required | type  | description                                |
    | ----- | --------- | -------- | ----- | ------------------------------------------ |
    | 0     | chainId   | true     | Short | Chain ID                                   |
    | 1     | txHex     | true     | array | Alias transaction serialization data array |

    txHex description

    ```
    {
        "type":3,
        "time":"12546545596",
        "scriptSig":"",
        "hash":"",
        "coinData":
        {
            "froms":
            [{
                "address":"Nse8m2Te1UNGPhD1tjZ3A4GDW3dCJxqE",
                "amount":10000,
                "nonce":"123",
            }],
            "to":
            [{
                "address":"Nse8m2Te1UNGPhD1tjZ3A4GDW3dCJxqE",
                "amount":1
            }]
        },
        "txData":
        {
            "chainId":"12345",
            "address":"Nse8m2Te1UNGPhD1tjZ3A4GDW3dCJxqE",
            "alias":"lucas"
        }
    }
    ```

  - Return example

    ```
    {
        "code": 0,
        "msg": "success",
        "version":1.0,
        "result": {
           "list":["txHex", "txHex", "txHex", ...]
        }
    }
    ```

  - Return field description

    | parameter | type      | description                                  |
    | :-------- | :-------- | :------------------------------------------- |
    | code      | Integer   | Return result status                         |
    | msg       | String    | Failure message                              |
    | result    | jsonObj   | Data return object                           |
    | list      | jsonArray | Illegal transaction serialization data array |


#### 2.2.30 Alias transaction verification

- Function Description

  alias transaction verification interface

- Process description

  ```
  1. deserialize txHex alias transaction data
  2. verify the alias format
  3. Verify that the alias is already occupied.
  4. Verify that the account has an alias set.
  5. verify the coinData input and output
  6. verify the script signature format
  7. Verify that the signature contains the address of the alias. 
  If it is not included, it is a malicious foul. 
  Otherwise, the verification is passed.
  ```

- ac_aliasTxValidate interface

  - Interface Description

    This interface is used for a single alias transaction.

  - Request example

    ```
    {
      "cmd": "ac_aliasTxValidate",
      "minVersion":1.0,
      "params": [chainId,"txHex"]
    }
    
    ```

  - Request parameter description

    | index | parameter | required | type   | description                          |
    | ----- | --------- | -------- | ------ | ------------------------------------ |
    | 0     | chainId   | true     | Short  | Chain ID                             |
    | 1     | txHex     | true     | String | alias transaction serialization data |

    txHex description

    ```
    {
        "type":3,
        "time":"12546545596",
        "scriptSig":"",
        "hash":"",
        "coinData":
        {
            "froms":
            [{
                "address":"Nse8m2Te1UNGPhD1tjZ3A4GDW3dCJxqE",
                "amount":10000,
                "nonce":"123",
            }],
            "to":
            [{
                "address":"Nse8m2Te1UNGPhD1tjZ3A4GDW3dCJxqE",
                "amount":1
            }]
        },
        "txData":
        {
            "chainId":"12345",
            "address":"Nse8m2Te1UNGPhD1tjZ3A4GDW3dCJxqE",
            "alias":"lucas"
        }
    }
    ```

  - Return example

    ```
    {
        "code": 0,
        "msg": "success",
        "version":1.0,
        "result": {
           "value":true
        }
    }
    ```

  - Return field description

    | parameter | type    | description                            |
    | :-------- | :------ | :------------------------------------- |
    | code      | Integer | Return result status                   |
    | msg       | String  | Failure message                        |
    | result    | jsonObj | Data return object                     |
    | value     | boolean | Whether the verification is successful |


#### 2.2.31 Alias transaction submit

- Function Description

  Alias transaction submit, save alias

- Process description

  ```
  1. deserialize txHex alias transaction data
  2. save the alias alias to the database
  3. set the alias to account and save to the database
  4. Re-cache the modified account
  5. return the alias save is successful
  ```

- ac_aliasTxCommit interface

  - Interface Description

    This interface is used to save aliases.

  - Request example

    ```
    {
      "cmd": "ac_aliasTxCommit",
      "minVersion":1.0,
      "params": [chainId,"txHex","secondaryDataHex"]
    }
    
    ```

  - Request parameter description

    | index | parameter        | required | type   | description                          |
    | ----- | ---------------- | -------- | ------ | ------------------------------------ |
    | 0     | chainId          | true     | Short  | Chain ID                             |
    | 1     | txHex            | true     | String | Alias transaction serialization data |
    | 2     | secondaryDataHex | true     | String | Block header serialization data      |

    txHex description

    ```
    {
        "type":3,
        "time":"12546545596",
        "scriptSig":"",
        "hash":"",
        "coinData":
        {
            "froms":
            [{
                "address":"Nse8m2Te1UNGPhD1tjZ3A4GDW3dCJxqE",
                "amount":10000,
                "nonce":"123",
            }],
            "to":
            [{
                "address":"Nse8m2Te1UNGPhD1tjZ3A4GDW3dCJxqE",
                "amount":1
            }]
        },
        "txData":
        {
            "chainId":"12345",
            "address":"Nse8m2Te1UNGPhD1tjZ3A4GDW3dCJxqE",
            "alias":"lucas"
        }
    }
    ```
    secondaryDataHex description

    ```
    "txData":
        {
            "hash":"",
            "height":1,
            "time":13369748564
        }
    ```

    

  - Return example

    ```
    {
        "code": 0,
        "msg": "success",
        "version":1.0,
        "result": {
           "value":true
        }
    }
    ```

  - Return field description

    | parameter | type    | description                                 |
    | :-------- | :------ | :------------------------------------------ |
    | code      | Integer | Return result status                        |
    | msg       | String  | Failure message                             |
    | result    | jsonObj | Data return object                          |
    | value     | boolean | Is the alias transaction saved successfully |

#### 2.2.32 Alias transaction rollback

- Function Description

  Alias transaction rollback interface

- Process description

  ```
  1. deserialize txHex alias transaction data
  2. delete the alias object data from the database
  3. take the corresponding account to clear the alias, re-sent the database
  4. re-cache account
  5. return the alias rollback is successful
  ```

- ac_aliasTxRollback interface

  - Interface Description

    This interface is used to roll back an alias.

  - Request example

    ```
    {
      "cmd": "ac_aliasTxRollback",
      "minVersion":1.0,
      "params": [chainId,"txHex","secondaryDataHex"]
    }
    
    ```

  - Request parameter description

    | index | parameter        | required | type   | description                          |
    | ----- | ---------------- | -------- | ------ | ------------------------------------ |
    | 0     | chainId          | true     | Short  | Chain ID                             |
    | 1     | txHex            | true     | String | Alias transaction serialization data |
    | 2     | secondaryDataHex | true     | String | Block header serialization data      |

    txHex description

    ```
    {
        "type":3,
        "time":"12546545596",
        "scriptSig":"",
        "hash":"",
        "coinData":
        {
            "froms":
            [{
                "address":"Nse8m2Te1UNGPhD1tjZ3A4GDW3dCJxqE",
                "amount":10000,
                "nonce":"123",
            }],
            "to":
            [{
                "address":"Nse8m2Te1UNGPhD1tjZ3A4GDW3dCJxqE",
                "amount":1
            }]
        },
        "txData":
        {
            "chainId":"12345",
            "address":"Nse8m2Te1UNGPhD1tjZ3A4GDW3dCJxqE",
            "alias":"lucas"
        }
    }
    ```
    secondaryDataHex description

    ```
    "txData":
        {
            "hash":"",
            "height":1,
            "time":13369748564
        }
    ```

    

  - Return example

    ```
    {
        "code": 0,
        "msg": "success",
        "version":1.0,
        "result": {
           "value":true
        }
    }
    ```

  - Return field description

    | parameter | type    | description                                  |
    | :-------- | :------ | :------------------------------------------- |
    | code      | Integer | Return result status                         |
    | msg       | String  | Failure message                              |
    | result    | jsonObj | Data return object                           |
    | value     | boolean | Is the alias transaction rollback successful |
  

  

### 2.3 Module internal function




## 3、Event description

### 3.1 Published event

* create Account

   event_topic : "evt_ac_createAccount"

  ```
  data:{
      address:''
      isEncrypted：true   //Is the password set
  }
  ```

* remove account

  event_topic : "evt_ac_removeAccount"

  ```
  data:{
  	address:''
  }
  ```

* change password

  event_topic : "evt_ac_updatePassword"

  ```
  data:{
  	address:''
  }
  ```


### 3.2 Subscribed event

no

## 4、protocol

### 4.1 Network communication protocol

- no



### 4.2 Transaction agreement

* Set alias

  * protocol

    Compared with the general transaction, only the type and txData are different, the specific difference is as follows

  ```
  type: n //Set the type of alias transaction
  txData:{
      address:  //VarByte Set the address of alias
      alias：   //VarByte Array of bytes converted into an alias string, decoded with UTF-8
  }
  ```

  - Alias transaction parameters

  | Len  | Fields  | Data Type | Remark                                                       |
  | ---- | ------- | --------- | ------------------------------------------------------------ |
  | 24   | address | byte[]    | Set the address of alias                                     |
  | 32   | alias   | byte[]    | Array of bytes converted into an alias string, decoded with UTF-8 |

  * Validator

  ```
  1. alias format legality verification.
  2. the address must be the satellite chain address, and an address can only be set to an alias.
  3. burn a token unit.
  4. Transaction fee.
  5. signature: set address, input, signature verification.
  ```

  * processor

  ```
  1. the asset processor.
  2. store alias data.
  3. Update local account information.
  ```



## 5、Module configuration item

```
server.ip:0.0.0.0   //Native ip, used to provide services to other modules
server.port:8080    //Service port
```

## 6、Java-specific design

* Account Object design

  The key used when the table is stored:

  NULS system：chainId+type+hash160

  non-NULS system：chainId+length+address


| `Field name`    | ` type` | `Description`                                      |
| :-------------- | :------ | :------------------------------------------------- |
| chainId         | short   | Chain ID                                           |
| address         | String  | Account address（Base58(address)+Base58(chainId)） |
| alias           | String  | Account alias                                      |
| status          | Integer | Account Status                                     |
| pubKey          | byte[]  | Public key                                         |
| priKey          | byte[]  | Private key - not encrypted                        |
| encryptedPriKey | byte[]  | Encrypted private key                              |
| extend          | byte[]  | Extended data                                      |
| remark          | String  | ramark                                             |
| createTime      | long    | create time                                        |

* Address object design (not persistent storage)

| `Field name` | ` type` | `Description`      |
| ------------ | ------- | ------------------ |
| chainId      | short   | Chain ID           |
| addressType  | byte    | Address type       |
| hash160      | byte[]  | Public key hash    |
| addressBytes | byte[]  | Address byte array |

- Alias object design

  The key used when the table is stored:

  Address and alias are stored as keys respectively, and alias data is stored in two copies.

  Need to create different alias tables according to different chains.

| `Field name` | ` type` | `Description`   |
| ------------ | ------- | --------------- |
| address      | byte[]  | Account address |
| alias        | String  | Account alias   |

- MultiSigAccount object design

| `Field name` | ` type`      | `Description`                           |
| ------------ | ------------ | --------------------------------------- |
| address      | String       | Account address                         |
| pubKeyList   | List<byte[]> | Public key list that needs to be signed |
| minSigns     | long         | Minimum number of signatures            |



## 

## 7、to add on

