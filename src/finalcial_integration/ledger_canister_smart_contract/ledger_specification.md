# `ledger`罐头（The ledger canister）

本文档是`ledger`罐头的公共接口规范。它提供了功能的概述，详细说明了一些内部方面，并记录了公开可用的方法。我们还提供了一个抽象的数学模型，它可以精确地确定罐头实现的方法的预期行为，尽管抽象级别有点高。

> 部分罐头接口仅供内部使用，因此不属于本规范。但是，只要相关，我们也会对这些方面提供一些见解。

## 概述和术语（Overview & terminology）

简而言之，`ledger`罐头维护者一组`IC`委托人拥有的账户；每个账户都关联了一个代币余额。账户所有者可以将代币从他们控制的账户转移到任何其它`ledger`账户。所有转账操作都记录在一个追加的交易罐头中。

`ledger`罐头的接口还允许铸造和烧毁代币，这是记录在交易罐头上的附加交易。

### 代币（Tokens）

`IC`系统可以同时有多种代币。但治理`IC`所用的代币是`ICP`代币。代币的最小不可分割单位是`e8s`：一个`e8`是`10^-8`个代币。

### 账户（Accounts）

`ledger`罐头跟踪账户的：

- 归属或是被该`IC`主体控制的所有账户。
- 每个账户仅有一个所有者（即没有“联名控制者”）。
- 委托人可以控制多个账户。我们通过（`32`字节字符串）`subaccount_identifier`区分同一委托人的不同账户。因此，从逻辑上讲，每个`ledger`账户对应一对`(account_owner, subaccount_identifier)`。
- 这样一对对应的账户标识符是一个`32`字节的字符串，计算如下：
  ``` unknwon
  account_identifier(principal,subaccount_identifier) = CRC32(h) || h

  where

  h = sha224(“\x0Aaccount-id” || principal || subaccount_identifier)
  ```
  即，主体和子账户标识符对应的账户标识符（或有时简称为账户）是通过使用`sha224`哈希获得域分隔符`\x0Account-id`、主体和子账户标识符的串联，并在前面个加上结果哈希值的（大端表示）`CRC32`。域分隔符由一个字符串（此处为`account-id`）组成，前面有一个等于字符串长度的字节（此处为`\x0A`）。
  - 对于任何的`principal`，我们将全`0`的`subaccount-identifier`对应的账户作为该`principal`的默认账户。
  - 治理罐头的默认账户在铸造、烧毁代币中起着重要作用（见下文），我们将其称为`minting_account_id`。

### 操作、交易、区块、交易罐头（Operations, transactions, blocks, transaction ledger）

账户余额因以下三种操作之一而发生变化：将代币从一个账户发送到另一个账户、将代币铸造到一个账户、从一个账户烧毁代币。每个操作都由对`ledger`罐头的调用触发并被记录为一个事务：除了操作的详细信息外，事务还包含一个`memo`字段（一个`64`位数字）和一个指示事务发生时间的时间戳字段。

每个事务都包含在一个块中（每个块只有一个事务），并且通过在每个块中包含前一个块的哈希来“链接”块（块如何序列化的详细信息如下）。区块链在账本中的位置称为区块索引（或区块高度）；从`0`开始计数。

下面在`candid`语法中指定了用于表示这些概念的类型。

**基本数据类型**：

``` candid
type Tokens = record {
     e8s : nat64;
};

// Account identifier  is a 32-byte array.
// The first 4 bytes is big-endian encoding of a CRC32 checksum of the last 28 bytes
type AccountIdentifier = blob;

//There are three types of operations: minting tokens, burning tokens & transferring tokens
type Transfer = variant {
    Mint: record {
        to: AccountIdentifier;
        amount: Tokens;
    };
    Burn: record {
         from: AccountIdentifier;
         amount: Tokens;
   };
    Send: record {
        from: AccountIdentifier;
        to: AccountIdentifier;
        amount: Tokens;
    };
};

type Memo = u64;

// Timestamps are represented as nanoseconds from the UNIX epoch in UTC timezone
type TimeStamp = record {
    timestamp_nanos: nat64;
};

Transaction = record {
    transfer: Transfer;
    memo: Memo;
    created_at_time: Timestamp;
};

Block = record {
    parent_hash: Hash;
    transaction: Transaction;
    timestamp: Timestamp;
};

type BlockIndex = nat64;

//The ledger is a list of blocks
type Ledger = vec Block
```

## 方法（Methods）

----

`ledger`罐头实现的方法用来：

- 从一个账户转移代币到另一个账户。
- 获取某个账户的余额。

###  转移代币（Transferring tokens）

账户的所有者可以使用`transfer`方法将代币从该账户转移到任何其它账户。该方法输入如下：

- `amount`：待转移的代币的数量。
- `fee`：执行转移操作的费用。
- `from_subaccount`：一个子账户标识符，它指定`ICP`应该从调用者的哪个账户发生。这个参数是可选的————如果调用者没有指定，则设置为全`0`向量。
- `to`：代币被转移到的账户的标识符。
- `memo`：发送者选择的`64`位数字；它可以以多种方式使用，例如用来确定某个具体的代币转移交易。
- `created_at_time`：指示调用者何时创建事务的时间戳————如果调用者未指定，则将其设置为当前`IC`的系统时间。

`ledger`罐头按照如下方式执行代币转移函数：

- 检查目标地址的账户标识符是否正确（的格式）。
- 检查事务是足够新（已在过去`24`小时内创建）并且不是“在未来”（即，它检查`created_at_time`是否在未来超过由参数指定的允许时间漂移在`IC`系统中，当前设置为`60`秒）。
- 计算源账户（使用调用`principal1`和`from_subaccount`）并检查它是否持有超过`amount+fee`数量的代币。
- 检查`fee`是否与`standard_fee`匹配（目前，`standard_fee`是一个常数，大小是`10^-4`，见下文例外）
- 检查过去`24`小时内没有提交相同的交易。
- 如果任何检查失败，则返回相应的错误。
- 否则它
  - 从源账户中减去`amount`+`fee`的代币。
  - 将`amount`数量的代币添加到目标账户。
  - 将交易`(Transfer(from, to, amount, fee), memo, created_at_time)`添加到`ledger`：
    - 它创建一个包含交易的块，将块中的`parent_hash`设置为`last_hash`（本质上是账本中最后一个块的哈希），并将块中的时间戳设置为`IC`的系统时间戳；
    - 它将`last_hash`计算为新创建的块的编码的哈希值（有关如何计算编码，请参见下文）；
    - 它将块附加到`ledger`罐头并返回其高度。

### 链接`ledger`区块（Chaining ledger blocks）

如上所述，`ledger`中包含的块是链式的（通过在块中包含前一个块的哈希）。这可以通过仅签署其最后一个块来验证整个`ledger`罐头的账本。

在本节中，我们通过指定块在散列之前如何序列化来描述区块链接的细节。

从高层次上来说，该块使用`protobuf`进行序列化。但是，由于`protobuf`编码不一定是确定性的（也不能保证保持固定），因此我们提供了使用的特定编码，保证不会改变。

下面的定义是递归的。它用`.`表示字符串的连接，还有两个在这里没有定义但已经很成熟的函数：我们写`len(x)`来表示字符串`x`的长度。我们还写了`varint(s)`用于整数`s`的可变长度编码。这个函数的精确定义可以在[`protobuf`文档](https://developers.google.com/protocol-buffers/docs/encoding#varints)中找到。

``` candid
encoded_block(Block{parent_hash, timestamp, transaction}) :=
    let encoded_transaction = encode_transaction(transaction)
    in encode_hash(parent_hash) .
       12 0a 08 . varint(timestamp) .
       1a . len(encoded_transaction) . encoded_transaction

encode_hash(Nil) := Nil
encode_hash(hash) := 0a 22 0a 20 . hash

encode_transaction(Transaction{operation, memo, created_at_time}) :=
    let encoded_operation = encode_operation(operation)
        encoded_memo = encode_memo(memo)
        encoded_timestamp = encode_timestamp(created_at_time)
    in encoded_operation .
       22 . len(encoded_memo) . encoded_memo .
       32 . len(encoded_timestamp) . encoded_timestamp

encode_memo(Nil) := Nil
encode_memo(Memo{memo}) := 08 . varint(memo)

encode_timestamp(Timestamp{timestamp_nanos}) := 08. varint(timestamp_nanos)

encode_operation(Burn{AccountIdentifier{from}, Tokens{amount}}) :=
    // identifiers can be 28 or 32 bytes (4 bytes checksum + 28 bytes hash)
    let encoded_account_identifier = 0a . len(from) . varint(from)
        encoded_amount = 08 . varint(amount)
        encoded_burn = 0a . len(encoded_account_identifier) . encoded_account_identifier .
                       1a . len(encoded_amount) . encoded_amount
    in 0a . len(encoded_burn) . encoded_burn

encode_operation(Mint{AccountIdentifier{to}, Tokens{amount}}) :=
    // identifiers can be 28 or 32 bytes (4 bytes checksum + 28 bytes hash)
    let encoded_account_identifier = 0a . len(to) . to
        encoded_amount = 08 . varint(amount)
        encoded_mint = 12 . len(encoded_account_identifier) . encoded_account_identifier .
                       1a . len(encoded_amount) . encoded_amount
    in 12 . len(encoded_mint) . encoded_mint

encode_operation(Transfer{AccountIdentifier{from},
                           AccountIdentifier{to},
                           Tokens{amount},
                           Tokens{fee}}) :=
    let encoded_from = 0a . len(from) . from
        encoded_to = 0a . len(to) . to
        encoded_amount = 08 . varint(amount)
        encoded_fee = 08 . varint(fee)
        encoded_transfer = 0a . len(encoded_from) . encoded_from .
                           12 . len(encoded_to) . encoded_to .
                           1a . len(encoded_amount) . encoded_amount .
                           22 . len(encoded_fee) . encoded_fee
    in 1a . len(encoded_transfer) . encoded_transfer
```

### 烧毁和铸造代币（Burning and minting Tokens）

典型的转账将代币从一个账户转移到另一个账户。一个重要的例外是当转账的来源或目的地是特殊的`minting_account_id`时，转移到铸币账户的效果是，代币只是从源账户中移除，而不是存放在任何地方；代币被烧毁。烧毁交易在`ledger`罐头中被记录为`(Burn(from, amount))`。重要的是，烧毁转账的费用为`0`，但要烧毁的代币数量必须超过`standard_fee`。

从`minting_account_id`账户转账的效果是简单地将代币添加到目标账户；代币是铸造地。调用时，交易`(Mint(to, amount))`被添加到`ledger`罐头中。请注意，`minting_account_id`由治理罐头控制，这使得铸币成为一个特权操作，仅适用于该罐头。

下面的`candid`表示了`transfer`方法的签名和需要的参数：

``` candid
// Arguments for the `transfer` call.
type TransferArgs = record {
    memo: Memo;
    amount: Tokens;
    fee: Tokens;
    from_subaccount: opt SubAccount;
    to: AccountIdentifier;
    created_at_time: opt TimeStamp;
};

type TransferError = variant {
    // The fee specified in the send request was not the one the ledger expects.
    BadFee : record { expected_fee : Tokens; };
    // The sender's (sub)account doesn't have enough funds for completing the transaction. Return an error with the debit account balance.
    InsufficientFunds : record { balance: Tokens; };
    // The transaction is too old, the ledger only accepts transactions created within 24 hours window. Return an error with the allowed time-window size in nanoseconds.
    TxTooOld : record { allowed_window_nanos: nat64 };
    // `created_at_time` is in future.
    TxCreatedInFuture : null;
    // The transaction was already submitted before.
    TxDuplicate : record { duplicate_of: BlockIndex; }
};

type TransferResult = variant {
    Ok : BlockIndex;
    Err : TransferError;
};


service : {
  transfer : (TransferArgs) -> (TransferResult);
}
```

### 余额（Balance）

`ledger`罐头以一种很自然的方式跟踪所有账号的余额（有关更正式的定义，请参阅下面的语义部分）。

任何委托人都可以通过`account_balance`方法获取任意账户的余额：入参为账户标识；结果是与账户相关联的余额。账户标识符`minting_account_id`的账户余额始终为`0`；任何其它账户的余额都是显而易见的方式计算。

``` candid
type AccountBalanceArgs = record {
    account: AccountIdentifier;
};

service : {
  // Get the amount of ICP on the specified account.
  account_balance : (AccountBalanceArgs) -> (Tokens) query;
}
```

## 语义（Semantics）

----

在本节中，我们将提供`ledger`的公共方法的语义。我们使用了一些特别的数学符号，我们保持接近上面介绍的符号。我们使用`·`来表示列表`list`的连接。我们为全`0`向量编写`default_subaccount`。如果`L`是一个列表`list`，那么我们将`|L|`记作`L`的长度，`L[i]`记作`L`的第`i`个元素。`L`的第一个元素是`L[0]`。

### 基本类型（Basic types）

``` candid
Operation =
  Transfer = {
    from: AccountIdentifier;
    to: AccountIdentifier;
    amount: Tokens;
    fee: Tokens;
  } |
  Mint = {
    to: AccountIdentifier;
    amount: Tokens;
  } |
  Burn = {
    from: AccountIdentifier;
    amount: Tokens;
  }
}

Block = {
   operation: Operation;
   memo: Memo;
   created_at_time: Timestamp;
   hash: Hash;
  }

Ledger = List(Block)
```

### `ledger`状态（Ledger State）

`ledger`罐头的状态包括：

- 交易账本（包含交易的区块链列表）；
- 全局变量；
  - `last_hash`：可选变量，记录账本中最后一个区块的哈希值；如果`ledger`罐头中不存在任何块，则将其设置为`None`；
    ``` candid
    State = {
      ledger: Ledger;
      last_hash: Hash | None;
    };
    ```
    初始化的时候，账本设置为空列表，此时`last_hash`为`None`；
    ``` motoko
    {
      ledger = [];
      last_hash = None;
    }
    ```

### 余额（Balances）

给定一个`ledger`罐头，我们定义`balance`函数为`ledger`账户中的代币余额：

``` candid
balance: Ledger x AccountIdentifier -> Nat
```

该函数以递归的方式定义：

``` candid
balance([],account_id) = 0

if (B = Block{Transfer{from,to,amount, fee}, memo, time, hash}) and (to = account_id)) |
   (B = Block{Mint{to, amount}, memo, time}) and (to = account_id)) then
   then
   balance(OlderBlocks · [B] , account_id) = balance(OlderBlocks, account_id) + amount,

if (B = Block{Transfer{from,to,amount,fee},memo,time}} and (from = account_id)
    then
    balance(OlderBlocks · [B], account_id) = balance(OlderBlocks,account_id) - (amount+fee)

if (B = Block{Burn{from,amount}) and (from = account_id)
   then
   balance(OlderBlocks · [B], account_id) = balance(OlderBlocks,account_id) - amount

otherwise
  balance(OlderBlocks · [B], account_id) = balance(OlderBlocks, account_id)
```

我们将账本方法的语义描述为一个函数，该函数将账本状态、调用参数作为输入并返回（可能）新状态和回复。在功能描述中，我们使用了一些反映系统提供信息的附加功能。其中包括`caller()`返回调用该方法的主体，`now()`返回`IC`系统的时间并漂移一个常数，指明`IC`和外部时间之间允许的时间漂移。我们还为`bool`函数编写了`well_formed(.)`，该函数检查其输入是否为格式良好的账户标识符（即前四个字节等于其余`28`个字节的`CRC32`）。

### `ledger`方法：`transfer`（Ledger Method: transfer）

**状态和参数**：

``` candid
S
A = {
  memo: Memo;
  amount: Tokens;
  fee: Tokens;
  from_subaccount: opt SubAccount;
  to: AccountIdentifier;
  created_at_time: opt TimeStamp;
  }
```

**返回结果和回复**：

``` candid
output (S',R) calculated as follows:

if created_at_time = None then created_at_time = now();
if timestamp > now() + drift then (S',R) = (S, Err);
if now() - timestamp > 24h then (S',R) = (S, Err);
if not(well_formed(to)) then (S',R) = (S, Err);

if to = `minting_account_id` and (fee ≠ 0 or amount < standard_fee) then (S',R) = (S, Err);

if from_subaccount = None then from_subaccount = default_subaccount;
from = account_identifier(caller(),from_subaccount)

 if from = `minting_account_id' then B = Block{Mint{to, amount}, memo, timestamp, S.last_hash}
      else
        if to = `minting_account_id` then B = Block{Burn{from, amount}, memo, timestamp, S.last_hash}
            else B = Block{Transfer{from, to, amount, fee}, memo, timestamp, S.last_hash};
  if exists i (ledger[i].operation, ledger[i].memo, ledger[i].timestamp) = (B.operation,B.memo,B.timestamp) then (S',R)=(S,Err)
  else
    (S'.ledger = [B] · S.ledger);
    (S'.lasthash = hash(B));
     R = |S'.ledger|-1;
```

### `ledger`方法：`balance_of`（Ledger Method: balance_of）

**状态和参数**：

``` candid
S
A = {
    account_id: AccountIdentifier
}
```

**返回结果和回复**：

``` candid
output (S',R) calculated as follows

S' = S
if account_id = `minting_account_id`
   then R = 0
   else R = balance(S.ledger,account_id))
```