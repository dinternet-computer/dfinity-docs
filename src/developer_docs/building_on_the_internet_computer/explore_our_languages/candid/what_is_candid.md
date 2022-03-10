# `candid`是什么

`candid`是一门语言无关的接口描述语言。它的主要目的是描述服务的公共接口，通常以程序的形式部署为在`IC`上运行的罐头。`candid`的主要优点之一是它与语言无关，并允许使用不同的编程语言（包括`Motoko`、`Rust`、`Javascript`）编写的服务和前端之间互相操作。

典型的`candid`描述如下：

``` candid
service counter : {
  add : (nat) -> ();
  subtract : (nat) -> ();
  get : () -> (int) query;
  subscribe : (func (int) -> ()) -> ();
}
```

在此示例中，所描述的服务————`counter`————由以下公共方法组成：

- `add`和`subtract`改变`counter`的当前值。
- `subscribe`方法可用于调用另一个函数，例如每次`counter`值更改时调用通知回调方法。

如本例所示，每个方法都有一系列参数和结果类型。方法还可以包括特定于`IC`的注释（如本例中所示的`query`）。

鉴于这个简单的接口描述，你可以直接从命令行或通过基于`web`的前端用户界面或从`rust`程序或通过其它编程或脚本语言以编程方式与此`counter`服务交互。

除了互操作性之外，`candid`通过精确指定可以在不破坏现有客户端的情况下进行的更改来支持服务接口的升级。例如，你可以安全地向服务添加新的可选参数，而不会丢失对现有客户端的兼容性。

## 为什么我们要发明一门新的`IDL`（Why create a new IDL）？

------

乍一看，你可能认为其它技术（例如`json`、`xml`、`protobuf`）就足够用了。但是，`candid`提供了这些其它技术中没有的独特功能组合。使得`candid`特别适合为`IC`开发去中心化应用：

- `JSON`、`XML`、`protobuf`等许多语言只描述了如何将单个值映射到字节或字符。这些数据描述语言并没有将服务作为一个整体来描述。这些语言专注于你要传输的数据类型，而不是使用这些数据类型的方法。
- `candid`实现将`candid`值直接映射到宿主语言的类型和值。使用`candid`，开发者无需构建或解构一些抽象的`candid`值。
- `candid`定义了如何以合理和组合的方式升级服务及其接口的规则。
- `candid`本质上是一种高级语言。使用`candid`，你可以传递的不仅仅是普通数据，包括对服务和方法的引用。对安全升级的支持考虑到了这种高阶使用。
- `candid`内置了对`IC`的支持，例如`query`注释。

## `candid`的类型和值（Candid types and values）

-----

`candid`是一种强类型系统，具有一组规范地涵盖大多数用途的类型系统。它有：

- 无界整数（`nat`、`int`）。
- 有界整数（`nat8`、`nat16`、`nat32`、`nat64`、`int8`、`int16`、`int32`、`int64`）。
- 浮点数（`float32`、`float64`）。
- 布尔值（`bool`）。
- 文本（`text`）和二进制（`blob`）。
- 容器类型，包括`opt`、`vec`、`record`、`variant`。
- 应用类型（`service`、`func`、`principal`）。
- 特殊的`null`、`reserved`、`empty`类型。

[参考手册](https://smartcontracts.org/docs/candid-guide/candid-ref.html)详细描述了所有类型。

这组类型背后的理念是它们足以描述数据的结构，因此可以对信息进行编码、传递、解码，但故意不描述超出描述表示所需的语义约束。例如，没有办法表示一个数字应该是偶数，一个向量有一定的长度，或者一个向量的元素是有序的。

`candid`支持这组类型允许基于适用于每种宿主语言的合理、规范选择对数据类型进行自然映射，无论你是使用`motoko`、`rust`、`javascript`还是其它编程语言写代码。

## `candid`服务描述（Candid service descriptions）

------

一旦熟悉了`candid`类型，就可以使用它们来描述服务。`candid`服务描述文件（一个`.did`文件）既可以手写，也可以从服务实现中生成。

在探索如何为特定宿主语言生成服务描述之前，让我们仔细看看示例服务描述的结构及其组成部分。

最简单的服务描述指定一个没有公共方法的服务，看起来像这样：

``` candid
service : {}
```

这个服务不是很有用，所以让我们添加一个简单的`ping`方法：

``` candid
service : {
  ping : () -> ();
}
```

此示例描述了一个服务，该服务支持称为`ping`的单个公共方法。方法名称可以是任意字符串，如果它们不是普通标识符，你可以引用它们（“带空格的方法”）。

方法声明了一系列参数和返回结果类型。在这种`ping`方法的情况下，不传递任何参数，也不返回任何结果，因此空序列`()`用于参数和结果。

现在你已经看到了最简单的情况，让我们考虑一个稍微复杂一点的服务描述。该服务包含两个方法————`reverse`和`divMod`————每个方法都包含一系列参数和返回结果类型：

``` candid
service : {
  reverse : (text) -> (text);
  divMod : (dividend : nat, divisor : nat) -> (div : nat, mod : nat);
}
```

方法`reverse`需要一个`text`类型的参数并返回一个`text`类型的值。

方法`divMod`期望并返回两个值，都是`nat`类型。

## 具名参数和返回结果（Naming arguments and results）

-----

## 返回复杂的类型（Returning complex types）

-----

## 指定一个`query`方法（Specifying a query method）

-----

## 编码和解码（Edcoding and decoding）

-----

## 服务升级（Service upgrades）

-----

## 文本值（Candid textual values）

------

## 生成服务描述（Generating service descriptions）

------

