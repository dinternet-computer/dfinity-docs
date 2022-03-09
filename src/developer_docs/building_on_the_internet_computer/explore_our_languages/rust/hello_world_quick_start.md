# `Rust CDK`快速入门（Hello, World! Rust CDK Quick Start）

适用于`rust`的罐头开发工具包（`CDK`）提供工具、示例代码和文档，以帮助你创建在`IC`上运行的去中心化应用。这个`helloworld`假设你第一次安装`Rust CDK`。

为了帮助你入门，本教程说明了如何修改传统的`hello world`第一个去中心化应用以使用`Rust CDK`。这个简单的`dapp`只有一个将文本打印到终端的功能，但它提供了一个很好的模型，可以在使用`rust`编写要部署在`IC`上的`dapp`时理解工作流程。

## 在你开始之前（Before you begin）

------

在开始项目之前，请验证以下内容：

- 你有互联网连接并可以访问本地`MacOS`或`Linux`计算机上的`shell`终端。
- 你已按照操作系统的`rust`安装说明下载并安装了`rust`编程语言和`Cargo`：
    ``` bash
    curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
    ```
    `rust`工具链必须是`1.46.0`或更高的版本。
- 你已按照下载和安装中的说明下载并安装了`dfx`。
- 你安装了`cmake`。例如，通过下面的命令使用`homebrew`：
    ``` bash
    brew install cmake
    ```
    有关如何安装`homebrew`的说明，请参阅[`homebrew`文档](https://docs.brew.sh/Installation)。
- 你已经停止在你的计算机上运行的任何本地执行环境进程。

如果你还不确定如何在本地计算机上打开终端、在终端中运行命令、如何检查和安装软件包，请参阅[新手的初步教程](https://smartcontracts.org/docs/quickstart/newcomers.html)。如果你愿意在没有说明的情况下满足先决条件，请继续[创建一个新项目](https://smartcontracts.org/docs/rust-guide/rust-quickstart.html#create-a-new-project)。

## 创建一个新项目（Create a new project）

------

`IC`的应用程序从项目开始。你可以使用`dfx`为`IC`创建新项目。因为你在构建这个项目以部署在`IC`上，所以本教程重点介绍如何使用`dfx`命令及其子命令创建、构建、部署`rust`程序。

要使用`dfx`创建新项目：

1. 如果你还没有打开终端，请在本地计算机上打开终端。
2. 通过运行以下命令，使用名为`rust_hello`的`rust`罐头创建一个新项目：
    ``` bash
    dfx new --type=rust rust_hello
    ```
    `dfx new --type=rust rust_hello`命令为你的项目创建一个新的`rust_hello`项目目录、模板文件、一个新的`git`仓库。
3. 通过运行以下命令切换到你的项目目录：
    ``` bash
    cd rust_hello
    ```

## 看看项目（Take a look at the project）

------

该项目已准备好编译并部署到`IC`。在此之前，让我们浏览一下项目文件。

**dfx.json**

项目目录中包含的模板文件之一是默认的`dfx.json`配置文件。该文件包含为`IC`构建项目所需的设置，就像`Cargo.toml`文件为`rust`程序提供构建和包管理配置详细信息一样。

配置文件默认内容应该像下面这样：

``` json 
{
  "canisters": {
    "rust_hello": {
      "candid": "src/rust_hello/rust_hello.did",
      "package": "rust_hello",
      "type": "rust"
    },
    "rust_hello_assets": {
      "dependencies": [
        "rust_hello"
      ],
      "frontend": {
        "entrypoint": "src/rust_hello_assets/src/index.html"
      },
      "source": [
        "src/rust_hello_assets/assets",
        "dist/rust_hello_assets/"
      ],
      "type": "assets"
    }
  },
  "defaults": {
    "build": {
      "args": "",
      "packtool": ""
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

请注意，在`canisters`键下，你有一些`rust_hello`罐头的默认设置。

- `"type": "rust"`：指定`rust_hello`是`rust`类型的罐头。
- `"candid": "src/rust_hello/rust_hello.did"`：指定用于罐头的`candid`接口描述文件的位置。
- `"package": "rust_hello"`：指定`rust`的`crate`的包名。它应该与`crate`的`Cargo.toml`文件中的相同。

**Cargo.toml**

在根目录下，有一个`Cargo.toml`文件。

它通过指定每个`crate`的路径来定义一个`rust`工作区。`rust`类型的罐头只是编译为`WebAssembly`的`crate`。在这里，我们在`src/rust_hello`有一个成员，这是唯一的`rust`罐头。

``` toml
[workspace]
members = [
    "src/rust_hello",
]
```

**src/rust_hello/**

我们在`rust`罐头中。与任何标准的`crate`一样，它有一个`Cargo.toml`文件，用于配置构建`crate`的详细信息。

**src/rust_hello/Cargo.toml**

``` toml
[package]
name = "rust_hello"
version = "0.1.0"
edition = "2018"

[lib]
path = "lib.rs"
crate-type = ["cdylib"]

[dependencies]
ic-cdk = "0.4"
ic-cdk-macros = "0.4"
```

请注意，`crate-type=["cdylib"]`行，这是将此`rust`程序编译为`WebAssembly`所必须的。

**src/rust_hello/lib.rs**

默认项目有一个简单的`greet`函数，它使用`Rust CDK`的`query`宏：

``` rust
#[ic_cdk_macros::query]
fn greet(name: String) -> String {
    format!("Hello, {}!", name)
}
```

**src/rust_hello/rust_hello.did**

`candid`是一种接口描述语言（`IDL`），用于与`IC`上运行的罐头进行交互。`candid`文件提供罐头接口的与语言无关的描述，包括罐头定义的每个函数的名称、参数、结果格式和数据类型。

通过将`candid`文件添加到你的项目中，你可以确保数据从`rust`中的定义正确转换为在`IC`上安全运行。

要查看有关`candid`接口描述语言语法的详细信息，请参阅[`candid`指南](https://smartcontracts.org/docs/candid-guide/candid-intro.html)或[`candid crate`文档](https://docs.rs/candid/)。

``` candid
service : {
    "greet": (text) -> (text) query;
}
```

此定义指定`greet`函数是一个以文本数据为输入并返回文本数据的查询方法。

## 启动本地环境（Start the local execution environment）

-----

在构建项目之前，你需要连接到你的开发环境中运行的本地执行环境或`IC`主网。

启动本地环境：

1. 如果需要，请检查你是否仍在项目的根目录中。
2. 通过运行以下命令在后台启动计算机上的本地执行环境：
    ``` bash
    dfx start --background
    ```
    根据你的平台和本地安全设置，你可能会看到显示的警告。如果系统提示你允许或拒绝传入的网络连接，请单击允许。

## 注册、构建、部署你的项目（Register, build, and deploy your project）

------

连接到你的开发环境中运行的本地执行环境后，你可以在本地注册、构建、和部署你的项目。

要注册、构建、部署：

1. 如果需要，请检查你是否仍在项目目录的根目录中。
2. 通过运行以下命令确保安装了`wasm32-unknown-unknuwn`：
    ``` bash
    rustup target add wasm32-unknown-unknown
    ```
3. 通过运行以下命令来注册、构建、部署`dfx.json`文件中指定的罐头：
    ``` bash
    dfx deploy
    ```
    `dfx deploy`命令输出显示有关它执行的每个操作的信息，类似于以下摘录：
    ``` text
    Creating a wallet canister on the local network.
    The wallet canister on the "local" network for user "default" is "rwlgt-iiaaa-aaaaa-aaaaa-cai"
    Deploying all canisters.
    Creating canisters...
    Creating canister "rust_hello"...
    "rust_hello" canister created with canister id: "rrkah-fqaaa-aaaaa-aaaaq-cai"
    Creating canister "rust_hello_assets"...
    "rust_hello_assets" canister created with canister id: "ryjl3-tyaaa-aaaaa-aaaba-cai"
    Building canisters...
    Executing: "cargo" "build" "--target" "wasm32-unknown-unknown" "--release" "-p" "rust_hello"
        Updating crates.io index
    Compiling unicode-xid v0.2.2
    ...
    Building frontend...
    Installing canisters...
    Creating UI canister on the local network.
    The UI canister on the "local" network is "r7inp-6aaaa-aaaaa-aaabq-cai"
    Installing code for canister rust_hello, with canister_id rrkah-fqaaa-aaaaa-aaaaq-cai
    ...
    Deployed canisters.
    ```

## 测试（Test the dapp）

-----

在本地测试部署的`dapp`：

1. 如果需要，请检查你是否仍在项目目录的根目录中。
2. 通过运行以下命令调用`dapp`中的`greet`函数：
    ``` bash
    dfx canister call rust_hello greet world
    ```
3. 验证对`rust_hello`罐头`greet`函数的调用是否返回文本消息`("Hello, world!")`。

## 停止本地执行环境（Stop the local execution environment）

-----

测试应用程序后，你可以停止本地执行环境，使其不再在后台继续运行。

停止本地执行环境：

1. 在显示网路操作的终端中，按下`ctrl-C`可以中断本地执行环境进程。
2. 通过运行以下命令停止在你的计算机上运行的本地执行环境。
    ``` bash
    dfx stop
    ```
