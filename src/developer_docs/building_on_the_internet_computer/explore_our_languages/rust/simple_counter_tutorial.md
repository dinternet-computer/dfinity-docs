# 增加一个计数器（Incrementing a counter）

在本教程中，你将编写一个`dapp`，它提供一些基本函数来增加计数器并说明值的持久性。

对于本教程，`dapp`将`counter`声明为可变变量，以包含表示计数器当前值的自然数。该`dapp`支持以下功能：

- `increment`函数更新当前值，递增`1`，没有返回值。
- `get`函数是一个简单的查询函数，它返回计数器当前值。
- `set`函数将当前值更新为你指定的参数的数值。

本教程提供了一个简单示例，说明如何通过调用已部署的罐头上的函数来增加计数器。通过调用该函数多次递增一个值，你可以验证变量状态（即两次调用之间的变量）是否持续存在。

与其他示例`dapp`一样，本教程演示了一个简单但现实的工作流程，你可以在其中执行以下步骤：

- 创建一个新项目。
- 编写一个编译成`WebAssembly`的`dapp`。
- 将该`dapp`部署到本地运行环境。
- 调用罐头里面的函数来递增然后读取计数器的值。

## 开始之前（Before you begin）

------

在开始项目之前，请验证以下内容：

- 你有互联网连接并可以访问本地`MacOS`或`Linux`计算机上的`shell`中断。
- 你已按照操作系统的`rust`安装说明下载并安装了`rust`编程语言和`cargo`。
    ``` bash
    curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
    ```
    `rust`工具链必须是`1.46.0`或更高版本。
- 你已按照下载和安装中的说明下载并安装了`dfx`。
- 你已经安装了`cmake`。例如，通过以下命令使用`homebrew`：
    ``` bash
    brew install cmake
    ```
    有关如何安装`homebrew`的说明，请参阅[`homebrew`文档](https://docs.brew.sh/Installation)。
- 你已停止在你的计算机上运行的任何本地罐头执行环境。

如果你还不确定如何在本地计算机上打开新的终端、在终端中运行命令、如何检查和安装软件包，请参阅[新手的初步步骤](https://smartcontracts.org/docs/quickstart/newcomers.html)。如果你愿意在没有说明的情况下满足先决条件，请继续创建一个新项目。

完成本教程大约需要`20`分钟。

## 创建一个新项目（Create a new project）

------

要为本教程创建一个新项目：

1. 如果你还没有打开终端，请在你的本地计算机上打开一个终端。
2. 通过运行以下命令创建一个新项目：
    ``` bash
    dfx new --type=rust rust_counter
    ```
3. 通过运行以下命令切换到你的项目目录：
    ``` bash
    cd rust_counter
    ```

## 修改默认这个项目（Modify the default project）

-----

在[Rust CDK快速入门](./hello_world_quick_start.md)，你已经了解了`rust`类型的罐头的默认项目中的文件。

要完成本教程，你需要完成以下步骤：

- [替换默认的`dapp`](#替换默认的dappreplace-the-default-dapp)
- [更新`candid`接口文件](#更新candid接口文件update-interface-description-file)

## 替换默认的`dapp`（Replace the default dapp）

-----

现在你已经为你的`Rust dapp`准备好了文件，我们将模板`lib.rs`替换为我们想要在`IC`部署的`dapp`代码。

替换默认`dapp`：

1. 如果需要，请检查你是否仍在项目的根目录。
2. 在文本编辑器中打开模板`src/rust_counter/lib.rs`文件并删除现有文件内容。
    下一步是编写一个声明`counter`变量并实现增加、获取、设置函数的`dapp`。
3. 将以下示例代码复制到`lib.rs`文件中：
    ``` rust
    use ic_cdk_macros::*;
    use ic_cdk::export::candid;

    static mut COUNTER: Option<candid::Nat> = None;

    #[init]
    fn init() {
        unsafe {
            COUNTER = Some(candid::Nat::from(0));
        }
    }

    #[update]
    fn increment() -> () {
        unsafe {
            COUNTER.as_mut().unwrap().0 += 1u64;
        }
    }

    #[query]
    fn get() -> candid::Nat {
        unsafe { COUNTER.as_mut().unwrap().clone() }
    }

    #[update]
    fn set(input: candid::Nat) -> () {
        unsafe {
            COUNTER.as_mut().unwrap().0 = input.0;
        }
    }
    ```
4. 保存更改并关闭`lib.rs`文件以继续。

## 更新`candid`接口文件（Update interface description file）

`candid`是一种语言无关的接口描述语言，用于与在`IC`上运行的`dapp`交互。`candid`文件提供罐头接口的与语言无关的描述，包括罐头定义的每个函数的名称、参数、返回结果格式、数据类型。

通过将`candid`文件添加到你的项目中，你可以确保数据从`rust`中的定义正确转换为在`IC`上安全地运行。

要查看有关`candid`接口描述语言语法的详细信息，请参阅[`candid`指南](https://smartcontracts.org/docs/candid-guide/candid-intro.html)或[`candid crate`文档](https://docs.rs/candid/)。

要更新本教程的`candid`文件：

1. 如果需要，请检查你是否仍在项目的根目录。
2. 在本文编辑器中打开`src/rust_counter/rust_counter.did`文件，然后复制并粘贴以下用于增加、获取、设置函数的服务定义：
    ``` candid
    service : {
        "increment": () -> ();
        "get": () -> (nat) query;
        "set": (nat) -> ();
    }
    ```
3. 保存你的更改并关闭`rust_counter.did`文件继续。

## 打开本地罐头执行环境（Start the local canister execution environment）

------

在构建`rust_counter`项目之前，你需要连接到你的开发环境中运行的本地罐头执行环境或`IC`主网。

启动本地罐头执行环境：

1. 如果需要，请检查你是否仍在项目目录根目录中。
2. 通过运行以下命令在后台启动本地罐头执行环境：
    ``` bash
    dfx start --background
    ```
    根据你的平台和本地安全设置，你可能会看到显示的警告。如果系统提示你允许或拒绝传入的网路连接，请单击允许。

## 注册、构建、部署你的项目（Register, build, and deploy your project）

-----

连接到在你的开发环境中运行的本地罐头执行环境后，你可以在本地注册、构建和部署你的项目。

要注册、构建、部署：

1. 如果需要，请检查你是否仍在项目目录的根目录中。
2. 通过运行以下命令来注册、构建和部署`dfx.json`中指定的罐头：
    ``` bash
    dfx deploy
    ```
    `dfx deploy`命令输出显示有关它执行的每个操作的信息，类似与以下摘录：
    ``` text
    Creating a wallet canister on the local network.
    The wallet canister on the "local" network for user "default" is "rwlgt-iiaaa-aaaaa-aaaaa-cai"
    Deploying all canisters.
    Creating canisters...
    Creating canister "rust_counter"...
    "rust_counter" canister created with canister id: "rrkah-fqaaa-aaaaa-aaaaq-cai"
    Creating canister "rust_counter_assets"...
    "rust_counter_assets" canister created with canister id: "ryjl3-tyaaa-aaaaa-aaaba-cai"
    Building canisters...
    Executing: "cargo" "build" "--target" "wasm32-unknown-unknown" "--release" "-p" "rust_counter"
    ...
        Finished release [optimized] target(s) in 53.36s
    Building frontend...
    Installing canisters...
    Creating UI canister on the local network.
    The UI canister on the "local" network is "r7inp-6aaaa-aaaaa-aaabq-cai"
    Installing code for canister rust_counter, with canister_id rrkah-fqaaa-aaaaa-aaaaq-cai
    ...
    Deployed canisters.
    ```

## 调用函数和测试`dapp`（Call functions and test the dapp）

-----

成功部署罐头后，你可以通过调用罐头提供的功能来测试罐头。对于本教程：

- 调用`get`函数查询计数器的值。
- 每次调用`increment`函数来递增计数器值。
- 调用`set`函数以传递参数将计数器更新为你指定的任意值。

测试`dapp`：

1. 通过运行以下命令调用`get`函数以读取`counter`变量的当前值：
    ``` bash
    dfx canister call rust_counter get
    ```
    该命令将返回`counter`当前值返回：
    ``` text
    (0 : nat)
    ```
2. 调用`increment`函数将`counter`变量的值加一：
    ``` bash
    dfx canister call rust_counter increment
    ```
    这个命令增加变量的值————改变它的状态————但不返回结果。
3. 重新运行命令调用`get`函数来查看`counter`变量的当前值：
    ``` bash
    dfx canister call rust_counter get
    ```
    该命令将`counter`的当前值返回：
    ``` text
    (1 : nat)
    ```
4. 运行其它命令来试验调用函数并使用不同的值。例如，尝试类似以下的命令设置和返回计数器值：
    ``` bash
    dfx canister call rust_counter set '(987)'
    dfx canister call rust_counter get
    ```
    返回`987`；
    ``` text
    dfx canister call rust_counter increment
    dfx canister call rust_counter get
    ```
    返回`988`；

## 停止本地罐头执行环境（Stop the local canister execution environment）

----

完成实验后，你可以停止本地罐头执行环境，使其不在后台持续运行。

要停止本地罐头执行环境：

- 在显示网络操作的终端中，按下`ctrl-C`可终端本地罐头执行环境。
- 通过运行以下命令停止本地罐头执行环境：
    ``` bash
    dfx stop
    ```