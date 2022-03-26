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

在前面的示例中，`divMod`方法的签名包括参数名称和结果值。为方法命名参数或结果存粹是出于文档的目的。你使用的名称不会更改方法的类型或传递的值。相反，参数和结果由它们的位置标识，与名称无关。

特别是，`candid`不会阻止你将类型更改为：

``` candid
divMod : (dividend : nat, divisor : nat) -> (mod : nat, div : nat);
```

或将上述`divMod`传递给期望首先返回`mod`方法的服务。

因此，这与语义相关的命名记录字段非常不同。

## 重用复杂类型（Reusing complex types）

-----

通常，服务中的多个方法可能引用同一个复杂类型。在这种情况下，可以多次命名和重用该类型。例如：

``` candid
type address = record {
  street : text;
  city : text;
  zip_code : nat;
  country : text;
};
service address_book : {
  set_address: (name : text, addr : address) -> ();
  get_address: (name : text) -> (opt address) query;
}
```

这些类型定义只是缩写现有类型，它们没有定义新的类型。无论你是在函数签名中使用地址还是写出记录都没有关系。此外，两个名字不同但定义相同的缩写描述了相同的类型并且可以互换。换句话说，`candid`使用结构类型。

## 指定一个`query`方法（Specifying a query method）

-----

在最后一个示例中，你可能已经注意到`get_address`方法使用了查询注释。例如：

``` candid
service address_book : {
  set_address: (name : text, addr : address) -> ();
  get_address: (name : text) -> (opt address) query;
}
```

此注释表明`get_address`方法可以作为`IC`查询调用来调用。如查询和更新方法中所述，查询提供了一种无需经过共识即可从罐头中检索信息的有效方法，因此能够将方法识别为查询是使用`candid`与`IC`。

## 编码和解码（Edcoding and decoding）

-----

`Candid`的要点是允许无缝调用服务方法，传递编码为二进制格式的参数并通过底层传输方法（例如进入或在`IC`内的消息）传输，并在另一端进行解码。

作为`candid`用户，你不必担心这种二进制格式的细节。如果你计划自己实现`Candid`（例如，支持一种新的语言），你可以查阅`candid`规范以获取详细信息。但是，格式的某些方面值得了解：

- `candid`二进制以`DIDL...`开头（或者，在十六进制中，`4449444c...`。如果你在一些低级日志输出中看到这一点，你很可能会观察到一个`candid`编码的值。
- `candid`二进制格式总是对值序列进行编码，因为方法参数和结果是类型序列。
- 二进制非常紧凑。具有`125000`个条目的`(vec nat64)`占用`1000007`个字节。
- 二进制是自描述的，并且包括其中值的类型的（压缩的）类型描述。这允许接收方检测消息是否以不同的、不兼容的类型发送。
- 只要发送方将参数序列化为接收方期望的类型，反序列化就会成功。

## 服务升级（Service upgrades）

-----

服务随着时间的推移而发展：它们获得了新的方法，现有的方法返回更多的数据，或者期待额外的参数。通常，开发人员希望在不破坏现有客户端的情况下做到这一点。

`Candid`通过定义精确的规则来支持这种演变，这些规则指示新服务类型何时仍能够与使用先前接口描述的所有其它方进行通信。潜在的形式主义是子类型化。

服务可以通过以下方式安全地演化：

- 可以添加新方法。
- 现有方法可以返回附加值，即可以扩展结果类型的序列。老客户端只会忽略附加值。
- 现有方法可以缩短其参数列表。老客户端可能仍会发送额外的参数，但它们将被忽略。
- 现有方法可以使用可选参数（类型`opt...`）扩展其参数列表。从不传递该参数的老客户端读取消息时，返回`null`。
- 现有参数类型可以更改，但只能更改为先前类型的超类型。
- 现有结果类型可能会更改，但只能更改为先前类型的子类型。

有关给定类型的超类型和子类型的信息，请参阅该类型的相应参考部分。

让我们看一个服务如何发展的具体例子。考虑一个具有以下`API`的服务：

``` candid
service counter : {
  add : (nat) -> ();
  subtract : (nat) -> ();
  get : () -> (int) query;
  subscribe : (func (int) -> ()) -> ();
}
```

该服务可以演化为如下接口：

``` candid
type timestamp = nat;
service counter : {
  set : (nat) -> ();
  add : (int) -> (new_val : nat);
  subtract : (nat, trap_on_underflow : opt bool) -> (new_val : nat);
  get : () -> (nat, last_change : timestamp) query;
  subscribe : (func (nat) -> (unregister : opt bool)) -> ();
}
```

## 文本值（Candid textual values）

------

`Candid`的主要目的是将使用某种宿主语言（例如`Motoko`、`Rust`或`Javascript`）编写的程序与`IC`连接起来。因此，在大多数情况下，你不必将程序数据作为`Candid`值处理。相反，你使用熟悉的`javascript`值使用`javascript`之类的宿主语言，然后依靠`Candid`将这些值透明地传输到用`Rust`或`Motoko`编写的罐头。接收这些值的罐头将它们视为原生`Rust`或`Motoko`值。

但是，在某些情况下（例如，在命令行上记录、调试、与服务交互时）直接以人类可读的形式查看`Candid`的值很有用。在这些情况下，你可以使用`Candid`值的文本表示。

语法类似于`Candid`类型的语法。例如，一个`Candid`值的典型文本表示可能如下所示：

``` candid
(record {
  first_name = "John";
  last_name = "Doe";
  age = 14;
  membership_status = variant { active };
  email_addresses =
    vec { "john@doe.com"; "john.doe@example.com" };
})
```

`Candid`二进制格式不包括实际的字段名称，仅包括数字哈希。因此，在不知道预期类型的情况下漂亮地打印这样的值将不包括记录和变体的字段名称。然后可能会按如下方式打印上述值：

``` candid
(record {
   4846783 = 14;
   456245371 = variant {373703110};
   1443915007 = vec {"john@doe.com"; "john.doe@example.com"};
   2797692922 = "John"; 3046132756 = "Doe"
})
```

## 生成服务描述（Generating service descriptions）

------

在上面的部分中，你学习了如何从头开始编写`Candid`服务描述。但通常，这甚至不需要！根据你用于实现服务的语言，你可以获得从你的代码生成的`candid`服务的描述。

例如，在`Motoko`中，你可以像这样编写罐头：

``` motoko
actor {
  var v : Int = 0;
  public func add(d : Nat) : async () { v += d; };
  public func subtract(d : Nat) : async () { v -= d; };
  public query func get() : async Int { v };
  public func subscribe(handler : func (Int) -> async ()) { … }
}
```

当你编译这个程序时，`Motoko`编译器会自动生成一个`Candid`服务描述文件，接口如上。

在其它语言中，例如`Rust`或`C`，你仍然可以使用该语言的本地类型来开发你的服务，例如，使用本地`Rust`类型，但是，在使用`Rust`之类的语言开发服务后，目前无法在`Candid`中自动生成服务描述。因此，如果你使用`Rust`或`C`为服务编写程序，则需要按照`Candid`规范中描述的约定手动编写`Candid`接口描述。

有关如何为`Rust`程序编写`Candid`服务描述的示例，请参阅[Rust CDK示例](https://github.com/dfinity/cdk-rs/tree/next/examples)和[Rust教程](https://smartcontracts.org/docs/rust-guide/rust-intro.html)。

无论你使用哪种宿主语言，了解宿主语言类型和`Candid`类型之间的映射都很重要。在支持的类型参考部分，你将找到针对`Motoko`、`Rust`、`Javascript`描述的`Candid`类型映射。