# 部署到`Internet Computer`

本教程假设你已经安装好了`dfx`，并准备部署项目到`Internet Computer`。

如果你只是准备部署项目到本地的罐头运行环境，请浏览[本地部署](./local_development.md)。

ok，让我们现在开始部署一个只有单个`greet`函数的罐头（这个罐头接受一个文本参数，并返回类似`Hello everyone!`这样的返回）到`Internet Computer`。

## 创建一个新项目

`Internet Computer`上的应用都是从创建一个新项目开始的。

在本节，我们将会使用`dfx`生成的默认代码说明如何创建部署一个简单的应用到`Internet Computer`。

步骤：

1. 打开一个终端
2. 通过下面的命令创建一个新项目
    ``` bash
    dfx new hello
    ```
3. 打开新创建的项目
    ``` bash 
    cd hello
    ```

## 检查是否能正常连接到`Internet Computer`主网

1. 进入新项目的根目录。
2. 使用下面的命令获取`Internet Computer`的状态，以及你是否能正常连接到`Internet Computer`主网。
    ``` bash
    dfx ping ic
    ```
3. 查看返回结果，如果像下面这样，那就是正常的：
    ``` json
    {
    "ic_api_version": "0.18.0"  "impl_hash": "d639545e0f38e075ad240fd4ec45d4eeeb11e1f67a52cdd449cd664d825e7fec"  "impl_version": "8dc1a28b4fb9605558c03121811c9af9701a6142"  "replica_health_status": "healthy"  "root_key": [48, 129, 130, 48, 29, 6, 13, 43, 6, 1, 4, 1, 130, 220, 124, 5, 3, 1, 2, 1, 6, 12, 43, 6, 1, 4, 1, 130, 220, 124, 5, 3, 2, 1, 3, 97, 0, 129, 76, 14, 110, 199, 31, 171, 88, 59, 8, 189, 129, 55, 60, 37, 92, 60, 55, 27, 46, 132, 134, 60, 152, 164, 241, 224, 139, 116, 35, 93, 20, 251, 93, 156, 12, 213, 70, 217, 104, 95, 145, 58, 12, 11, 44, 197, 52, 21, 131, 191, 75, 67, 146, 228, 103, 219, 150, 214, 91, 155, 180, 203, 113, 113, 18, 248, 71, 46, 13, 90, 77, 20, 80, 95, 253, 116, 132, 176, 18, 145, 9, 28, 95, 135, 185, 136, 131, 70, 63, 152, 9, 26, 11, 170, 174]
    }
    ```

## 确认你的开发`Identity`和`ledger`账户

`Internet Computer`上的所有`ICP`代币转账都记录在一个叫做[ledger](https://smartcontracts.org/docs/developers-guide/glossary.html#g-ledger)的账户中。`ledger`记录了所有`ICP`持有者的账户`Identity`和余额。

任何人想使用存放于`ledger`账户中的`ICP`，都需要使用相应的`Identity`来签名认证对应的转账信息来完成这笔交易。

签名的方式取决于你存放私钥的方式，私钥可以存放在硬件中，或者是软件实现。比如，你可能会通过一个包含硬件安全模块（`HSM`即`Hardware security module`）的硬件来和`ledger`账户交互、或者通过`nns`前端网页应用、再或者通过`dfx`命令行工具。每种方法都只是不同的与`ledger`账户交互的接口而已。

### 关于你的开发`Identity`

首次使用`Dfinity Canister SDK`时，`dfx`命令行工具会自动创建一个名为`default`的开发`Identity`。这个`Identity`是一种用人类可读的格式显示的`principal`数据，就像是钱包地址之于比特币和以太坊。

然而，你的开发`Identity`对应的`principal`和你在`ledger`中的账户`Identity`是不一样的。开发`Identity`和`ledger`账户中的账户`Identity`都用人类可读的格式表示了你的个人`Identity`————只是使用了不同的格式。

### 连接到`ledger`账户获取账户信息

现在，我们尝试不依靠任何硬件钱包和额外的一些应用————直接使用`dfx`工具和你的开发`Identity`连接到`ledger`账户，并使用里面的`ICP`充值我们的`Cycles`钱包。

查看`ledger`中的账户：

1. 使用下面的命令查看当前正在使用的开发`Identity`：
    ``` bash
    dfx identity whoami
    ```
    一般情况下，当前使用的开发`Identity`都是`default`账户：
    ``` text
    default
    ```
2. 使用下面的命令查看当前`Identity`的`principal`：
    ``` bash
    dfx identity get-principal
    ```
    返回类似下面这样的结果：
    ``` text
    tsqwz-udeik-5migd-ehrev-pvoqv-szx2g-akh5s-fkyqc-zy6q7-snav6-uqe
    ```
3. 获取当前开发`Identity`对应的账户`identifier`（account identifier）：
    ``` bash
    dfx ledger account-id
    ```
    上面的命令将会返回当前`Identity`对应的`account identifier`：
    ``` text
    03e3d86f29a069c6f2c5c48e01bc084e4ea18ad02b0eec8fccadf4487183c223
    ```    
3. 查看当前账户余额：
    ``` bash
    dfx ledger --network ic balance
    ```
    这条命令返回当前`Identity`在`ledger`账户中的余额信息：
    ``` text
    10.00000000 ICP
    ```

## 将`ICP`转换为`Cycles`

你已经获取到当前`Identity`的账户信息，也得到了`ICP`余额。现在，让我们将一些`ICP`转换成`Cycles`并将这些`Cycles`转移到`Cycles`钱包中。

转移一些`ICP`到`Cycles`钱包：

1. 使用下面的命令将`Identity`在`ledger`账户中的一些`ICP`转出来创建一个新的罐头：
    ``` bash
    dfx ledger --network ic create-canister <principal-identifier> --amount <icp-tokens>
    ```
    上面的命令将`--amount`参数指定的数量的`ICP`转换成`Cycles`并充值到一个新创建的罐头————这个新罐头是由`<principal-identifier>`控制的。
    例如，使用默认开发`Identity`将它存放于`ledger`中的`0.25`个`ICP`转换成`Cycles`并充值到新创建的罐头中：
    ``` bash
    dfx ledger --network ic create-canister tsqwz-udeik-5migd-ehrev-pvoqv-szx2g-akh5s-fkyqc-zy6q7-snav6-uqe --amount .25
    ```
    如果上面的操作成功会返回类似下面的结果：
    ``` text
    Transfer sent at BlockHeight: 20
    Canister created with id: "gastn-uqaaa-aaaae-aaafq-cai"
    ```
2. 使用下面的命令安装`Cycles`钱包到某个罐头：
    ``` bash
    dfx identity --network ic deploy-wallet <canister-identifier>
    ```
    比如，安装`Cycles`钱包到刚刚创建的罐头中：
    ``` bash
    dfx identity --network ic deploy-wallet gastn-uqaaa-aaaae-aaafq-cai
    ```
    正常情况下，上面的命令会有如下结果作为输出：
    ``` text
    Creating a wallet canister on the ic network.
    The wallet canister on the "ic" network for user "default" is "gastn-uqaaa-aaaae-aaafq-cai"
    ```

## 确认你的`Cycles`钱包

在你完成上面的步骤之后，你能够通过命令查看你的`Cycles`钱包罐头并且检查`Cycles`余额。

确认`Cycles`钱包：

1. 使用下面的命令获取你刚刚部署的`Cycles`钱包的`canister identifier`：
    ``` bash
    dfx identity --network ic get-wallet
    ```
    正常情况下，直接返回钱包罐头的`identifier`：
    ``` text
    gastn-uqaaa-aaaae-aaafq-cai
    ```
2. 使用下面的命令获取`Cycles`钱包的余额：
    ``` bash
    dfx wallet --network ic balance
    ```
    正常情况下，直接返回`Cycles`余额：
    ``` text
    15430122328028812 cycles.
    ```
    当然，你也能够通过下面这个连接直接获取`Cycles`余额：
    ``` text
    https://<WALLET-CANISTER-ID>.raw.ic0.app
    ```
    首次打开这个网页时，你会看到它提示当前是匿名用户，这表明你需要认证你的`Identity`、授权访问`Cycles`钱包、注册设备。
3. 点击`Authenticate`按钮，网页会导航到`Internet Identity`让你登录`Identity`。
4. 输入先前注册好的`User Number`或者新注册一个用户。[点击这里](https://smartcontracts.org/docs/ic-identity-guide/auth-how-to.html)查看在`Internet Identity`注册用户的详情。
5. 在`Internet Identity`完成认证。
6. 点击`Proceed`按钮授权访问你的`Cycles`钱包。
7. 复制`Register`页面显示的命令到终端运行————注册本次会话使用的设备。比如，调用`Cycles`钱包的`authorize`方法：
    ``` bash
    dfx canister --no-wallet --network ic call "gastn-uqaaa-aaaae-aaafq-cai" authorize '(principal "ejta3-neil3-qek6c-i7rdw-sxreh-lypfe-v6hjg-6so7x-5ugze-3iohr-2qe")'
    ```
    注意，上面的命令必须确保使用了`--no-wallet`选项和正确指定网络为`ic`。
    运行完上面的命令后，刷新浏览器就能看到当前`Identity`绑定的`Cycles`钱包。
8. 在浏览器上查看`Cycles`余额和`Cycles`使用情况：更多使用默认`Cycles`钱包的例子可以在[这里](https://smartcontracts.org/docs/developers-guide/default-wallet.html)找到。

## 注册、创建、部署你的应用

在你完成上面的步骤之后，就能继续注册、创建和部署你的应用到`Internet Computer`了。

步骤：

1. 打开终端，进入项目的根目录。
2. 使用下面的命令确保成功安装好了`node`模块：
    ``` bash
    npm install
    ```
    更多信息点击[这里](https://smartcontracts.org/docs/developers-guide/webpack-config.html#troubleshoot-node)
3. 使用下面的命令注册、创建、部署应用到`Internet Computer`主网：
    ``` bash
    dfx deploy --network ic
    ```
    `--network`选项指定了应用将被部署到的网络。一切没有问题的情况下，上面的命令会有类似下面这样的输出：
    ``` text
    Deploying all canisters.
    Creating canisters...
    Creating canister "hello"...
    "hello" canister created on network "ic" with canister id: "5o6tz-saaaa-aaaaa-qaacq-cai"
    Creating canister "hello_assets"...
    "hello_assets" canister created on network "ic" with canister id: "5h5yf-eiaaa-aaaaa-qaada-cai"
    Building canisters...
    Building frontend...
    Installing canisters...
    Installing code for canister hello, with canister_id 5o6tz-saaaa-aaaaa-qaacq-cai
    Installing code for canister hello_assets, with canister_id 5h5yf-eiaaa-aaaaa-qaada-cai
    Authorizing our identity (default) to the asset canister...
    Uploading assets to asset canister...
    /index.html 1/1 (472 bytes)
    /index.html (gzip) 1/1 (314 bytes)
    /index.js 1/1 (260215 bytes)
    /index.js (gzip) 1/1 (87776 bytes)
    /main.css 1/1 (484 bytes)
    /main.css (gzip) 1/1 (263 bytes)
    /sample-asset.txt 1/1 (24 bytes)
    /logo.png 1/1 (25397 bytes)
    /index.js.map 1/1 (842511 bytes)
    /index.js.map (gzip) 1/1 (228404 bytes)
    /index.js.LICENSE.txt 1/1 (499 bytes)
    /index.js.LICENSE.txt (gzip) 1/1 (285 bytes)
    Deployed canisters.
    ``` 
    如果`Cycles`不足，通过类似下面这样的命令增加`Cycles`：
    ``` bash
    dfx ledger --network ic top-up gastn-uqaaa-aaaae-aaafq-cai --amount 1.005
    ```
    正常情况下，成功充值`Cycles`之后命令行会显示：
    ``` text
    Transfer sent at BlockHeight: 81520
    Canister was topped up!
    ```
4. 调用`hello`罐头的`greet`函数：
    ``` bash
    dfx canister --network ic call hello greet '("everyone": text)'
    ```
    `--network`：指定罐头所在的网络为`ic`，即`Internet Computer`主网。
    `hello`：指定调用罐头的名称。
    `greet`：指定罐头内的函数。
    `everyone`：传递给`greet`函数的参数。
5. 正常情况下，终端会返回：
    ``` bash
    ("Hello, everyone!")
    ```
6. 运行`dfx wallet balance`检查当前的`Cycles`余额。

## 测试前端

1. 打开浏览器
2. 使用`hello_assets`的`Identity`拼接到`ic0.app`，并打开这个网页。如果你还没有记录`hello_assets`的`Identity`，可以通过下面这个命令查看：
    ``` bash
    dfx canister --network ic id hello_assets
    ```
    最后的链接应该类似下面这样：
    ``` text
    https://gsueu-yaaaa-aaaae-aaagq-cai.raw.ic0.app
    ```

## 更多

建议阅读：

- [Tutorial](https://smartcontracts.org/docs/developers-guide/tutorials-intro.html)：探索在本地环境开发运行罐头和前端应用。
- [什么是Candid](https://smartcontracts.org/docs/candid-guide/candid-concepts.html)：了解`Candid`接口语言是如何赋予`service`互操作性和组合性的。
- [Motoko一览](https://smartcontracts.org/docs/languages/motoko-at-a-glance.html)：了解`Motoko`语言的语法和特性。