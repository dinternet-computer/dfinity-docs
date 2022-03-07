# `Rosetta`质押教程（Rosetta staking tutorial）

本文档将引导你完成使用`Rosetta API`的`IC`实现创建神经元的过程。我们将资金转移到治理罐头，创建一个神经元，并配置新创建的神经元。

## 前置要求（Prerequisites）

-----

你需要以下内容才能完成本教程：

1. 一个正在运行的`Rosetta`节点，你可以使用来自`https://hub.docker.com/r/dfinity/rosetta-api`的最新`docker`镜像。本教程假设`Rosetta`节点在地址`localhost:8080`上运行。
2. curl
3. 一个`ed25519`密钥和你最喜欢使用该密钥签署消息的工具。
4. 这个密钥控制的`ledger`罐头中的账户上至少有`2`个`ICP`代币（请参阅`ledger`以进行质押部分）。

## 概述（Overview）

------

从`IC`的角度看来，创建一个神经元是一个多步骤的操作：

1. 调用`ledger`罐头将代币转移到治理罐头（`governance canister`）以进行质押。
2. 通知治理层有关转移。在这一步，治理罐头分配一个新的神经元。
3. 配置溶解的神经元：设置溶解延迟并开始溶解。

`Rosetta API`将这些步骤表示为事务的操作。这些操作的执行必须遵循构造`API`流程。

## 派生账户标识符（Deriving account identifiers）

-----

`Staking`需要将代币转移到治理罐头的子账户。要完成转账，你需要知道你的账本账户标识符（转账来源）和治理子账户的账户标识符（转装目的地）。

调用`/construction/derive`接口以获取由你的公钥控制的默认`ledger`账户以及由该密钥控制的神经元账户。

## 计算源`ledger`账户（Compute the source ledger account）

----


> 注意，将以下请求中的`hex_bytes`替换为你的公钥字节。

``` bash
curl -0 -X POST http://localhost:8080/construction/derive \
  -H 'Content-Type: application/json; charset=utf-8' \
  --data-binary @- <<EOF
{
  "network_identifier": { "blockchain": "Internet Computer", "network": "00000000000000020101" },
  "public_key": {
    "hex_bytes": "ba5242d02642aede88a5f9fe82482a9fd0b6dc25f38c729253116c6865384a9d",
    "curve_type": "edwards25519"
  }
}
EOF
```

预期的响应如下：

``` json
{
  "account_identifier": {
    "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
  }
}
```

## 计算用于质押的`ledger`账户（Compute the ledger account for staking）

----

> 注意，将以下请求中的`hex_bytes`替换为你的公钥字节。

``` bash
curl -0 -X POST http://localhost:8080/construction/derive \
  -H 'Content-Type: application/json; charset=utf-8' \
  --data-binary @- <<EOF
{
  "network_identifier": { "blockchain": "Internet Computer", "network": "00000000000000020101" },
  "public_key": {
    "hex_bytes": "ba5242d02642aede88a5f9fe82482a9fd0b6dc25f38c729253116c6865384a9d",
    "curve_type": "edwards25519"
  },
  "metadata": {
    "account_type": "neuron",
    "neuron_index": 0
  }
}
```

> 注意，我们在请求元数据将`account_type`设置为`neuron`，以告诉`Rosetta`节点我们想要获取用于质押的账户。`neuron_index`参数是调用者选择用来标识神经元的任意`64`位无符号整数。单个用户可以控制多个神经元并通过指定不同的`neuron_index`值来区分它们。`neuron_index`是可选的，默认等于`0`。

响应如下：

``` json
{
  "account_identifier": {
    "address": "92bfc8dd46076c46de4444f15afbb8f7a5050af240384a8a2d115b359f100fc6"
  }
}
```

## 获取要签名的有效载荷（Obtain payloads to sign）

----

注意：

- 将`address`替换为你在前面的步骤中获得的地址。只有第二个`TRANSACTION`操作（下面代码片段中的索引为`1`）应该使用神经元的账本地址。所有其它操作都应使用神经元控制器的账本地址。
- `TRANSACTION`操作中的金额是你质押的金额。再此示例中，金额为`1ICP`。
- 调整`SET_DISSOLVE_TIMESTAMP`操作的溶解时间`_utc_seconds`元数据字段。当前时间戳加上`31557600`秒对应于一年的质押。
- 所有神经元管理操作都支持可选的`neuron_index`元数据字段。你应该使用在计算`ledger`账户以进行质押步骤中指定的相同的神经元索引值。

``` bash
curl -0 -X POST http://localhost:8080/construction/payloads \
  -H 'Content-Type: application/json; charset=utf-8' \
  --data-binary @- <<EOF
{
  "network_identifier": { "blockchain": "Internet Computer", "network": "00000000000000020101" },
  "operations": [
    {
      "operation_identifier": { "index": 0 },
      "type": "TRANSACTION",
      "account": {
        "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
      },
      "amount": {
        "value": "-100000000",
        "currency": { "symbol": "ICP", "decimals": 8 }
      }
    },
    {
      "operation_identifier": { "index": 1 },
      "type": "TRANSACTION",
      "account": {
        "address": "92bfc8dd46076c46de4444f15afbb8f7a5050af240384a8a2d115b359f100fc6"
      },
      "amount": {
        "value": "100000000",
        "currency": { "symbol": "ICP", "decimals": 8 }
      }
    },
    {
      "operation_identifier": { "index": 2 },
      "type": "FEE",
      "account": {
        "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
      },
      "amount": {
        "value": "-10000",
        "currency": { "symbol": "ICP", "decimals": 8 }
      }
    },
    {
      "operation_identifier": { "index": 3 },
      "type": "STAKE",
      "account": {
        "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
      },
      "metadata": {
        "neuron_index": 0
      }
    },
    {
      "operation_identifier": { "index": 4 },
      "type": "SET_DISSOLVE_TIMESTAMP",
      "account": {
        "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
      },
      "metadata": {
        "neuron_index": 0,
        "dissolve_time_utc_seconds": 1879939507
      }
    },
    {
      "operation_identifier": { "index": 5 },
      "type": "START_DISSOLVE",
      "account": {
        "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
      },
      "metadata": {
        "neuron_index": 0
      }
    }
  ],
  "public_keys": [
    {
      "hex_bytes": "ba5242d02642aede88a5f9fe82482a9fd0b6dc25f38c729253116c6865384a9d",
      "curve_type": "edwards25519"
    }
  ]
}
EOF
```

预期响应如下：

``` json
{
  "unsigned_transaction": "a2677570646174657384826b5452414e53414354494f4ea56b63616e69737465725f69644a000000000000000201016b6d6574686f645f6e616d656773656e645f706263617267584b0a0b089a8fa881d7d8b4b0c50112070a050880c2d72f1a0308904e2a220a2092bfc8dd46076c46de4444f15afbb8f7a5050af240384a8a2d115b359f100fc63a0a08a8edf6e.....",
  "payloads": [
    {
      "account_identifier": {
        "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
      },
      "hex_bytes": "0a69632d72657175657374e5f2ad1ffc1c7136c7dc59a183b9dc85af38c620d46492676eb016bb2c7b9a56",
      "signature_type": "ed25519"
    },
    {
      "account_identifier": {
        "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
      },
      "hex_bytes": "0a69632d726571756573742eae2f191b885b6ed7238b2e67001569ce1f789873ebf312e84a8e5dbc9a38d0",
      "signature_type": "ed25519"
    },
    {
      "account_identifier": {
        "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
      },
      "hex_bytes": "0a69632d72657175657374788935ee98703620ec63813b2036daa46989ead54a6b940264d44a9fc0241fae",
      "signature_type": "ed25519"
    },
    {
      "account_identifier": {
        "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
      },
      "hex_bytes": "0a69632d7265717565737463166a6d97d170f9a4a49c779cc032b376b441bbb56346730ec92e3033eebe9f",
      "signature_type": "ed25519"
    },
    {
      "account_identifier": {
        "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
      },
      "hex_bytes": "0a69632d726571756573743652695a86832a2bb361f3ba21418f461ca5f604c2d43b47d05eeb5ba77f114f",
      "signature_type": "ed25519"
    },
    {
      "account_identifier": {
        "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
      },
      "hex_bytes": "0a69632d72657175657374f80fa04953de2944d8d8a0c1ec24fd43008f9636ccb2eaa3e4163145a4071e60",
      "signature_type": "ed25519"
    },
    {
      "account_identifier": {
        "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
      },
      "hex_bytes": "0a69632d72657175657374f79dc941816eb5744a48e0ba6ed493442c4089ac5430de5b3e11d925db436a20",
      "signature_type": "ed25519"
    },
    {
      "account_identifier": {
        "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
      },
      "hex_bytes": "0a69632d726571756573748a67e5347e053335269988ff1a00b19b426ea389ad1b9f17e38a698b21f9884a",
      "signature_type": "ed25519"
    }
  ]
}
```

> 注意，响应中的`payload`字段包含在提交交易之前需要签名的有效负载列表。

> 注意，根据`Rosetta API`规范（操作流程），客户端在调用`/construction/payloads`之前调用`/construction/preprocess`和`/construction/metadata`，因为可能有一些元数据需要附加到负载请求。目前，`IC`的`Rosetta`节点的实现不需要这样做，所以我们跳过了这些不必要的步骤。

## 检查你将要签署的交易的内容（Check the contents of the transaction you are about to sign）

-----

调用`/construction/parse`接口来解码事务的内容。

> 注意，将下面的`transaction`字段的值替换为你从上一步中获得的`unsigned_transaction`字段的值。请注意，签名字段设置为`false`。

``` bash
curl -0 -X POST http://localhost:8080/construction/parse \
  -H 'Content-Type: application/json; charset=utf-8' \
  --data-binary @- <<EOF
{
  "network_identifier": { "blockchain": "Internet Computer", "network": "00000000000000020101" },
  "signed": false,
  "transaction": "a2677570646174657384826b5452414e53414354494f4ea56b63616e69737465725f69644a000000000000000201016b6d6574686f645f6e616d656773656e645f706263617267584b0a0b089a8fa881d7d8b4b0c50112070a050880c2d72f1a0308904e2a220a2092bfc8dd46076c46de4444f15afbb8f7a5050af240384a8a2d115b359f100fc63a0a08a8edf6e....."
}
EOF
```

响应必须与交易意图相匹配：

``` json
{
  "operations": [
    {
      "operation_identifier": {"index": 0},
      "type": "TRANSACTION",
      "status": null,
      "account": {
        "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
      },
      "amount": {
        "value": "-100000000",
        "currency": {"symbol": "ICP", "decimals": 8}
      }
    },
    {
      "operation_identifier": {"index": 1},
      "type": "TRANSACTION",
      "status": null,
      "account": {
        "address": "92bfc8dd46076c46de4444f15afbb8f7a5050af240384a8a2d115b359f100fc6"
      },
      "amount": {
        "value": "100000000",
        "currency": {"symbol": "ICP", "decimals": 8}
      }
    },
    {
      "operation_identifier": {"index": 2},
      "type": "FEE",
      "status": null,
      "account": {
        "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
      },
      "amount": {
        "value": "-10000",
        "currency": {"symbol": "ICP", "decimals": 8}
      }
    },
    {
      "operation_identifier": {"index": 3},
      "type": "STAKE",
      "status": null,
      "account": {
        "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
      },
      "metadata": {
        "neuron_index": 0
      }
    },
    {
      "operation_identifier": {"index": 4},
      "type": "SET_DISSOLVE_TIMESTAMP",
      "status": null,
      "account": {
        "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
      },
      "metadata": {
        "dissolve_time_utc_seconds": 1879939507,
        "neuron_index": 0
      }
    },
    {
      "operation_identifier": {"index": 5},
      "type": "START_DISSOLVE",
      "status": null,
      "account": {
        "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
      },
      "metadata": {
        "neuron_index": 0
      }
    }
  ],
  "account_identifier_signers": []
}
```

## 签署有效载荷并构建签署的交易（Sign payloads and construct a signed transaction）

----

使用你选择的工具对`/constrcution/payloads`响应中的有效负载进行签名。将签名包好到`/construction/combine`请求中，如下面的命令所示：

``` bash
curl -0 -X POST http://localhost:8080/construction/combine \
  -H 'Content-Type: application/json; charset=utf-8' \
  --data-binary @- <<EOF
{
  "network_identifier": { "blockchain": "Internet Computer", "network": "00000000000000020101" },
  "signatures": [
    {
      "signing_payload": {
        "account_identifier": {
          "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
        },
        "hex_bytes": "0a69632d72657175657374e5f2ad1ffc1c7136c7dc59a183b9dc85af38c620d46492676eb016bb2c7b9a56",
        "signature_type": "ed25519"
      },
      "public_key": {
        "hex_bytes": "ba5242d02642aede88a5f9fe82482a9fd0b6dc25f38c729253116c6865384a9d",
        "curve_type": "edwards25519"
      },
      "signature_type": "ed25519",
      "hex_bytes": "5a5297f7555df2d98bd945ecff7afb11456164621da2439f665900abaaa17655e2e4801284c458639fc0e648037cc70116bfafe0315338897f63f9c3bb8b150b"
    },
    {
      "signing_payload": {
        "account_identifier": {
          "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
        },
        "hex_bytes": "0a69632d726571756573742eae2f191b885b6ed7238b2e67001569ce1f789873ebf312e84a8e5dbc9a38d0",
        "signature_type": "ed25519"
      },
      "public_key": {
        "hex_bytes": "ba5242d02642aede88a5f9fe82482a9fd0b6dc25f38c729253116c6865384a9d",
        "curve_type": "edwards25519"
      },
      "signature_type": "ed25519",
      "hex_bytes": "ab973292cdcbe1c3f54d07a7a822d3c48616abfd9067987e105d8901d0ab946a4b2b6ea038a28e4c8af66e4aecaf503909d67c8dd4153625a5dd121ea45fa301"
    },
    {
      "signing_payload": {
        "account_identifier": {
          "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
        },
        "hex_bytes": "0a69632d72657175657374788935ee98703620ec63813b2036daa46989ead54a6b940264d44a9fc0241fae",
        "signature_type": "ed25519"
      },
      "public_key": {
        "hex_bytes": "ba5242d02642aede88a5f9fe82482a9fd0b6dc25f38c729253116c6865384a9d",
        "curve_type": "edwards25519"
      },
      "signature_type": "ed25519",
      "hex_bytes": "adc30181799a4474ad03c3a909dcd1d8ee08fe9d23f74314a031ebeb69a340a87680d9549ed6632956130d9e62ee3fa487402c7ab1dd3497aeaf83d36bedec05"
    },
    {
      "signing_payload": {
        "account_identifier": {
          "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
        },
        "hex_bytes": "0a69632d7265717565737463166a6d97d170f9a4a49c779cc032b376b441bbb56346730ec92e3033eebe9f",
        "signature_type": "ed25519"
      },
      "public_key": {
        "hex_bytes": "ba5242d02642aede88a5f9fe82482a9fd0b6dc25f38c729253116c6865384a9d",
        "curve_type": "edwards25519"
      },
      "signature_type": "ed25519",
      "hex_bytes": "e5a4c7bdd8a17e6581f05c25d9532fd322ea18b1fdd03b06ea40f886c50a74e6adfa530dc9a5a6a0754535004eaa4cf2dbf3fab9684e0d495f3b4eb4ff449b0e"
    },
    {
      "signing_payload": {
        "account_identifier": {
          "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
        },
        "hex_bytes": "0a69632d726571756573743652695a86832a2bb361f3ba21418f461ca5f604c2d43b47d05eeb5ba77f114f",
        "signature_type": "ed25519"
      },
      "public_key": {
        "hex_bytes": "ba5242d02642aede88a5f9fe82482a9fd0b6dc25f38c729253116c6865384a9d",
        "curve_type": "edwards25519"
      },
      "signature_type": "ed25519",
      "hex_bytes": "e2ada2cf653ebbba2a93d1b013415c5102881ae6a5c58e9abf8683a33aa624e404d2958e1b47e0e98f9e37092221fd816fc13b965ca98fc1e61c4653a279640f"
    },
    {
      "signing_payload": {
        "account_identifier": {
          "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
        },
        "hex_bytes": "0a69632d72657175657374f80fa04953de2944d8d8a0c1ec24fd43008f9636ccb2eaa3e4163145a4071e60",
        "signature_type": "ed25519"
      },
      "public_key": {
        "hex_bytes": "ba5242d02642aede88a5f9fe82482a9fd0b6dc25f38c729253116c6865384a9d",
        "curve_type": "edwards25519"
      },
      "signature_type": "ed25519",
      "hex_bytes": "0dc0bce4b86c0fd6a1b21ddaa5380b432a637bc5084c1b77356188308618bd9f97513ae5d351a2067c6e8ae8ce7f022ded21f21c3aaaf030a2f47040df1db90b"
    },
    {
      "signing_payload": {
        "account_identifier": {
          "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
        },
        "hex_bytes": "0a69632d72657175657374f79dc941816eb5744a48e0ba6ed493442c4089ac5430de5b3e11d925db436a20",
        "signature_type": "ed25519"
      },
      "public_key": {
        "hex_bytes": "ba5242d02642aede88a5f9fe82482a9fd0b6dc25f38c729253116c6865384a9d",
        "curve_type": "edwards25519"
      },
      "signature_type": "ed25519",
      "hex_bytes": "fb216f00ff6540e9759e8b6095487a7ea258f4d3fa459f30bb58816932082a14df78c22924941ff3a1897bf3dc468ceefe0e83d94580cdd2ec21ba3a51380e05"
    },
    {
      "signing_payload": {
        "account_identifier": {
          "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
        },
        "hex_bytes": "0a69632d726571756573748a67e5347e053335269988ff1a00b19b426ea389ad1b9f17e38a698b21f9884a",
        "signature_type": "ed25519"
      },
      "public_key": {
        "hex_bytes": "ba5242d02642aede88a5f9fe82482a9fd0b6dc25f38c729253116c6865384a9d",
        "curve_type": "edwards25519"
      },
      "signature_type": "ed25519",
      "hex_bytes": "b120ab9da353f23cbc854f09df28e8a59a9227a4e9d615dbe102ea0f7f6991a34f109accd03d9c8ac3e30ecf2f22fdcea2019424c851bd8a3d68bcd2dba6fc00"
    }
  ],
  "unsigned_transaction": "a2677570646174657384826b5452414e53414354494f4ea56b63616e69737465725f69644a000000000000000201016b6d6574686f645f6e616d656773656e645f706263617267584b0a0b089a8fa881d7d8b4b0c50112070a050880c2d72f1a0308904e2a220a2092bfc8dd46076c46de4444f15afbb8f7a5050af240384a8a2d115b359f100fc63a0a08a8edf6e..."
}
EOF
```

预期响应如下：

``` json
{
  "signed_transaction": "84826b5452414e53414354494f4e81a266757064617465a367636f6e74656e74bf6c726571756573745f747970656463616c6c6b63616e69737465725f69644a000000000000000201016b6d6574686f645f6e616d656773656e645f706263617267584b0a0b089a8fa881d7d8b4b0c50112070a050880c2d72f1a0308904e2a220a2092bfc8dd46076c46de4444f15afbb8f7a5050af240384a8a2d115b359f100fc63a0a08a8edf6e0fd928cdf166673656e646572581d48c5a6e93d9a3d10c4ba3c7927c97c18df4b0fcdac3a38ad3f08106e026e..."
}
```


## 检查你将要提交的交易的内容（Check the contents of the transaction you are about ot submit）

-----

调用`/construction/parse`接口来解码事务的内容。

> 注意，将下面的`transaction`字段的值替换为你从上一步中获得的`signed_transaction`字段的值。请注意，签名字段设置为`true`。

``` bash
curl -0 -X POST http://localhost:8080/construction/parse \
  -H 'Content-Type: application/json; charset=utf-8' \
  --data-binary @- <<EOF
{
  "network_identifier": { "blockchain": "Internet Computer", "network": "00000000000000020101" },
  "signed": true,
  "transaction": "84826b5452414e53414354494f4e81a266757064617465a367636f6e74656e74bf6c726571756573745f747970656463616c6c6b63616e69737465725f69644a000000000000000201016b6d6574686f645f6e616d656773656e645f706263617267584b0a0b089a8fa881d7d8b4b0c50112070a050880c2d72f1a0308904e2a220a2092bfc8dd46076c46de4444f15afbb8f7a5050af240384a8a2d115b359f100fc63a0a08a8edf6e0fd928cdf166673656e646572581d48c5a6e93d9a3d10c4ba3c7927c97c18df4b0fcdac3a38ad3f08106e026e696e67726573735f6578706972791b16be30cfbd3b16a8ff6d73656e6465725f7075626b6579582c302a300506032b6570032100ba5242d02642aede88a5f9fe82482a9fd0b6dc2...."
}
EOF
```

响应必须与交易意图相匹配：

``` json
{
  "operations": [
    {
      "operation_identifier": {"index": 0},
      "type": "TRANSACTION",
      "status": null,
      "account": {
        "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
      },
      "amount": {
        "value": "-100000000",
        "currency": {"symbol": "ICP", "decimals": 8}
      }
    },
    {
      "operation_identifier": {"index": 1},
      "type": "TRANSACTION",
      "status": null,
      "account": {
        "address": "92bfc8dd46076c46de4444f15afbb8f7a5050af240384a8a2d115b359f100fc6"
      },
      "amount": {
        "value": "100000000",
        "currency": {"symbol": "ICP", "decimals": 8}
      }
    },
    {
      "operation_identifier": {"index": 2},
      "type": "FEE",
      "status": null,
      "account": {
        "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
      },
      "amount": {
        "value": "-10000",
        "currency": {"symbol": "ICP", "decimals": 8}
      }
    },
    {
      "operation_identifier": {"index": 3},
      "type": "STAKE",
      "status": null,
      "account": {
        "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
      },
      "metadata": {
        "neuron_index": 0
      }
    },
    {
      "operation_identifier": {"index": 4},
      "type": "SET_DISSOLVE_TIMESTAMP",
      "status": null,
      "account": {
        "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
      },
      "metadata": {
        "dissolve_time_utc_seconds": 1879939507,
        "neuron_index": 0
      }
    },
    {
      "operation_identifier": {"index": 5},
      "type": "START_DISSOLVE",
      "status": null,
      "account": {
        "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
      },
      "metadata": {
        "neuron_index": 0
      }
    }
  ],
  "account_identifier_signers": [
    {
      "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
    }
  ]
}
```

## 提交交易（Submit the transaction）

-----

调用`/construction/submit`接口以提交签名交易。将以下请求中的`signed_transaction`字段替换为你从`/construction/combine`接口获得的`singed_transaction`字段的值。

``` bash
curl -0 -X POST http://localhost:8080/construction/submit \
  -H 'Content-Type: application/json; charset=utf-8' \
  --data-binary @- <<EOF
{
  "network_identifier": { "blockchain": "Internet Computer", "network": "00000000000000020101" },
  "signed_transaction": "84826b5452414e53414354494f4e81a266757064617465a367636f6e74656e74bf6c726571756573745f747970656463616c6c6b63616e69737465725f69644a000000000000000201016b6d6574686f645f6e616d656773656e645f706263617267584b0a0b089a8fa881d7d8b4b0c50112070a050880c2d72f1a0308904e2a220a2092bfc8dd46076c46de4444f...."
}
EOF
```

预期响应如下：

``` json
{
  "transaction_identifier": {
    "hash": "aa0de8d73d930e49f6e3fd4db66cb34360710ee57b9aef5ef66e2c1cb0513b65"
  },
  "metadata": {
    "operations": [
      {
        "operation_identifier": { "index": 0 },
        "status": "COMPLETED",
        "type": "TRANSACTION",
        "account": {
          "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
        },
        "amount": {
          "value": "-100000000",
          "currency": { "symbol": "ICP", "decimals": 8 }
        }
      },
      {
        "operation_identifier": { "index": 1 },
        "status": "COMPLETED",
        "type": "TRANSACTION",
        "account": {
          "address": "92bfc8dd46076c46de4444f15afbb8f7a5050af240384a8a2d115b359f100fc6"
        },
        "amount": {
          "value": "100000000",
          "currency": { "symbol": "ICP", "decimals": 8 }
        }
      },
      {
        "operation_identifier": { "index": 2 },
        "status": "COMPLETED",
        "type": "FEE",
        "account": {
          "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
        },
        "amount": {
          "value": "-10000",
          "currency": { "symbol": "ICP", "decimals": 8 }
        }
      },
      {
        "operation_identifier": { "index": 3 },
        "status": "COMPLETED",
        "type": "STAKE",
        "account": {
          "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
        },
        "metadata": {
          "neuron_index": 0,
          "neuron_id": 8654960738044813476
        }
      },
      {
        "operation_identifier": { "index": 4 },
        "status": "COMPLETED",
        "type": "SET_DISSOLVE_TIMESTAMP",
        "account": {
          "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
        },
        "metadata": {
          "neuron_index": 0,
          "dissolve_time_utc_seconds": 1879939507
        }
      },
      {
        "operation_identifier": { "index": 5 },
        "status": "COMPLETED",
        "type": "START_DISSOLVE",
        "account": {
          "address": "8af54f1fa09faeca18d294e0787346264f9f1d6189ed20ff14f029a160b787e8"
        },
        "metadata": {
          "neuron_index": 0
        }
      }
    ]
  }
}
```

> 注意，神经元管理操作并不严格遵守`Rosetta`规范。如果你使用`/block/transaction`接口通过哈希查找上面的交易，它将只包含`ledger`转移。此外，如果交易只包含神经元管理操作，则返回的交易哈希根本不会在账本链上。这就是为什么响应元数据字段包含所有操作的状态，格式相同`/block/transaction`将返回它们。`STAKE`操作返回治理罐头的神经元的唯一标识符。你可以使用此标识符在`NNS`或`https://ic.rocks`上查找神经元。

## 检查神经元的状态（Check the status of the neuron）

----

现在让我们检查新创建神经元的状态。

> 注意，将下面的`address`替换为你在步骤[计算用于质押的`ledger`账户](#计算用于质押的ledger账户compute-the-ledger-account-for-staking)中的神经元的地址；将`public_key.hex_bytes`替换为你的公钥的字节。

``` bash
curl -0 -X POST http://localhost:8080/account/balance \
  -H 'Content-Type: application/json; charset=utf-8' \
  --data-binary @- <<EOF
{
  "network_identifier": { "blockchain": "Internet Computer", "network": "00000000000000020101" },
  "account_identifier": {
    "address": "92bfc8dd46076c46de4444f15afbb8f7a5050af240384a8a2d115b359f100fc6"
  },
  "metadata": {
    "account_type": "neuron",
    "neuron_index": 0,
    "public_key": {
      "hex_bytes": "ba5242d02642aede88a5f9fe82482a9fd0b6dc25f38c729253116c6865384a9d",
      "curve_type": "edwards25519"
    }
  }
}
EOF
```

预期响应如下：

``` json
{
  "block_identifier": {
    "index": 1157,
    "hash": "04012aff1447b92589ccafab3c45243db4aa4b7c36c93fc5d22a7cac15ce7797"
  },
  "balances": [
    {
      "value": "100000000",
      "currency": { "symbol": "ICP", "decimals": 8 }
    }
  ],
  "metadata": {
    "verified_query": false,
    "retrieved_at_timestamp_seconds": 1640273314,
    "state": "DISSOLVING",
    "age_seconds": 0,
    "dissolve_delay_seconds": 239666197,
    "voting_power": 779728174,
    "created_timestamp_seconds": 1638799843
  }
}
```
