# Protocol design

[TOC]

## Common standards

### Hashes

Usually, when a hash is computed within Satellite-chain, it is computed twice. Most of the time [SHA-256](http://en.wikipedia.org/wiki/SHA-2) hashes are used, however [RIPEMD-160](http://en.wikipedia.org/wiki/RIPEMD) is also used when a shorter hash is desirable (for example when creating a NULS address).

Example of double-SHA-256 encoding of string "nuls":

```
nuls
96cc377699e3289875d6d40da436110d692e823eb8900ae13a7107cb36c7310c (first round of sha-256)
7e0590c37ab91aacabdac1bedec9ff6acd609215d019e489cc0a5dcf38b7b055(second round of sha-256)

```

For NULS addresses (RIPEMD-160) this would give:

```
nuls
96cc377699e3289875d6d40da436110d692e823eb8900ae13a7107cb36c7310c (first round is sha-256)
f8e8d1c6f1511c356effad8b986b0aa439a88c44(with ripemd-160)
```

### Merkle Trees

Merkle trees are binary trees of hashes. Merkle trees in NULS use a **double** SHA-256, the SHA-256 hash of the SHA-256 hash of something.

If, when forming a row in the tree (other than the root of the tree), it would have an odd number of elements, the final double-hash is duplicated to ensure that the row has an even number of hashes.

First form the bottom row of the tree with the ordered double-SHA-256 hashes of the byte streams of the transactions in the block.

Then the row above it consists of half that number of hashes. Each entry is the double-SHA-256 of the 64-byte concatenation of the corresponding two hashes below it in the tree.

This procedure repeats recursively until we reach a row consisting of just a single double-hash. This is the **Merkle root** of the tree.

For example, imagine a block with three transactions *a*, *b* and *c*. The Merkle tree is:

```
d1 = dhash(a)
d2 = dhash(b)
d3 = dhash(c)
d4 = dhash(c)            # a, b, c are 3. that's an odd number, so we take the c twice

d5 = dhash(d1 concat d2)
d6 = dhash(d3 concat d4)

d7 = dhash(d5 concat d6)

```

where

```
dhash(a) = sha256(sha256(a))

```

*d7* is the Merkle root of the 3 transactions in this block.

### Addresses

A NULS address is in fact the hash of a ECDSA public key, computed this way:

```
ChainId = 2 byte,The ID of the chain in which the account belongs.
addressType = 1 byte,The type of the account,Example: 1 general account, 2 contract account……
pkh = 20 byte , RIPEMD-160(SHA-256(public key))
xor = 1 byte, XOR(chainId+addressType+pkh)
address = Base58Encode(chainId+addressType+pkh+xor)
```

非nuls体系的地址格式如下：

例如：比特币地址，在地址之前追加两个字节的chainId，之后跟随比特币的原始地址，地址解析方式根据链配置决定

```
address = Base58Encode(chainId+原始地址长度+原始地址+xor)
```

## Message Structre

Satellite-chain uses custom messaging for communication over the TCP protocol.

- Digital binary stream using little endian。
- Floating point numbers convert to integers and transfer by little endian

### Message

The message consists of a 24-byte header and payload.

```
*---------------------------------------------------------------*
|       Header(24 Byte)         |            Payload            |
*---------------------------------------------------------------*
```

#### message header

The main role of the header is to indicate the payload length, verify data integrity, and solve TCP sticky packets.

| Len  | Fields        | Data Type | Remark                                   |
| ---- | ------------- | --------- | ---------------------------------------- |
| 4    | MagicNumber   | uint32    | Packet valid flag                        |
| 12   | command       | char[12]  | ASCII string identifying the packet content, NULL padded |
| 4    | PayloadLength | uint32    | Length of payload in number of bytes     |
| 4    | checksum      | uint32    | First 4 bytes of  sha256(sha256(payload)) |
|      |               |           |                                          |

[^MagicNumber]: MagicNumber In addition to being validated as a valid check, it is also used for the division of the main network and test.

## Common Structre

### VarInt

Variable-length integers that can be encoded based on the values expressed to save space.

| Value         | Len  | Structure     |
| ------------- | ---- | ------------- |
| < 0xfd        | 1    | uint8         |
| <= 0xffff     | 3    | 0xfd + uint16 |
| <= 0xffffffff | 5    | 0xfe + uint32 |
| > 0xffffffff  | 9    | 0xff + uint64 |

### VarString

A variable-length byte stream consisting of a variable-length buffer. The string is encoded in UTF8.

| Len    | Fields | Data Type     | Remark               |
| ------ | ------ | ------------- | -------------------- |
| ?      | length | VarInt        | Length of the string |
| length | value  | uint8[length] | The string itself    |

### VarByte

Variable-length buffer, consistent with the VarString implementation.

| Len    | Fields | Data Type    | Remark                               |
| ------ | ------ | ------------ | ------------------------------------ |
| ?      | length | VarInt       | Length of payload in number of bytes |
| length | data   | byte[length] | payload                              |

### Int48

6-byte number.

### Network address

When a network address is needed somewhere, this structure is used. 

| Len  | Fields | Data type | Remark                                                       |
| ---- | ------ | --------- | ------------------------------------------------------------ |
| 16   | IPv6/4 | char[16]  | IPv6/4 address. Network byte order.  The IPv4 address is written into the message as a 16 byte |
| 2    | port1  | uint16    | port number, network byte order                              |
| 2    | port2  | uint16    | port number,for cross-chain module                           |

## Block_headers

| Len  | Fields     | Data Type | Remark                             |
| ---- | ---------- | --------- | ---------------------------------- |
| 4    | version    | uint32    | version bumber                     |
| 32   | preHash    | byte[32]  |                                    |
| 32   | merkleRoot | byte[32]  |                                    |
| ？   | stateRoot  | VarByte   |                                    |
| 4    | time       | uint32    | second                             |
| 2    | txCount    | uint16    |                                    |
| ？   | extends    | VarByte   | 本字段内容不同的链可设置不同的约束 |
| ?    | signature  | Varbyte   |                                    |

[^ 注]: 可以考虑是否添加stateRoot字段适配智能合约

## Transactions

| Len  | Fields   | Data Type | Remark                   |
| ---- | -------- | --------- | ------------------------ |
| 2    | type     | uint16    | 交易类型                 |
| 4    | time     | uint32    | 时间，精确到秒           |
| ？   | txData   | VarByte   | 业务数据                 |
| ？   | coinData | VarByte   | 资产数据                 |
| ？   | remark   | VarString | 备注                     |
| ？   | sig      | VarByte   | 只包含签名，不包含pubkey |
|      |          |           |                          |

交易特性

```
多账户转出
多账户转入
多资产转账
```

coinData结构

```
froms://List<CoinForm>格式，
tos://List<CoinTo>格式
```

CoinForm结构[40]

```
address:  //byte[24] 账户地址
assetsChainId://uint16 资产发行链的id
assetsId: //uint16 资产id
amount：  //uint128，转出数量
nonce  ： //byte[8] 交易顺序号，前一笔交易的hash的后8个字节
locked ： //byte 是否是锁定状态(locktime:-1),1代表锁定，0代表非锁定
```

CoinTo结构[44]

```
address:  //byte[24],目标地址
assetsChainId://uint16 资产发行链的id
assetsId: //uint16 资产id
amount :  //uint128，转账金额
lockTime：//uint32,解锁高度或解锁时间，-1为永久锁定
```

手续费

```
forms-tos剩余的部分就是手续费（模型中支持多种资产缴纳手续费，约束条件由经济模型设计决定）
```

## Message Types

参考各个模块





