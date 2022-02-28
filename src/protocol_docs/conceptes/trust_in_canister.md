# 罐头里的信任

## 背景（background）

-----

罐头中的`Defi`和相关应用的一个关键方面是转移价值的能力，例如，`ICP`或者比特币。这让非常考验罐头的可信任性。`IC`是如何确保存在罐头中的价值（`token`）是安全的呢？

这个问题的答案有两个不同的维度：

1. 相信罐头会做它应该做的事情，并且
2. 对罐头行为不会意外改变的信心。

## 罐头会做它应该做的事情（The canister does what it is supposted to do）

可以通过两个步骤检查罐头的正确行为。首先，检查用于生成部署在罐头中的`WASM`代码的源代码，以确保它实现了预期或者声明的功能，并且仅实现了此功能。其次，确保罐头运行的`WASM`确实是从声明的源代码生成的。在这里，构建的可重复性至关重要：开发人员应该构建`WASM`模块，以便可以从头开始构建完全相同的`WASM`。然后用户可以将重建的`WASM`模块的`hash`与`IC`给出的`hash`进行比较。开发人员可以在[可重建的罐头](https://smartcontracts.org/docs/developers-guide/tutorials/reproducible-builds.html)一节找到重建指南。

## 罐头的行为不能意外地改变（The behavior of the canister cannot unexpectedly change）

罐头由控制着（`controller`）部署和掌管。在其它功能中，控制着可以更改它们掌管的罐头的代码，因此罐头的代码是可变的，这与其它区块链上的智能合约不同。此功能是罐头更接近典型的软件，并使它们适用于可以根据需要更改软件逻辑的广泛应用程序。

对于像`DeFi`中使用的关键应用程序，可变性可能是危险的；控制者可以将一个正常的罐头编程一个可以盗取`ICP`的罐头来骗取`ICP`。下面我们概述了一些可供开发人员使用的关于如何可验证地限制可变性的选项。

### 完全不可变

最简单的方法是通过清空罐头的控制者`controller`数组来实现完全不可变性。用户可以通过`dfx`非常用以地检验罐头的控制者。例如下面的命令：

``` bash
dfx canister --no-wallet --network ic info ryjl3-tyaaa-aaaaa-aaaba-cai
```

将会返回`pricipal`为`ryjl3-tyaaa-aaaaa-aaaba-cai`的罐头的所有控制者（在这里，`ryjl3-tyaaa-aaaaa-aaaba-cai`罐头的控制者只有`ledger`罐头）。

如果一个罐头的控制者是空的，那这个罐头就是不可变的。

用户可以通过`read_state`请求获取另一个罐头的控制者数组，以获取包含控制者数组的罐头的相关信息。注意：目前罐头无法获取此信息。

不可变性也可以通过设置罐头的控制者数组仅包含自身来实现。但是在这种情况下，你需要仔细验证罐头不能以某种方式提交升级自身的请求，例如通过发送重新安装请求。在这里，代码检查和可重现的构建至关重要。

最后，一个更有用的解决方案是将罐头的控制权交给所谓的“黑洞罐头”。这个罐头本身是不可变的（它只有自己作为自己的控制者），但允许第三方获取有关黑洞控制的罐头的有用信息，例如黑洞罐头的一个实例是`e3mmv-5qaaa-aaaah-aadma-cai`，关于这个黑洞罐头的详细文档在[这里](https://github.com/ninegua/ic-blackhole)。

### 受控可变

一种更复杂但更加强大的方法是通过分布式治理机制来控制罐头。可以想象这样一种治理机制：实现不同级别的复杂性和控制。一个例子是（即将推出的）[`SNS`](https://medium.com/dfinity/how-the-service-nervous-system-sns-will-bring-tokenized-governance-to-on-chain-dapps-b74fb8364a5c)功能，它允许开发者将其罐头的控制者设置为某个管理者罐头。

不用多说，这样的情况下，对罐头的信任就被转移到了`SNS`，所以有关于代码检查和再现性的考虑都使用。