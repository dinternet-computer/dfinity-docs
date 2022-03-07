# 标准代币转账（Regular token transfers）

本文档详细记录了如何使用`Rosetta`接口来转账`ICP`代币。相关高级概述，请参阅[`API`概述](https://www.rosetta-api.org/docs/construction_api_introduction.html)。

## 转账操作（Transfer operations）

-----

从地址`A`转账数量`T`的代币到地址`B`的交易必须包含`3`个操作：

- `TRANSACTION`类型的操作作用于地址`A`，数量为`-T`。
- `TRANSACTION`类型的操作作用于地址`B`，数量为`T`。
- `FEE`类型的操作作用于地址`A`，其数量由`/construction/metadata`接口提供（请参阅[`ConstructionMetaResponse`类型](https://www.rosetta-api.org/docs/models/ConstructionMetadataResponse.html)的`suggested_fee`字段）。

交易中的操作顺序无关紧要。

不允许在单笔交易中进行多次转账。这种交易的结果是未定义的。

前提条件：

- 地址`A`至少持有`T`+`suggested_fee`数量的`ICP`代币。
- 地址`A`是从你用于签署交易的公钥派生的委托人的子账户。
- `FEE`操作中指定的金额绝对要等于`suggested_fee`。

## 可选元数据字段（Optional metadata fields）

------

`Rosetta`节点识别`ConstructionPayloadRequest`中的以下可选元数据字段：

- `memo`是与交易关联的任意`64`位无符号整数。你可以使用它将你的数据与交易相关联。例如，你可以将备忘录设置为数据库中的行键。备注字段的有效值范围为`0`到`2^64-1`。
- `ingress_start`、`ingress_at_time`是`64`位无符号整数，表示`UTC`时区中从`UNIX`纪元传递的纳秒数。你可以使用这些字段提前构建和签署交易，并在稍后提交签署的交易。你可以在`created_at_time`开始的`24`小时内提交签名交易（默认情况下等于你调用`/contruction/payloads`接口的时间）。

### 例子（Example）

下面是一个将`1`个`ICP`代币从地址`bdc4ee05d42cd0669786899f256c8fd7217fa71177bd1fa7b9534f568680a938`转移到地址`b64ec6f964d8597afa06d4209dbce2b2df9fe722e86aeda2351bd9550cfff800`的交易示例：

``` json
{
  "network_identifier": {
    "blockchain": "Internet Computer",
    "network": "00000000000000020101"
  },
  "operations": [
    {
      "operation_identifier": {
        "index": 0
      },
      "type": "TRANSACTION",
      "account": {
        "address": "bdc4ee05d42cd0669786899f256c8fd7217fa71177bd1fa7b9534f568680a938"
      },
      "amount": {
        "value": "-100000000",
        "currency": {
          "symbol": "ICP",
          "decimals": 8
        }
      }
    },
    {
      "operation_identifier": {
        "index": 1
      },
      "type": "TRANSACTION",
      "account": {
        "address": "b64ec6f964d8597afa06d4209dbce2b2df9fe722e86aeda2351bd95500cf15f8"
      },
      "amount": {
        "value": "100000000",
        "currency": {
          "symbol": "ICP",
          "decimals": 8
        }
      }
    },
    {
      "operation_identifier": {
        "index": 2
      },
      "type": "FEE",
      "account": {
        "address": "bdc4ee05d42cd0669786899f256c8fd7217fa71177bd1fa7b9534f568680a938"
      },
      "amount": {
        "value": "-10000",
        "currency": {
          "symbol": "ICP",
          "decimals": 8
        }
      }
    }
  ],
  "public_keys": [
    {
      "hex_bytes": "97d0b490ec4097b3653878274b1d9dd00bb1316ea3df0bfdf98327ef68fade63",
      "curve_type": "edwards25519"
    }
  ]
}
```

### 响应（Response）

``` json
{
  "transaction_identifier": {
    "hash": "97f4a8289f96ef46d8c8fa911f13cc402e4f69b36f4dd1ddc2579bb54dba5557"
  },
  "block_index": 1043
}
```