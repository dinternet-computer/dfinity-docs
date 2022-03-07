本教程将逐步指导你将自己的代币部署到`IC`并将`Rosetta API`连接到它。

# 部署你的`ledger`（Deploy your Ledger）

-----

1. 确保你拥有账本镜像、私有`ledger`接口和共有`ledger`接口。如果没有，在[本地运行`ledger`](./setup_ledger_locally.md)一节学习如何下载。
2. 确保你安装了最新版本的`dfx`。如果没有安装，请到[https://smartcontracts.org/](https://smartcontracts.org/)学习如何安装。
3. 将以下罐头定义添加到项目的`dfx.json`文件中：
    ``` json
    {
        "canisters": {
            "custom-ledger": {
            "type": "custom",
            "wasm": "ledger.wasm",
            "candid": "ledger.private.did"
            }
        }
    }
    ```
4. 将`ledger`部署到`IC`：
    ``` bash
    # Change the variable to "ic" to deploy the ledger on the mainnet.
    export NETWORK=local

    # Change the variable to the account that can mint and burn tokens.
    export MINT_ACC=$(dfx ledger account-id)

    # Change the variable to the principal that controls archive canisters.
    export ARCHIVE_CONTROLLER=$(dfx identity get-principal)

    export TOKEN_NAME="My Token"
    export TOKEN_SYMBOL=XMTK

    dfx deploy --network ${NETWORK} custom-ledger --argument '(record {
    name = "'${TOKEN_NAME}'";
    symbol = "'${TOKEN_SYMBOL}'";
    minting_account = "'${MINT_ACC}'";
    initial_values = vec {};
    send_whitelist = vec {};
    archive_options = opt record {
        trigger_threshold = 2000;
        num_blocks_to_archive = 1000;
        controller_id = principal "'${ARCHIVE_CONTROLLER}'";
        cycles_for_archive_creation = opt 10_000_000_000_000
    }
    })'
    ```

其中，

- `NETWORK`是你要在其中部署`ledger`罐头的`URL`或名称（例如，使用`IC`作为主网）。
- `TOKEN_NAME`是新代币的名称。
- `TOKEN_SYMBOL`是你的新代币的符号。
- `MINT_ACC`是负责铸造和销毁代币的委托人的账户（请参阅`ledger`文档）。
- `ARCHIVE_CONTROLLER`是归档罐头的控制器主体。

> 重要提示，当你部署到主网时，请始终设置`archive_options`字段，如果归档被禁用，你的`ledger`的容量将被限制为单个罐头的内存；确保`ledger`罐头有足够的燃料。罐头需要燃料来按需生成归档罐头的新实例。附加到`create_canister`消息的确切燃料数由`cycles_for_archive_creation`选项控制。

5. 更新`dfx.json`文件中的罐头定义以使用公共`candid`接口：
    ``` json
    {
        "canisters": {
            "ledger": {
            "type": "custom",
            "wasm": "ledger.wasm",
        -       "candid": "ledger.private.did"
        +       "candid": "ledger.public.did"
            }
        }
    }
    ```
6. 检查`ledger`罐头是否健康。执行以下命令：
    ``` bash
    dfx canister --network ${NETWORK} call ledger symbol
    ```
    预期输出如下：
    ``` text
    (record { symbol = "XMTK" })
    ```
    至此，你的代币已经部署并可以正常使用。

## 连接到`Rosetta API`（Connect rosetta-api）

-----

`Rosetta-api`是一个连接到`ledger`罐头并公开`rosetta-api`标准接口的应用程序。其主要目的是促进与交易所的整合。该应用程序可作为`docker`镜像使用。

现在让我们将`rosetta-api`连接到现有的`ledger`罐头。

1. 获得代币符号：
    ``` bash
    dfx canister call <ledger_canister_id> symbol
    ``` 
    预期输出如下：
    ``` text
    (record { symbol = <token_symbol> })
    ```
2. 运行`rosetta-api`：
    ``` bash
    docker run \
        --interactive \
        --tty \
        --publish 8081:8080 \
        --rm \
        dfinity/rosetta-api:v1.3.0 \
        --canister-id <ledger_canister_id> \
        --ic-url <replica> \
        -t <token_symbol>
    ```
    预期输出如下：
    ``` text
    16:31:45.472550 INFO [main] ic_rosetta_api::rosetta_server - Starting Rosetta API server
    16:31:45.506905 INFO [main] ic_rosetta_api::ledger_client - You are all caught up to block <x>
    ```
    上面的`<x>`代表`ledger`中的最后一个区块索引。

现在，`rosetta-api`已经连接到你的`ledger`罐头并可以正常使用。阅读[传输代币文章](https://smartcontracts.org/docs/rosetta-api/transfers.html)以了解`Rosetta`代币传输的操作。
