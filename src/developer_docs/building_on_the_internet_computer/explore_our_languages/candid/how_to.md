# 怎么做

正如[什么是`Candid`](./what_is_candid.md)中讨论的那样，`Candid`提供了一种语言无关的方式去与罐头交互。通过使用`Candid`，你能够指定服务的输入参数并显示罐头服务的返回值，无论你是使用`dfx`命令行界面从终端、通过`Web`浏览器还是从编写的`Javascript`、`Motoko`、`Rust`或其它任何语言程序与`IC`交互。现在你已经熟悉了`Candid`是什么以及它是如何工作的，本节将提供有关如何在一些常见场景中使用它的说明。

作为一个具体示例，假设`IC`上已经部署了一个计数器罐头，具有以下`candid`接口：

``` candid
service Counter : {
  inc : (step: nat) -> (nat);
}
```

现在，让我们在`candid`的帮助下探索如何在各种场景与该罐头交互。

## 从终端与一个罐头交互（Interact with a service in a terminal）

------

和罐头或`IC`交互的最常见的方法是通过`dfx`命令行工具。

`dfx`提供了`dfx canister call`命令用来调用一个已经部署的罐头的方法。

当你使用`dfx canister call`命令时，你能够像[`Candid`值的文本表示](https://smartcontracts.org/docs/candid-guide/candid-concepts.html#textual-values)中描述的那样传递参数。

当你在命令行中传递多个参数给`dfx canister call`时，你能够通过下小括号包裹值并用逗号分割的形式来传递，例如`(42, true)`表示传递两个参数，第一个参数是`42`，第二个参数是`true`。

以下示例说明如何使用`dfx canister call`命令调用计数器罐头的`inc`方法并传递参数：

``` bash
dfx canister call counter inc '(42)'
```

你也能够省略参数，让`dfx`生成与方法类型匹配的随机值。例如：

``` bash
dfx canister call counter inc

Unspecified argument, sending the following random argument:
(1_543_454_453)

(1_543_454_454)
```

## 从浏览器与一个罐头交互（Interact with service from a browser）

-----

`Candid`接口描述语言为指定罐头的签名提供了一种通用语言。基于罐头提供的服务的类型签名，`Candid`提供了一个`Web`界面——`Candid UI`——允许你从`Web`浏览器调用罐头函数进行测试和调试，而无需编写任何前端代码。

要使用`Candid`的`Web`测试界面测试`counter`罐头：

1. 使用`dfx canister id __Candid_UI`命令查找与计数器罐头相关的`Candid UI`罐头的标识符。
    ``` bash
    dfx canister id __Candid_UI
    ```
    该命令显示`Candid UI`的罐头标识符，输出类似于以下内容：
    ``` bash
    r7inp-6aaaa-aaaaa-aaabq-cai
    ```
2. 打开本地罐头执行环境：
    ``` bash
    dfx start --background
    ``` 
3. 打开浏览器并导航到`dfx.json`配置文件中指定的地址和端口号。
    默认情况下，本地罐头执行环境绑定到`127.0.0.1:8000`地址和端口。
4. 添加所需的`canisterId`参数和`dfx canister id`命令返回的`Candid UI`罐头的标识符。
    例如，完整的`URL`应类似于以下内容，但带有`dfx canister id`命令返回的`CANISTER-UI-CANISTER-IDENTIFIER`：
    ``` bash
    http://127.0.0.1:8000/?canisterId=<CANDID-UI-CANISTER-IDENTIFIER>
    ```
    浏览器会显示一个表单，供你指定罐头标识符或选择`candid`描述文件。
    如果你不确定要使用哪个罐头标识符，你可以运行`dfx canister id`命令查看特定罐头名称的标识符。
    例如，如果要查看计数器罐头，可以通过运行以下命令来查找罐头标识符：
    ``` bash
    dfx canister id counter
    ```
5. 指定罐头标识符或描述文件，然后单击执行以显示服务描述。
6. 查看程序中定义的函数调用和类型列表。
7. 为函数键入适当类型的值或单击`Random`生成一个值，然后单击`call`或`query`以查看结果。

有关任何罐头的`candid`界面创建`web`界面的工具的更多信息，请参阅[`Candid UI`仓库](https://github.com/dfinity/candid/tree/master/tools/ui)。

## 从`Motoko`罐头与其它罐头交互（Interact with service from a Motoko canister）

------

如果你在`Motoko`中编写罐头，`Motoko`编译器会自动将罐头的顶级`Actor`或`Actor`类的签名转换为`candid`描述，并且`dfx build`命令确保在需要的地方正确引用服务。

例如，如果你想写一个`hello`罐头，调用`motoko`中的`counter`罐头：

``` motoko
import Counter "canister:Counter";
import Nat "mo:base/Nat";
actor {
  public func greet() : async Text {
    let result = await Counter.inc(1);
    "The current counter is " # Nat.toText(result)
  };
}
```

在这个例子中，当计数器罐头的导入依赖项（导入计数器`canister:Counter`声明）由`dfx build`命令处理时，`dfx build`命令确保将计数器罐头标识符和`candid`描述传递给`motoko`编译器。然后`motoko`编译器将`candid`类型转换为适当的原生`motoko`类型。这种翻译使你能够调用本地的`inc`方法——就好像它是一个`motoko`函数一样——即使计数器罐头是用不同语言实现的，即使你没有导入该罐头的源代码。有关`candid`和`motoko`之间的类型映射，你可以参考[这里](https://smartcontracts.org/docs/candid-guide/candid-types.html)。

## 从`Rust`编写的罐头与其它罐头交互（Interact with service from a Rust canister）

-----

## 通过`javascript`与罐头交互（Interact with a service from Javascript）

-----

## 创建一个新的`Candid`实现（Create a new Candid implementation）