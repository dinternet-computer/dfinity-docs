# 质押和神经元管理（Staking and neuron management）

本文档详细说明了`Rosetta API`的扩展，可以在`IC`上进行质押资金和管理治理“神经元”。

> 注意，交易中的操作是按顺序应用的，因此操作的顺序很重要。包含此`API`提供的幂等操作的事务可以在`24`小时内的时间窗口重试。

> 注意，由于治理罐头的限制，神经元管理操作没有反映到链上。如果你通过从`/construction/submit`接口返回的标识符查找交易，这些交易可能不存在或丢失神经元操作。相反`/construction/submit`接口使用与`/block/transaction`接口将返回的格式相同的格式，返回元数据字段中所有操作的状态。

## 导出神经元地址（Deriving neuron address）

------

|开始提供的版本|1.3.0|
|------|------|

调用`/construction/derive`接口，元数据字段`account_type`设置为`neuron`，计算公钥控制的神经元对应的账本地址。

**请求**：

``` json
{
  "network_identifier": {
    "blockchain": "Internet Computer",
    "network": "00000000000000020101"
  },
  "public_key": {
    "hex_bytes": "1b400d60aaf34eaf6dcbab9bba46001a23497886cf11066f7846933d30e5ad3f",
    "curve_type": "edwards25519"
  },
  "metadata": {
    "account_type": "neuron",
    "neuron_index": 0
  }
}
```

> 注意，从`1.3.0`版本开始，你可以使用同一个键控制多个神经元。你可以通过指定`neuron_index`元数据字段的不同值来区分神经元。`Rosetta`节点支持所有神经元管理操作中的`neuron_index`是`0`到`2^64-1`之间的任意整数。如果未指定，则为零。如果你使用`javascript`构造对`Rosetta`节点的请求，请考虑使用`BigInt`类型来表示神经元索引。`Number`类型只能表示精度低于`2^53-1`的值。

**响应**：

``` json
{
  "account_identifier": {
    "address": "531b163cd9d6c1d88f867bdf16f1ede020be7bcd928d746f92fbf7e797c5526a"
  }
}
```

## 质押资金（Stake funds）

-----

|开始提供的版本|1.0.5|
|-------|-----|
|是否幂等|是|

要质押资金，请执行向神经元地址转账，然后执行`STAKE`操作。

你必须为`STAKE`操作设置唯一字段是账户，它应该等于神经元控制器的`ledger`账户。你可以在`STAKE`操作的元数据字段中指定`neuron_index`字段。如果你确实指定了神经元索引，它的值必须与你用于派生神经元账户标识符的值相同。

**请求**：

``` json
{
  "network_identifier": {
    "blockchain": "Internet Computer",
    "network": "00000000000000020101",
  },
  "operations": [
    {
      "operation_identifier": { "index": 0 },
      "type": "TRANSACTION",
      "account": { "address": "907ff6c714a545110b42982b72aa39c5b7742d610e234a9d40bf8cf624e7a70d" },
      "amount": {
        "value": "-100000000",
        "currency": { "symbol": "ICP", "decimals": 8 }
      }
    },
    {
      "operation_identifier": { "index": 1 },
      "type": "TRANSACTION",
      "account": { "address": "531b163cd9d6c1d88f867bdf16f1ede020be7bcd928d746f92fbf7e797c5526a" },
      "amount": {
        "value": "100000000",
        "currency": { "symbol": "ICP", "decimals": 8 }
      }
    },
    {
      "operation_identifier": { "index": 2 },
      "type": "FEE",
      "account": { "address": "907ff6c714a545110b42982b72aa39c5b7742d610e234a9d40bf8cf624e7a70d" },
      "amount": {
        "value": "-10000",
        "currency": { "symbol": "ICP", "decimals": 8 }
      }
    },
    {
      "operation_identifier": { "index": 3 },
      "type": "STAKE",
      "account": { "address": "907ff6c714a545110b42982b72aa39c5b7742d610e234a9d40bf8cf624e7a70d" },
      "metadata": {
        "neuron_index": 0
      }
    }
  ]
}
```

**响应**：

``` json
{
  "transaction_identifier": {
    "hash": "2f23fd8cca835af21f3ac375bac601f97ead75f2e79143bdf71fe2c4be043e8f"
  },
  "metadata": {
    "operations": [
      {
        "operation_identifier": { "index": 0 },
        "type": "TRANSACTION",
        "status": "COMPLETED",
        "account": { "address": "907ff6c714a545110b42982b72aa39c5b7742d610e234a9d40bf8cf624e7a70d" },
        "amount": {
          "value": "-100000000",
          "currency": { "symbol": "ICP", "decimals": 8 }
        }
      },
      {
        "operation_identifier": { "index": 1 },
        "type": "TRANSACTION",
        "status": "COMPLETED",
        "account": { "address": "531b163cd9d6c1d88f867bdf16f1ede020be7bcd928d746f92fbf7e797c5526a" },
        "amount": {
          "value": "100000000",
          "currency": { "symbol": "ICP", "decimals": 8 }
        }
      },
      {
        "operation_identifier": { "index": 2 },
        "type": "FEE",
        "status": "COMPLETED",
        "account": { "address": "907ff6c714a545110b42982b72aa39c5b7742d610e234a9d40bf8cf624e7a70d" },
        "amount": {
          "value": "-10000",
          "currency": { "symbol": "ICP", "decimals": 8 }
        }
      },
      {
        "operation_identifier": { "index": 3 },
        "type": "STAKE",
        "status": "COMPLETED",
        "account": { "address": "907ff6c714a545110b42982b72aa39c5b7742d610e234a9d40bf8cf624e7a70d" },
        "metadata": {
          "neuron_index": 0
        }
      }
    ]
  }
}
```

## 管理神经元（Managing neurons）

-----

### 设置溶解时间戳（Setting dissolve timestamp）

|开始提供的版本|1.1.0|
|-----|----|
|是否幂等|是|
|所需的最小权限|控制者|

此操作更新神经元可以达到`DISSOLVED`状态的时间。

溶解时间戳总是单调增加。

- 如果神经元处于`DISSOLVING`状态，则此操作可以将溶解时间戳移到更远的将来。
- 如果神经元处于`NOT_DISSOLVING`状态，使用时间`T`调用`SET_DISSOLVE_TIMIESTAMP`接口会尝试将神经元的溶解延迟（溶解神经元所需的最短时间）增加到`T-current_time`。
- 如果神经元处于`DISSOLVED`状态，调用`SET_DISSOLVE_TIMESTAMP`会将其移至`NOT_DISSOLVING`状态并相应地设置溶解延迟。

前提条件：

- `account.address`是神经元控制器的账本地址。

例子：

``` json
{
  "operation_identifier": { "index": 4 },
  "type": "SET_DISSOLVE_TIMESTAMP",
  "account": {
    "address": "907ff6c714a545110b42982b72aa39c5b7742d610e234a9d40bf8cf624e7a70d"
  },
  "metadata": {
    "neuron_index": 0,
    "dissolve_time_utc_seconds": 1879939507
  }
}
```

### 开始溶解（Start dissolving）

|开始提供的版本|1.1.0|
|-------|------|
|是否幂等|是|
|最小所需的权限|控制者|

`START_DISSOLVING`操作将神经元的状态更改为`DISSOLVING`。

前提条件：

- `account.address`是神经元控制者在`ledger`罐头中的地址。

后置条件：

- 神经元处于`DISSOLVING`状态。

例子：

``` json
{
  "operation_identifier": { "index": 5 },
  "type": "START_DISSOLVING",
  "account": {
    "address": "907ff6c714a545110b42982b72aa39c5b7742d610e234a9d40bf8cf624e7a70d"
  },
  "metadata": {
    "neuron_index": 0
  }
}
```

### 停止溶解（Stop dissolving）

|开始提供的版本|1.1.0|
|------|-----|
|是否幂等|是|
|最小权限要求|控制者|

`STOP_DISSOLVING`操作将神经元的状态更改为`NOT_DISSOLVING`。

前提条件：

- `account.address`是神经元控制者在`ledger`罐头中的地址。

后置条件：

- 神经元处于`NOT_DISSOLVING`状态。

例子：

``` json
{
  "operation_identifier": { "index": 6 },
  "type": "STOP_DISSOLVING",
  "account": {
    "address": "907ff6c714a545110b42982b72aa39c5b7742d610e234a9d40bf8cf624e7a70d"
  },
  "metadata": {
    "neuron_index": 0
  }
}
```

### 添加热键（Add hotkeys）

|开始提供的版本|1.2.0|
|------|-----|
|是否幂等|是|
|最小所需权限|控制者|

`ADD_HOTKEY`操作为神经元添加一个热键。治理罐头允许使用热键而不是控制器的密钥来签署一些非关键操作（例如，投票和查询成熟度）。

前提条件：

- `account.address`是神经元控制者在`ledger`罐头中的地址。
- 神经元的热键小于`10`个。

该命令有两种形式：一种形式接受`IC`主体作为热键，另一种形式接受公钥。

**添加一个主体`id`作为热键**：

``` json
{
  "operation_identifier": { "index": 0 },
  "type": "ADD_HOTKEY",
  "account": { "address": "907ff6c714a545110b42982b72aa39c5b7742d610e234a9d40bf8cf624e7a70d" },
  "metadata": {
    "neuron_index": 0,
    "principal": "sp3em-jkiyw-tospm-2huim-jor4p-et4s7-ay35f-q7tnm-hi4k2-pyicb-xae"
  }
}
```

**添加一个公钥作为热键**：

``` json
{
  "operation_identifier": { "index": 0 },
  "type": "ADD_HOTKEY",
  "account": { "address": "907ff6c714a545110b42982b72aa39c5b7742d610e234a9d40bf8cf624e7a70d" },
  "metadata": {
    "neuron_index": 0,
    "public_key": {
      "hex_bytes":  "1b400d60aaf34eaf6dcbab9bba46001a23497886cf11066f7846933d30e5ad3f",
      "curve_type": "edwards25519"
    }
  }
}
```

### 繁殖神经元（Spawn neurons）

|开始提供的版本|1.3.0|
|------|------|
|是否幂等|是|
|最小所需的权限|控制者|

`SPAWN`操作从具有足够成熟度的现有神经元创建一个新的神经元。此操作将所有成熟度从现有的神经元转移到新生成的神经元。

前提条件：

- `account.address`是神经元控制者在`ledger`罐头中的地址。
- 父神经元至少有`1`个`ICP`代币的成熟度。

后置条件：

- 父神经元的成熟度被设置为`0`。
- 产生一个新的神经元，余额等于转移的成熟度。

``` json
{
  "operation_identifier": { "index": 0 },
  "type": "SPAWN",
  "account": { "address": "907ff6c714a545110b42982b72aa39c5b7742d610e234a9d40bf8cf624e7a70d" },
  "metadata": {
    "neuron_index": 0,
    "controller": "sp3em-jkiyw-tospm-2huim-jor4p-et4s7-ay35f-q7tnm-hi4k2-pyicb-xae",
    "spawned_neuron_index": 1
  }
}
```

> 注意，控制器元数据字段是可选的，默认情况下等于现有的神经元控制器。`spawned_neuron_index`元数据字段是必须的。`Rosetta`节点使用此索引来计算生成神经元的子账户。所有生成的神经元必须具有不同的`spawned_neuron_index`值。

## 合并神经元成熟度（Merge neuron maturity）

|最小的提供版本|1.4.0|
|-------|------|
|是否幂等|否|
|最小权限需求|控制者|

`MERGE_MATURITY`操作将神经元的现有成熟度合并到其权益中。可以指定要合并的成熟度百分比，否则合并整个成熟度。

前提条件：

- `account.address`是神经元控制者在`ledger`罐头中的账户地址。
- 该神经元的成熟度不等于`0`。

后置条件：

- 成熟度因合并的数量而减少。
- 神经元质押的数量因成熟度的合并而增加。

例子：

``` json
{
  "operation_identifier": { "index": 0 },
  "type": "MERGE_MATURITY",
  "account": { "address": "907ff6c714a545110b42982b72aa39c5b7742d610e234a9d40bf8cf624e7a70d" },
  "metadata": {
    "neuron_index": 0,
    "percentage_to_merge": 14
  }
}
```

> 注意，`percent_to_merge`元数据字段是可选的，默认情况下等于`100`。如果指定，该值必须是`1`到`100`之间的整数（包括边界）。

## 访问神经元属性（Accessing neuron attributes）

|最小的提供的版本|1.3.0|
|-------|-----|
|最小所需的权限|公开|

调用`/account/balance`接口以访问质押金额和公开可用的神经元数据。

前提条件：

- `public_key`包含神经元控制者的公钥。

> 注意，此操作仅在在线模式下可用。请求不应指定任何块标识符，因为接口总是返回神经元的最新状态。

**请求**：

``` json
{
  "network_identifier": {
    "blockchain": "Internet Computer",
    "network": "00000000000000020101"
  },
  "account_identifier": {
    "address": "a4ac33c6a25a102756e3aac64fe9d3267dbef25392d031cfb3d2185dba93b4c4"
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
```

**响应**：

``` json
{
  "block_identifier": {
    "index": 1150,
    "hash": "ca02e34bafa2f58b18a66073deb5f389271ee74bd59a024f9f7b176a890039b2"
  },
  "balances": [
    {
      "value": "100000000",
      "currency": {
        "symbol": "ICP",
        "decimals": 8
      }
    }
  ],
  "metadata": {
    "verified_query": false,
    "retrieved_at_timestamp_seconds": 1639670156,
    "state": "DISSOLVING",
    "age_seconds": 0,
    "dissolve_delay_seconds": 240269355,
    "voting_power": 195170955,
    "created_timestamp_seconds": 1638802541
  }
}
```