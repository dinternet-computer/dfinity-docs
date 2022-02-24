# Cycles水龙头

准备好部署到主网的罐头了吗？使用我们的水龙头，几分钟就能免费获得价值20美元的`Cycles`。

首先，确认你已经安装好了`dfx`：

``` bash
sh -ci "$(curl -fsSL https://smartcontracts.org/install.sh)"
```

# 获取你的`Cycles`钱包

## 步骤一：认证

首先，打开[https://faucet.dfinity.org](https://faucet.dfinity.org)，你需要一个活跃的`github`账号，如果系统判定你的`github`账号没有资格，请点击[Network Deployment](https://smartcontracts.org/docs/quickstart/cycles-faucet.html#quickstart:network-quickstart.html)

## 步骤二：Principal ID

只要你的`github`账号在[https://faucet.dfinity.org](https://faucet.dfinity.org)成功登录了，打开终端，输入一下命令

``` bash
dfx identity get-principal
```

复制粘贴终端输出的`identity`到网页的输入框中，该`identity`是唯一的，它能让你管理你的钱包和用它部署的罐头。

## 步骤三：