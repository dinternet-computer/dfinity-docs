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

####  特殊形式的 Pricipal

在本段，`H`表示`SHA-224`，`·`表示二进制连接符，`|p|`表示`p`的字节长度。

`identity`可以分为下面几类：

1. 不透明`identity`：
    由`IC`自动生成。
    > 注意，通常情况下这类`identity`有`0x01`结尾，不过一般用户不用理会这类`identity`。
2. 自认证的`identity`：
    格式：`H(public_key) · 0x02`（一共29个字节）。
3. 派生`identity`：
    格式：`H(|registering_pricipal| · registering_pricipal · derivation_nonce) · 0x03`（一共29字节）。
    当需要注册一个`identity`时，这些`identity`就会被特殊处理。无论谁注册一个`identity`都需要提供一个`derivation_nonce`，用这个`derivation_nonce`和注册者的`identity`一起`hash`，这样，每个用户就都有一个属于自己的`identity`空间。
    > 注意，派生`identity`目前暂时未在本文档中明确使用，但它们可能会在内部或将来使用。
4. 匿名`identity`：
    以`0x04`结尾，用作匿名调用者的`identity`，用类`identity`能够免除签名来调用`IC`接口。
    当`IC`创建一个新的`identity`（`fresh identity`）时，它永远不会创建一个自我验证的`identity`、一个匿名的`identity`或一个从罐头或用户派生的`identity`。

#### Principal 的文本表示

> 注意，`IC`接口实际上不是用文本表示的`pricipal`（总是用二进制格式的`principal`）

现在，我们规定一个标准的、文本表示的`principal`————在任何需要的时候使用，比如打印`principal`到终端、显示到网页给用户、输出到日志文件、或者是用在代码中等等。

对于二进制流`b`，它的文本表示是`Grouped(Base32(CRC32(b) · b))`：
- `CRC32`：四字节的纠错码，`ISO 3309`和`ITU-T V.42`标准，详情查看[这里](https://www.w3.org/TR/2003/REC-PNG-20031110/#5CRC-algorithm)
- `Base32`：一种编码方式，[RFC4648](https://tools.ietf.org/html/rfc4648#section-6)标准。
- `Grouped`：一个函数，接收`ASCII`字符串，并且每隔5个字符插入一个`-`但是首尾永远不会有`-`分隔符。

`principal`应该总是用小写字母的形式，但解析的时候不应该是大小写敏感的。

因为`principal`的最大长度是`29`字节，所以`principal`的文本表示最大不超过`63`个字节（`10*5+3` + `10`个分隔符）。

> 注意，一个`identity`为`0xABCD01`的罐头的校验码为`0x233FF206`（[在线计算器](https://crccalc.com/?crc=ABCD01&method=crc32&datatype=hex&outtype=hex)），所以最终文本格式的`identity`应该是`em77e-bvlzu-aq`。

编码和解码`principal`的实例代码（可以直接复制到`bash`执行）：

编码`principal`：

``` bash
function textual_encode() {
  ( echo "$1" | xxd -r -p | /usr/bin/crc32 /dev/stdin; echo -n "$1" ) |
  xxd -r -p | base32 | tr A-Z a-z |
  tr -d = | fold -w5 | paste -sd'-' -
}
```

解码`principal`：

``` bash
function textual_decode() {
  echo -n "$1" | tr -d - | tr a-z A-Z |
  fold -w 8 | xargs -n1 printf '%-8s' | tr ' ' = |
  base32 -d | xxd -p | tr -d '\n' | cut -b9- | tr a-z A-Z
}
```

### Canister 生命周期

`IC`上的去中心化应用又叫做罐头`canister`。概念上来说，一个罐头有下面几种状态：

- 罐头`identity`（是一个`principal`）
- 罐头的控制着`controller`（一个可为空的`principal`数组）
- `Cycles`余额
- 罐头的运行状态，可以是`running`、`stopping`、`stopped`
- 保留资源

一个罐头可以是空的（刚创建的时候），也可以是非空的，一个非空的罐头还有如下状态：

- 以罐头格式组织的代码
- 数据（内存，全局变量等）
- 可能还有一些其它的数据用来实现`IC`（例如队列）

可以通过[安装代码](https://smartcontracts.org/docs/interface-spec/index.html#ic-install_code)使罐头变成未空的状态，已安装代码的罐头也可以卸载代码。

如果一个空的罐头收到响应时，该响应会被丢弃，就好像罐头在处理响应时被捕获一样。

#### Canister 的`Cycles`

`IC`区块链依赖`Cycles`管理它的资源。罐头使用`Cycles`来为它使用的资源付费。

当一个罐头的`Cycles`被消耗到零时，它的状态会变为`deallocated`，这个下面两种情况是一样的效果：

- 卸载这个罐头中的代码（IC method [uninstall_code](https://smartcontracts.org/docs/interface-spec/index.html#ic-uninstall_code)）
- 设置该罐头的所有资源保留为零

在这之后，罐头就是空的。充值之后就能重新安装。

> 注意，一旦某个罐头被清空，它的`identity`、`Cycles`余额、控制者（`controller`）`identity`会被保留在`IC`长达十年，至于十年后会发生什么，我们还没决定好。

#### Canister 的状态

罐头的状态能够用来控制罐头是否去处理进来的请求：

- `running`：正常处理进来的请求
- `stopping`：进来的请求会被`IC`拒绝，但是正在返回的请求正常返回
- `stopped`：进来的请求会被`IC`拒绝，并且此时`canister`没有正在返回的请求

特殊情况：不管[管理员罐头](https://smartcontracts.org/docs/interface-spec/index.html#ic-management-canister)是什么状态，对它的请求都会被处理。

可以在`stop_canister`函数和`start_canister`函数期间初始化罐头的操作者`controller`（查询罐头状态使用`canister_status`函数，罐头自己也能调用这个函数查询罐头的状态）。

注意：此状态与罐头是否为空的问题正交：空罐头可以处于运行状态。但是调用一个空罐头还是会响应一个`reject`。

### 签名

数字签名方案用于验证`IC`基础设施各个部分中的消息。签名是域分隔的（`domain separated`），这意味着每条消息都以一个字节字符串为前缀，该字节字符串对于签名的目的是唯一的。

`IC`支持多种签名方案，下面的章节会详细给出。对于每一种方案，我们都指定在公钥中编码的数据（始终是`DER`编码，并指示要使用的方案）以及签名的形式（对于本规范的其余部分而言，它们是不透明的`blob`）。

在所有情况下，签名的有效负载都是域分割符和消息的串联。本规范中对签名的所有使用都表示域分隔符，以唯一表示签名的目的。域分隔符在构造上是无前缀的，因为它们的第一个字节表示它们的长度。

#### Ed25519 和椭圆曲线签名

本方案支持普通的签名：

- [`Ed25519`](https://ed25519.cr.yp.to/index.html)或者
- `P-256`曲线上的[`ECDSA`](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf)（也叫做`secp256r1`），使用`SHA-256`作为`hash`函数
- 公钥必须对签名方案`Ed25519`或`ECDSA`有效，并且被编码为`DER`
  - 查看[RFC 8410](https://tools.ietf.org/html/rfc8410)了解如果对`Ed25519`公钥进行`DER`编码
  - 查看[RFC 5480](https://tools.ietf.org/rfc/rfc5480)了解如何对`ECDSA`公钥进行`DER`编码；对于曲线`secp256k1`，使用`OID 1.3.132.0.10`，这些点必须以未压缩的格式指定（即`0x04`后跟`x`和`y`的大端`32`字节编码）。
- 签名是以`32`字节大端编码的`r`、`s`连接组成的。

### Web Authentication

`web authentication`允许的签名方案是：

- `P-256`曲线上的`ECDSA`，使用`SHA-256`作为`hash`函数
- [RSA PKCS#1v1.5 (RSASSA-PKCS1-v1_5)](https://datatracker.ietf.org/doc/html/rfc8017#section-8.2)，同样使用`SHA-256`作为`hash`函数

签名是通过使用有效负载作为`web authentication`断言中的挑战来做的。

通过验证挑战字段是否包含有效负载的`basg64url`编码来检查签名，并且该签名在`authenticationatorData · SHA-256(utf8(clientDataJSON))`上进行验证，如[WebAuthen w3c](https://www.w3.org/TR/webauthn/#op-get-assertion)中所指定那样。

- 公钥被编码为`DER`包装的`COSE`密钥。

!! TODO

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