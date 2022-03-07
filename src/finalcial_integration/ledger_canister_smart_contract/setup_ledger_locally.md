按照下面的步骤将`ledger`罐头部署到本地环境。

1. 获取预构建的`ledger`罐头模块和`candid`接口文档。

``` bash
export IC_VERSION=a7058d009494bea7e1d898a3dd7b525922979039
curl -o ledger.wasm.gz https://download.dfinity.systems/ic/${IC_VERSION}/canisters/ledger-canister_notify-method.wasm.gz
gunzip ledger.wasm.gz
curl -o ledger.private.did https://raw.githubusercontent.com/dfinity/ic/${IC_VERSION}/rs/rosetta-api/ledger.did
curl -o ledger.public.did https://raw.githubusercontent.com/dfinity/ic/${IC_VERSION}/rs/rosetta-api/ledger_canister/ledger.did
```

> 注意，`IC_VERSION`变量来自[`http://github.com/dfinity/ic`](http://github.com/dfinity/ic)仓库提交的哈希。
2. 确保你使用的是最新版本的`dfx`，如果你没有安装`dfx`，请按照[`https://smartcontracts.org/`](https://smartcontracts.org/)上的说明进行安装。
3. 如果你还没有`dfx`项目，请按照以下说明创建新的`dfx`项目：[https://smartcontracts.org/docs/developers-guide/cli-reference/dfx-new.html](https://smartcontracts.org/docs/developers-guide/cli-reference/dfx-new.html)。
4. 将你在第一步获得的文件（`ledger.wasm`、`ledger.private.did`、`ledger.public.did`）复制到项目的根目录中。
5. 将以下罐头定义添加到项目中的`dfx.json`文件中：
    ``` json
    {
    "canisters": {
        "ledger": {
        "type": "custom",
        "wasm": "ledger.wasm",
        "candid": "ledger.private.did"
        }
    }
    }
    ```
6. 启动本地环境：
    ``` bash
    dfx start --background
    ```
7. 创建一个将用作铸币账户的新身份：
    ``` bash
    dfx identity new minter
    dfx identity use minter
    export MINT_ACC=$(dfx ledger account-id)
    ```
    来自铸币账户的转账将创建铸币交易。转移到铸币账户将创建`Burn`交易。
8. 切换回你的默认身份并记录其`ledger`账户标识符。
    ``` bash
    dfx identity use default
    export LEDGER_ACC=$(dfx ledger account-id)
    ```
9. 将`ledger`罐头部署到你的网络：
    ``` bash
    dfx deploy ledger --argument '(record {minting_account = "'${MINT_ACC}'"; initial_values = vec { record { "'${LEDGER_ACC}'"; record { e8s=100_000_000_000 } }; }; send_whitelist = vec {}})'
    ```
    使用归档进行部署：
    如果你想以与生成部署相匹配的方式设置`ledger`，你应该在启用归档的情况下部署它。在此设置中，`ledger`罐头动态分配创建新罐头来存储旧块。如果你打算练习获取块的界面，我们建议使用此设置。

    获取你用于开发的身份的主体。该主体将是归档内容的控制器。
    ``` bash
    dfx identity use default
    export ARCHIVE_CONTROLLER=$(dfx identity get-principal)
    ```
    使用归档选项部署`ledger`罐头：
    ``` bash
    dfx deploy ledger --argument '(record {minting_account = "'${MINT_ACC}'"; initial_values = vec { record { "'${LEDGER_ACC}'"; record { e8s=100_000_000_000 } }; }; send_whitelist = vec {}; archive_options = opt record { trigger_threshold = 2000; num_blocks_to_archive = 1000; controller_id = principal "'${ARCHIVE_CONTROLLER}'" }})'
    ```
    你可能希望将`trigger_threshold`和`num_blocks_to_archive`选项设置为较低的值（例如`10`和`5`），以便仅在几个块后触发归档。
10. 更新`dfx.json`文件中的罐头定义以使用公共`candid`接口：
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
11. 检查`ledger`罐头是否健康。执行以下命令：
    ``` bash
    dfx canister call ledger account_balance '(record { account = '$(python3 -c 'print("vec{" + ";".join([str(b) for b in bytes.fromhex("'$LEDGER_ACC'")]) + "}")')' })'
    ```
    预期输出如下：
    ``` text
    (record { e8s = 100_000_000_000 : nat64 })
    ```

你本地的`ledger`罐头现在已经成功启动并运行。你现在可以部署需要与`ledger`罐头通信的其它罐头。