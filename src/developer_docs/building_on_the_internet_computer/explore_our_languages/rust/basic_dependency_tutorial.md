# 基本依赖教程（Basic dependency）

`dapp`设计的一种常见方法是在一个罐头中计算或存储数据，然后你可以在另一个罐头中使用这些数据。这种共享和使用不同罐头中定义的功能的能力，即使底层罐头是用不同的编程语言编写的，也是构建`dapp`以运行`IC`罐头的重要策略。本教程提供了有关如何使用一种语言（例如`Motoko`）编写函数的基本介绍，然后使用另一种语言（本例是`Rust`）中的数据。

对于本教程，两个罐头都在同一个项目中。

- `Motoko`罐头创建一个带有`cell`变量的`actor`，以包含操作产生的当前值。
- `mul`函数将一个自然数作为输入，将输入值乘以`3`并将结果存储在`cell`变量中。
- `Rust`罐头提供了一个简单的`read`函数，它返回`cell`变量的当前值。请注意，即使要调用的方法是`query`类型的，进行罐头间调用的函数也应该是`update`类型的方法。

## 开始之前（Before you begin）

-----

在开始项目之前，请验证以下内容：

- 你有互联网连接并可以访问本地`MacOS`或`Linux`计算机上的`shell`终端。
- 你已经按照操作系统的`rust`安装说明下载并安装了`rust`编程语言和`cargo`。
    ``` bash
    curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
    ```
    `rust`工具链版本必须大于等于`1.46.0`。
- 你已经按照下载和安装中的说明下载并安装了`dfx`。
- 你已经安装了`cmake`。例如，通过如下命令使用`homebrew`：
    ``` bash
    brew install cmake
    ```
- 你已经停止在你的计算机上运行的任何本地罐头执行环境进程。

如果你不确定如何在本地计算机上打开新的终端`shell`、在终端中运行命令、如何检查和安装软件包，请参阅[新手的初步步骤](https://smartcontracts.org/docs/quickstart/newcomers.html)。如果你愿意在没有说明的情况下满足先决条件，请继续创建一个新项目。

完成本教程大约需要`20`分钟。

## 新建一个项目（Create a new project）

------

要为本教程创建一个新的项目目录：

1. 如果你还没有打开终端，请在本地计算机打开一个终端。
2. 通过运行以下命令创建一个新项目：
    ``` bash
    dfx new --type=rust rust_deps
    ```
3. 切换目录到新创建的项目：
    ``` bash
    cd rust_deps
    ```

## 修改默认项目（Modify the default project）

----

### 编辑默认的罐头设置（Edit the default canister settings）

由于此示例项目将包含两个罐头————`Motoko`罐头和`Rust`罐头————你需要修改默认的`dfx.json`配置文件以包含构建`Motoko`罐头和`Rust`罐头的信息。

修改`dfx.json`配置文件：

1. 如果需要，请检查你是否仍在项目的根目录。
2. 在文本编辑器打开`dfx.json`配置文件。
3. 在`canisters.rust_deps`设置之前插入一个新部分，其中包含用于构建`Motoko`程序的设置，例如，在`canister`键，添加一个新的`multiply_deps`键，其设置如下：
    ``` json
    "multiply_deps": {
        "main": "src/multiply_deps/main.mo",
        "type": "motoko"
    }
    ```
4. 将`dependencies`添加到`rust_deps`：`dependencies`是你能够从一个罐头导入函数以在另一个罐头中使用。对于本教程，我们希望从用`Motoko`编写的`multiply_deps`罐头中导入一个函数，并从用`Rust`编写的`rust_deps`罐头中使用它。然后，`rust_deps`字段如下：
    ``` json
    "rust_deps": {
        "candid": "src/rust_deps/rust_deps.did",
        "package": "rust_deps",
        "type": "rust",
        "dependencies": [
            "multiply_deps"
        ]
    }
    ```
5. 从文件中删除所有`rust_deps_assets`配置设置。
    本教程的示例`dapp`不使用任何前端资源，因此你可以从配置文件中直接删除这些设置。你还可以删除`defaults`和`dfx`版本设置，例如，修改后，你的配置文件可能像下面这样：
    ``` json
    {
        "canisters": {
            "multiply_deps": {
            "main": "src/multiply_deps/main.mo",
            "type": "motoko"
            },
            "rust_deps": {
            "candid": "src/rust_deps/rust_deps.did",
            "package": "rust_deps",
            "type": "rust",
            "dependencies": [
                "multiply_deps"
            ]
            }
        },
        "networks": {
            "local": {
            "bind": "127.0.0.1:8000",
            "type": "ephemeral"
            }
        },
        "version": 1
    }
    ```
6. 保存更改并关闭`dfx.json`文件以继续。

### 实现`Motoko`罐头（Implement Motoko canister smart constract）

下一步是在`src/multiply_deps/main.mo`中创建一个文件，其中包含实现`mul`和`read`函数的代码。

编写`Motoko`源代码：

1. 如果有需要，检查及是否仍然在项目根目录。
2. 为`Motoko`罐头创建目录：
    ``` bash
    mkdir multiply_deps
    ```
3. 在文本编辑器中创建并打开`src/multiply_deps/main.mo`文件。
4. 将下面的代码复制到`main.mo`文件中：
    ``` motoko
    actor Multiply {

        var cell : Nat = 1;

        public func mul(n:Nat) : async Nat { cell *= n*3; cell };

        public query func read() : async Nat {
            cell
        };
    }
    ```
5. 保存更改并关闭文件以继续。

### 替换默认的`rust`罐头（Replace the default Rust canister smart contract）

现在我们有了`rust`罐头所依赖的`motoko`罐头，让我们将`rust`罐头添加到项目中。

要替换默认的`rust`罐头：

1. 如果需要，请检查你是否仍在项目根目录中。
2. 在文本编辑器中打开模板文件`src/rust_deps/lib.rs`并删除现有内容。
    下一步是编写一个导入`motoko`罐并实现读取功能的`rust`程序。
3. 将下面的代码复制到`lib.rs`文件中：
    ``` rust
    use ic_cdk_macros::*;
    use ic_cdk::export::candid;

    #[import(canister = "multiply_deps")]
    struct CounterCanister;

    #[update]
    async fn read() -> candid::Nat {
        CounterCanister::read().await.0
    }
    ```
4. 保存并关闭你的`lib.rs`文件以继续。

### 更新接口文件（Update interface description file）

`candid`是一种接口描述语言，用于与在`IC`上运行的罐头交互。`candid`文件提供罐头接口的语言无关的描述，包括罐头定义的每个函数的名称、参数、返回结果格式和数据类型。

通过将`candid`文件添加到你的项目中，你可以确保将数据从`rust`中的定义正确转换为在`IC`上安全地运行。

要查看有关`candid`接口描述语言语法的详细信息，请参阅[`candid`指南](https://smartcontracts.org/docs/candid-guide/candid-intro.html)或[`candid crate`文档](https://docs.rs/candid/)。

要更新本教程的`candid`文件：

1. 如果需要，请检查你是否仍在项目根目录中。
2. 在文本编辑器中打开`src/rust_deps/rust_deps.did`文件。
3. 复制并粘贴`read`函数的以下服务定义：
    ``` candid
    service : {
        "read": () -> (nat);
    }
    ```
4. 保存更改并关闭`deps.did`文件以继续。

## 开启本地罐头执行环境（Start the local canister execution environment）

-----

## 注册、构建、部署你的项目（Register, build, and deploy your project）

------

## 调用已部署的罐头的函数（Call functions on the deployed canister）

------

## 停止本地罐头执行环境（Stop the local canister execution environment）

------