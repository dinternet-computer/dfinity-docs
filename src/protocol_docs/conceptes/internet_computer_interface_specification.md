# `Internet Computer`接口手册

## 介绍

-----

欢迎来到`Internet Computer`！我们之所以称`Internet Computer`为`Internet Computer`，是因为事实上`Internet Computer`是由无数的实实在在的物理计算机组成的区块链。阅读本节后，你将会看到一个开放的、世界范围内可用的计算机。如果仅仅是开发人员想在`Internet Computer`上开发去中心化的应用或者一般用户想使用`Internet Computer`上的去中心化应用，那可能只需要了解一点点`ICP`协议就行了（`Internet Computer Protocol`）。不过，感兴趣的开发者或任何人，都能一览`Internet Computer`的整个架构的任何细节特性

### 目标用户

本文档将从底层讲解`Internet Computer`的接口以及如何使用这些接口。

> 注意，本节涉及的仅仅是底层接口，可能不适合应用层开发者或者一般的终端用户，想了解`SDK`，`Motoko`可以[点击这里](https://sdk.dfinity.org/)。

目标观众：

- 使用底层接口的开发者（例如需要开发`agent`，`canister developments kits`，模拟器和其它基础工具）。
- 实现底层接口的开发者（例如`Internet Computer`的开发者）。
- 希望完全了解`Internet Computer`的人（例如对`Internet Computer`做安全分析）。

> 注意，本文档是严格的、技术性的文档，它不是`Internet Computer`的介绍文档，如果有需要，请先阅读`Internet Computer`的高级文档。

### 本文档涉及的范围

如果你能够将`Internet Computer`视做一个执行`WebAssembly`的分布式引擎，那本文档就是讲述如何运行这些`Webassembly`的。为了文档的可扩展性，本节不会涉及共识协议、节点、子网、正交一致性和治理。

本文档尝试达到与实现无关：它能帮助你实现一个与`Internet Computer Protocol`兼容的新的`Internet Computer`。这意味着本文档不包含与运行`Internet Computer`相关的接口（例如数据中心运营商、协议开发者、治理用户），因为节点的更新、监控、日志记录等话题与实际实施及其架构有着内在的紧要联系。

### `Internet Computer`的整体架构

`Internet Computer`（缩写为`IC`）上的去中心化应用`Dapp`，是通过罐头智能合约（`canister smart contracts`，缩写为`canister`，又叫罐头）实现的。如果你想在`IC`上部署一个智能合约，首先需要创建一个包含`WebAssembly`代码和配置文件的罐头`canister`，然后通过[Http 接口](https://smartcontracts.org/docs/interface-spec/index.html#http-interface)将它部署到`IC`网络上。最方便的方式就是通过`Motoko`语言开发`canister`然后通过`dfx`直接部署到`IC`；当然你也能够制作并使用自己的工具来开发和部署`canister`，[canister模块是怎样的](https://smartcontracts.org/docs/interface-spec/index.html#canister-module-format)和[为什么WebAssembly代码能和IC交互](https://smartcontracts.org/docs/interface-spec/index.html#system-api)这两节讲述了如何做。

一旦你的`canister`成功部署到`IC`主网，那它就是一个智能合约了。其它想用它的用户，能通过[Http接口](https://smartcontracts.org/docs/interface-spec/index.html#http-interface)按照[System API](https://smartcontracts.org/docs/interface-spec/index.html#system-api)里面指定的方式与这个`canister`交互。

用户能过通过`Http`接口进行一种称作`query`的查询调用，这种调用非常快速，不能它不能改变`canister`的内部状态。

![img](../../assets/images/a.svg "img")

### 命名法

为了达成一致，我们定义一下词汇：

我们避免使用`client`，因为`client`既可以是使用`Internet Computer`的`client`，也可以是组成`Internet Computer`的`client`，所以我们使用`user`来表示使用`Internet Computer`的外部用户，并且在许多情况下（一些代码中）它被叫做`agent`来帮助用户与`IC`交互。

罐头的公共入口称作`method`，`method`有两种类型。一是`update`类型的`method`，它能改变`canister`的内部状态；二是`query`类型的`method`，它不能改变`canister`的内部状态，并且没有远程调用。

`method`是由`caller`到`callee`被调用的，通常产生一个包含`reply`或者`reject`的`response`。`method`可以包含参数。

外部的`caller`可以发起一个`update`调用，`update`调用可以可以调用`update`或`query`类型的`method`；而如果外部`caller`发起的是`query`调用，那就只能调用`query`类型的`method`。罐头内的调用可以调用两种类型的`method`。记住，从罐头`A`到罐头`B`的调用也是罐头`B`的内部调用。

在内部，一个调用`call`或者`response`被视作从发送者`sender`到接收者`receiver`的消息。消息是没有回复的。

`WebAssembly`函数是由`WebAssembly`模块或者`System API`提供的。这些函数的调用可能被捕获（发生错误）也可能返回（能够带有返回值）。函数能够接收参数。

外部`user`通过`Http`接口提交请求到`IC`。请求可能被接受也可能被拒绝，一些请求也可能创建内部消息。

`canister`和`user`都被`principal`标识区分，有时候又叫做`id`。

## 共同的概念

-----

在详细介绍四大部分接口之前（即`agent`使用的[Http接口](https://smartcontracts.org/docs/interface-spec/index.html#http-interface)，`canister`使用的[System API](https://smartcontracts.org/docs/interface-spec/index.html#system-api)、[virtual Management canister](https://smartcontracts.org/docs/interface-spec/index.html#ic-management-canister)、[System State Tree](https://smartcontracts.org/docs/interface-spec/index.html#state-tree)），本节讲述这些接口之前的相同部分。

### 缺省常量和限制

本规范可能会应用某些常量和限制，但尚未指定它们的具体值，即使它们的实现是已经定义好的。许多资源的限制仅与指定`IC`的错误处理行为相关（如上所述，本文档中还没有准确描述）。此列表还不完整：

- `MAX_CYCLES_PER_MESSAGE`：在`canister`执行某条消息之前，`canister`内必须具有的最小的`Cycles`，在消息执行之前从`canister`余额扣除。详情查看[Message执行](https://smartcontracts.org/docs/interface-spec/index.html#rule-message-execution)。
- `MAX_CYCLES_PER_RESPONSE`：在`canister`执行调用时，预留出来的最小`Cycles`，在调用执行之后从`canister`余额扣除，用于处理`response`，多余的`Cycles`会退还。详情查看[Message执行](https://smartcontracts.org/docs/interface-spec/index.html#rule-message-execution)。
- `MAX_CANISTER_BALANCE`：一个`canister`最大的`Cycles`余额，大于2^128的值会被丢弃。

### Principal

`principal`是`canister`、`user`和其它未来可能的概念的通用标识符。就`IC`的大多数用途而言，`principal`是长度从`0`到`29`字节的不透明的二进制`blob`，没有什么特别的机制区分`canister`和`user`的`principal`。

然而，这些`principal`有一些结构来编码特定的身份验证和授权行为。

#### 

#### Principal 的文本表示

### Canister 生命周期

#### Canister 的 Cycles

#### Canister 的状态

### 签名

#### Ed25519 和椭圆曲线签名

### Web Authentication

### Canister 签名

## 系统状态数（System state tree）

-----

### 时间（Time）

### 子网信息（Subnet information）

### 请求状态（Request status）

### 已签证的数据（Certified data）

### 罐头信息（Canister information）

## HTTPS 接口（HTTPS interface）

------

### 罐头调用的整体流程（Overview of canister calling）

### 调用请求（Request: call）

### 读状态请求（Request: Read state）

### 查询请求（Request: Query call）

### 有效的罐头`id`（Effective canister id）

### 认证（Authentication）

### 结构化数据的表观`hash`（Representation-independent hashing of structured data）

### 请求`ids`（Request ids）

### 拒绝码（Reject codes）

### 状态接口（Status endpoint）

### `CBOR`编码的请求和响应（CBOR encoding of requests and responses）

### `CDDL`描述语言描述的请求和响应（CDDL description of requests and responses）

### 顺序保证（Ordering guarantees）

### 跨节点同步（Synchronicity acriss nodes）

## 罐头的组织格式（Canister module format）

-----

## 罐头的接口（System API）

----- 

### WASM 模块要求（WebAssembly module requirements）

### 数字表示（Interpretation of numbers）

### 入口点（Entry points）

### 罐头初始化（Canister initialization）

### 罐头升级（Canister upgrades）

### 公共方法（Public methods）

### 心跳（Heartbeat）

### 回调（Callbacks）

### 导入预览（Overview of imports）

### `Blob`类型的参数和结果（Blob-typed arguments and results）

### 方法参数（Method arguments）

### 响应（Reponding）

### 入口消息检查（Ingress message inspection）

### 罐头状态（Canister status）

### Cycles

### 持久性内存（Stable memory）

### 系统时间（System time）

### 已签证的数据（Certified data）

### 调试助手（Debugging aids）


### 主机的使用指引（Using Host References）

## 管理罐头（The IC management canister）

----- 

### 接口预览（Interface overview）

### 创建罐头（IC method：create_canister）

### 更新设置（IC method：update_settings）

### 安装代码（IC method：install_code）

### 卸载代码（IC method：uninstall_code）

### 查询罐头状态（IC method：canister_status）

### 停止罐头（IC method：stop_canister）

### 打开罐头（IC method：start_canister）

### 删除罐头（IC method：delete_canister）

### 充值`Cycles`（IC method：deposit_cycles）

### 随机数（IC method：raw_rand）