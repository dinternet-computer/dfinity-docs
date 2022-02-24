# 本地开发

假设你已经安装好了`dfx`。本地开发有助于调试。

首先，让我们一起创建和部署一个只有单个名为`greet`函数的罐头。`greet`函数接收一个文本参数，然后返回一个类似`Hello, everyone!`的消息在终端中。你能够通过终端或者浏览器来调用这个罐头。

## 开始之前

在下载和安装`dfx`之前，确保下面两件事：

- 有网络链接；打开了`Terminal`。
- 如果你想为你的罐头搭建前端，那你需要`nodejs`。

## 下载和安装`dfx`

在新版的`dfx`仅通过命令行就能直接安装。

1. 在你的电脑打开一个终端
    比如`MacOS`上直接`cmd` + `spacebar`打开搜索框，然后输入`terminal`。
2. 通过下面的命令下载最新版的`dfx`：
    ``` bash
    sh -ci "$(curl -fsSL https://sdk.dfinity.org/install.sh)"
    ```
3. 输入`y`同意所有的条款。

## 确定`dfx`已经安装成功了

上面的安装脚本没有输出任何错误的情况下，那一切都安装好了。

1. 输入下面的命令
    ``` bash
    dfx --version
    ```
    会有类似下面这样的输出：
    ``` bash
    dfx 0.8.4
    ```    
2. 显示帮助信息
    ``` bash
    dfx --help
    ```

## 创建一个新项目

`Internet Computer`主网上的每个`Dapp`都是一个项目，你可以非常轻松地用`dfx`创建一个新项目。

下面我们就用`dfx`来创建一个简单的，带有默认的实例代码的项目来说明一下使用`dfx`创建新项目的流程。

1. 打开一个新的终端。
2. 使用下面的命令创建一个名为`hello`的项目：
    ``` bash
    dfx new hello
    ```
    上面的命令创建一个新的名为`hello`的文件夹、样板文件、`hello`的`git`仓库
3. 打开这个项目
    ``` bash
    cd hello
    ```

## 开始本地开发

在写项目代码之前，你需要连接到本地的罐头运行环境。最好的建议是同时打开两个终端，这样你能够在一个终端查看罐头的输出，另一个终端管理罐头。

准备本地运行环境步骤：

1. 打开第二个终端。
2. 进入你项目的根目录。
3. 使用下面的命令一个本地的罐头运行环境：
    ``` bash
    dfx start
    ```

## 注册、创建、部署

只有本地罐头运行环境成功启动了，才能在它上面部署运行调试罐头。

部署的步骤：

1. 检查是否在项目的根目录（`dfx.json`所在的目录）
2. 确保`node`包全部已经安装好，否则：
    ``` bash
    npm install
    ```
3. 部署：
    ``` bash
    dfx deploy
    ```
    上面这条命令会输出每一步操作的结果，比如第一步是注册两个`identity`：一个是给`hello`主罐头用户；另一个是给`hello_assets`前端罐头用的：

    ``` text
    Creating a wallet canister on the local network.
    The wallet canister on the "local" network for user "default" is "rwlgt-iiaaa-aaaaa-aaaaa-cai"
    Deploying all canisters.
    Creating canisters...
    Creating canister "hello"...
    "hello" canister created with canister id: "rrkah-fqaaa-aaaaa-aaaaq-cai"
    Creating canister "hello_assets"...
    "hello_assets" canister created with canister id: "ryjl3-tyaaa-aaaaa-aaaba-cai"
    Building canisters...
    Building frontend...
    Installing canisters...
    Creating UI canister on the local network.
    The UI canister on the "local" network is "r7inp-6aaaa-aaaaa-aaabq-cai"
    Installing code for canister hello, with canister_id rrkah-fqaaa-aaaaa-aaaaq-cai
    Installing code for canister hello_assets, with canister_id ryjl3-tyaaa-aaaaa-aaaba-cai
    Authorizing our identity (default) to the asset canister...
    Uploading assets to asset canister...
    /index.html 1/1 (573 bytes)
    /index.html (gzip) 1/1 (342 bytes)
    /index.js 1/1 (605692 bytes)
    /index.js (gzip) 1/1 (143882 bytes)
    /main.css 1/1 (484 bytes)
    /main.css (gzip) 1/1 (263 bytes)
    /sample-asset.txt 1/1 (24 bytes)
    /logo.png 1/1 (25397 bytes)
    /index.js.map 1/1 (649485 bytes)
    /index.js.map (gzip) 1/1 (149014 bytes)
    Deployed canisters.
    ```
    你应该注意到了，在首次部署的时候，`dfx`会用默认的`identity`的`cycles`钱包创建一个默认的受控的`identity`。`cycles`钱包是一类特殊的罐头，它允许你转移`cycles`到其它的罐头。
    至此，你没必要马上全部了解部署一个罐头的时候，`dfx`究竟做了什么（如果使用`cycles`钱包、管理`cycles`），我们将会在后面的章节详细介绍。

4. 通过下面的命令调用`hello`罐头中的`greet`函数：
    ``` bash
    dfx canister call hello greet everyone
    ```
    上面命令的参数的含义：

    - `dfx canister call`：调用某个罐头的某个函数。
    - `hello`：待调用的罐头的名字。
    - `greet`：相应的方法。
    - `everyone`：传递给方法的参数。

5. 终端会输出`greet`方法的返回值：
    ``` text
    ("Hello, everyone!")
    ```

## 测试前端

现在你已经确认`hello`罐头已经成功在本地运行了，让我们继续测试前端罐头。

1. 使用`npm start`命令打开服务器
2. 打开一个浏览器
3. 导航到[http://localhost:8080/](http://localhost:8080/)

一切没问题的话，浏览器将会显示一个简单的画面。

## 停止本地罐头运行环境

测试完成前端后，你需要手动停止本地罐头运行环境，否则它会一直在后台运行！

步骤：

1. 在运行前端服务器的终端，使用快捷键`ctrl` + `c`停止服务器
2. 在本地罐头运行环境终端，同样使用`ctrl` + `c`停止本地网络
3. 在本地罐头运行环境终端，使用下面命令停止运行环境：
    ``` bash
    dfx stop
    ```

## 下一步

本节仅仅涉及一些部署罐头到`Internet Computer`的关键步骤。你能够在后续其它章节看到更详细的教程。

这里是一些建议：

- [Tutorials](https://smartcontracts.org/docs/developers-guide/tutorials-intro.html)：探索如何使用本地运行环境创建部署一个简单的罐头。
- [转换ICP成Cycles](https://smartcontracts.org/docs/quickstart/network-quickstart.html#convert-icp)：如果你有`ICP`，那么你可以将`ICP`转换成`Cycles`以至于你能够部署罐头到`Internet Computer`主网。
- [主网部署](https://smartcontracts.org/docs/quickstart/network-quickstart.html)：使用`Cycles`将罐头部署到`Internet Computer`主网。
- [什么是Candid](https://smartcontracts.org/docs/candid-guide/candid-concepts.html)：了解接口描述语言`Candid`是如何赋予各种服务`service`互操作性和组合性的。
- [motoko一览](https://smartcontracts.org/docs/languages/motoko-at-a-glance.html)：了解`Motoko`的语法和特性。