# `ledger`罐头（The ledger canister）

本文档是`ledger`罐头的公共接口规范。它提供了功能的概述，详细说明了一些内部方面，并记录了公开可用的方法。我们还提供了一个抽象的数学模型，它可以精确地确定罐头实现的方法的预期行为，尽管抽象级别有点高。

> 部分罐头接口仅供内部使用，因此不属于本规范。但是，只要相关，我们也会对这些方面提供一些见解。

## 概述和术语（Overview & terminology）

简而言之，`ledger`罐头维护者一组`IC`委托人拥有的账户；每个账户都关联了一个代币余额。账户所有者可以将代币从他们控制的账户转移到任何其它`ledger`账户。所有转账操作都记录在一个追加的交易罐头中。

`ledger`罐头的接口还允许铸造和烧毁代币，这是记录在交易罐头上的附加交易。

### 代币（Tokens）

### 账户（Accounts）

### 操作、交易、区块、交易罐头（Operations, transactions, blocks, transaction ledger）

## 方法（Methods）

----

###  转移代币（Transferring tokens）

### 链接`ledger`区块（Chaining ledger blocks）

### 烧毁和铸造代币（Burning and minting Tokens）

### 余额（Balance）

## 语义（Semantics）

----

### 基本类型（Basic types）

### `ledger`状态（Ledger State）

### 余额（Balances）

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