# 支持类型（Supported types）

## 文本（Type `text`）

-----

## 二进制（Type `blob`）

-----

## 自然数（Type `nat`）

-----

## 整数（Type `int`）

-----

## 有限精度自然数和有限精度整数（Type `natN` and `intN`）

-----

## `32`为浮点数和`64`为浮点数（Type `float32` and `float64`）

-----

## 布尔值（Type `bool`）

-----

## 空值（Type `null`）

-----

## 向量（Type `vec` t）

-----

## 可空值（Type `opt` t）

-----

类型`opt t`包含类型`t`的所有值，在加上类型`null`。`opt t`用来表示某个值是可空的，意味着该值为空的时候的类型是`t`，该值为空的时候是`null`。

`opt`是可以嵌套使用的（例如`opt opt text`），然后，`null`和`opt null`是明显不同的类型。

`opt`类型在`candid`接口的演变中起着至关重要的作用，并且具有如下所述的特殊子类型规则。

**类型语法**：

`opt bool`、`opt nat8`、`opt opt text`等等。

**文本语法**：

``` candid
null 
opt true
opt 8
opt null
opt opt "test"
```

**子类型**：

使用`opt`进行子类型化的规范规则是：

- 只要`t`是`t'`的子类型，那么`opt t`就是`opt t'`的子类型。
- `null`是`opt t`的子类型。
- `t`是`opt t`的子类型（除非`t`本身是`null`、`opt ...`、`reserved`）。

此外，出于与升级和高阶服务相关的技术原因，每个类型都是`opt t`子类型，如果类型不匹配则返回`null`。但是，建议用户不要直接使用该规则。

**超类型**：

- 只要`t`是`t'`的超类型，那么`opt t`就是`opt t'`的超类型。

**对应`motoko`的类型**：

`?T`：其中`Motoko`类型`T`对应于`t`。

**对应`rust`的类型**：

`Option<T>`：其中`rust`类型`T`对应于`t`。

**对应`javascript`的值**：

`null`被翻译为`[]`。
`opt 8`被翻译为`[8]`。
`opt opt "test"`被翻译为`[["test"]]`。

## `record`（Type `record{ n : t, ... }`）

------

## `variant`（Type `variant{ n : t, ... }`）

------

`variant`类型表示恰好来自给定案例或标签之一的值。所以类型的值：

``` candid
type shape = variant {
  dot : null;
  circle : float64;
  rectangle : record { width : float64; height : float64 };
  "💬" : text;
};
```

要么是一个点，要么是一个圆（带有半径），要么是一个矩形（带有尺寸），要么是一个气泡（带有一些文本）。对话气泡说明了`unicode`标签名称（`💬`）的使用。

`variant`中的标签，就像记录中的标签一样，实际上是数字，字符串标签指的是它们的哈希值。

通常，部分或全部标签不携带数据。然后使用`null`类型是惯用的，如上面的点所示。事实上，`candid`通过允许你在`variant`中省略`: null`类型注释来鼓励这一点，因此：

``` candid
type season = variant { spring; summer; fall; winter }
```

和下面这样是等价的：

``` candid
type season = variant {
  spring : null; summer: null; fall: null; winter : null
}
```

都用于表示枚举。

`variant`类型`{}`是合法的，但没有值。如果这是故意的，则[`empty`类型](#emptytype-empty)可能更合适。

**类型语法**：

``` candid
variant {}
variant { ok : nat; error : text }
variant { "name with spaces" : nat; "unicode, too: ☃" : bool }
variant { spring; summer; fall; winter }
```

**文本语法**：

``` candid
variant { ok = 42 }
variant { "unicode, too: ☃" = true }
variant { fall }
```

**子类型**：

`variant`类型的子类型是删除了一些标签的`variant`类型，并且某些标签的类型本身更改为子类型。

如果你希望能够在方法结果的变体中添加新标签，则可以在`variant`本身包含在`opt ...`中时执行此操作。这需要提前计划！当你设计一个接口时，而不是写：

``` candid
service {
  get_member_status (member_id : nat) -> (variant {active; expired});
}
```

最好使用这个：

``` candid
service {
  get_member_status (member_id : nat) -> (opt variant {active; expired});
}
```

这样，如果你以后需要添加`honorary`的身份，你可以展开身份列表。旧客户端将收到未知字段为`null`。

**超类型**：

`variant`类型的超类型是带有附加标签的`variant`，并且可能具有某些标签的类型更改为超类型。

**相应的`Motoko`的类型**：

`variant`类型表示为`Motoko`的`variant`类型，例如：

``` candid
type Shape = {
  #dot : ();
  #circle : Float;
  #rectangle : { width : Float; height : Float };
  #_2669435721_ : Text;
};
```

请注意，如果标签的类型为`null`，这对于`Motoko`中的`()`，以保留将枚举建模为`variant`的各个惯用方式之间的映射。

**相应的`Rust`的类型**：

使用宏`#[derive(CandidType, Deserialize)]`修饰枚举`enum`。你可以使用`#[serde(rename = "DifferentFieldName")]`来重命名域中的字段。

**相应的`javascript`的类型**：

具有单个条目的对象。例如，`{ dot: null }`。
如果键名是哈希值，我们用`_hash_`替换键名，例如：`{ _32937428_: "test" }`。

## 函数（Type `func (...) -> (...)`）

------

`candid`旨在支持高阶用例，其中服务可以接收或提供对其它服务或其方法的引用，例如作为回调。`func`类型对此至关重要：它指示函数的签名（参数和返回结果类型、注释），并且此类型的值是对具有该签名的函数的引用。

支持的注释是：

- `query`表示引用的函数是一个查询方法，这意味着它不会改变罐头的状态，并且可以使用更便宜的“查询调用”机制来调用它。
- `oneway`表示此函数不返回任何响应，适用于即发即弃的场景。

有关参数命名的更多信息，请参阅[命名参数和返回结果](https://smartcontracts.org/docs/candid-guide/candid-concepts.html#service-naming)。

**类型语法**：

``` candid
func () -> ()
func (text) -> (text)
func (dividend : nat, divisor : nat) -> (div : nat, mod : nat);
func () -> (int) query
func (func (int) -> ()) -> ()
```

**文本语法**：

目前，仅支持由其主体标识的公共服务方法：

``` candid
func "w7x7r-cok77-xa".hello
func "w7x7r-cok77-xa"."☃"
func "aaaaa-aa".create_canister
```

**子类型**：

对函数类型的以下修改将其更改为[服务升级](https://smartcontracts.org/docs/candid-guide/candid-concepts.html#upgrades)规则中讨论的子类型：

- 结果类型列表可以扩展。
- 参数类型列表可能会缩短。
- 参数类型列表可以使用可选参数（类型`opt ...`）进行扩展。
- 现有的参数类型可以更改为超类型！换句话说，函数类型在参数类型中是逆变的。
- 现有结果类型可能会更改为子类型。

**超类型**：

以下对函数类型的修改将其更改为超类型：

- 结果类型列表可能会缩短。
- 结果类型可以使用可选参数进行扩展（类型`opt ...`）。
- 参数类型列表可以扩展。
- 现有的参数类型可以更改为类型！换句话说，函数类型在参数类型中是逆变的。
- 现有的结果类型可以更改为超类型。

**相应的`Motoko`的类型**：

`candid`函数类型对应于共享的`Motoko`函数，结果类型被包裹在`async`中（除非它们用`oneway`注解，那么结果类型就是简单的`()`）。Arguments resp. results become tuples, unless there is exactly one, in which case it is used directly:

``` candid
type F0 = func () -> ();
type F1 = func (text) -> (text);
type F2 = func (text, bool) -> () oneway;
type F3 = func (text) -> () oneway;
type F4 = func () -> (text) query;
```

相应的`motoko`：

``` motoko
type F0 = shared () -> async ();
type F1 = shared Text -> async Text;
type F2 = shared (Text, Bool) -> ();
type F3 = shared (text) -> ();
type F4 = shared query () -> async Text;
```

**相应的`Rust`的类型**：

`candid::IDLValue::Func(Principal, String)`, 请查看[IDLValue](https://docs.rs/candid/0.6.15/candid/parser/value/enum.IDLValue.html)。

**相应的`javascript`的类型**：

`[Principal.fromText("aaaaa-aa"), "create_canister"]`

## 服务（Type `service {...}`）

------

`service`可能不仅要传递对单个函数的引用（使用`func`类型），还要传递对整个服务的引用。在这种情况下，可以使用`candid`类型来声明此类服务的完整接口。

有关`service`类型语法的更多详细信息，请参阅[`candid`服务描述](https://smartcontracts.org/docs/candid-guide/candid-concepts.html#candid-service-descriptions)。

**类型语法**：

``` candid
service {
  add : (nat) -> ();
  subtract : (nat) -> ();
  get : () -> (int) query;
  subscribe : (func (int) -> ()) -> ();
}
```

**文本语法**：

``` candid
service "w7x7r-cok77-xa"
service "zwigo-aiaaa-aaaaa-qaa3a-cai"
service "aaaaa-aa"
```

**子类型**：

`service`类型的子类型是那些可能具有附加方法的`service`类型，其中现有方法的类型更改为子类型。

这与`service upgrade`中的升级规则所讨论的原则完全相同。

**超类型**：

`service`类型的超类型是那些可能删除了某些方法的`service`类型，并且现有方法的类型被更改为超类型。

**相应的`Motoko`的类型**：

`candid`中的`service`类型直接对应于`motoko`中的`actor`类型。

``` candid
actor {
  add : shared Nat -> async ()
  subtract : shared Nat -> async ();
  get : shared query () -> async Int;
  subscribe : shared (shared Int -> async ()) -> async ();
}
```

**相应的`Rust`的类型**：

`candid::IDLValue::Service(Principal)`，请查看[IDLValue](https://docs.rs/candid/0.6.15/candid/parser/value/enum.IDLValue.html)。

**相应的`javascript`的类型**：

`Principal.fromText("aaaaa-aa")`

## 主体（Type `principal`）

------

`IC`使用主体`principal`作为识别罐头、用户和其它实体的通用方案。

**类型语法**：

`principal`

**文本语法**：

``` candid
principal "w7x7r-cok77-xa"
principal "zwigo-aiaaa-aaaaa-qaa3a-cai"
principal "aaaaa-aa"
```

**相应的`Motoko`的类型**：

`Principal`

**相应的`Rust`的类型**：

`candid::Principal`或者`ic_types::Principal`

**相应的`javascript`的类型**：

`Principal.fromText("aaaaa-aa")`

## `reserved`（Type `reserved`）

------

`reserved`是保留一个（无信息）值的类型，并且是所有其它类型的超类型。

`reserved`类型可用于删除方法参数。考虑具有以下签名的方法：

``` candid
service {
  foo : (first_name : text, middle_name : text, last_name : text) -> ()
}
```

并假设你不在关心`middle_name`。尽管`candid`不会阻止你将签名改为：

``` candid
service {
  foo : (first_name : text, last_name : text) -> ()
}
```

这将是灾难性的：如果客户使用旧界面与你交谈，将默认地忽略`last_name`并将`middle_name`作为`last_name`。请记住，方法参数名称只是约定，方法参数由它们的位置标识。

相反，你可以使用：

``` candid
service {
  foo : (first_name : text, middle_name : reserved, last_name : text) -> ()
}
```

表示`foo`曾经接受第二个参数，但你不在关心这一点。

你可以表示采用这种模式来避免这个陷阱，任何预期具有可变参数的函数，或者其参数只能通过位置而不是类型来区分的函数被声明为采用单个记录。例如：

``` candid
service {
  foo : (record { first_name : text; middle_name : text; last_name : text}) -> ()
}
```

现在，将签名更改为：

``` candid
service {
  foo : (record { first_name : text; last_name : text}) -> ()
}
```

做正确的事，你甚至不需要记录被删除的论点。

> 注意，一般来说，不建议从方法中删除参数。通常，最好引入一种省略参数的新方法。

**类型语法**：

`reserved`

**文本语法**：

`reserved`

**子类型**：

所有类型。

**相应的`Motoko`的类型**：

`Any`

**相应的`Rust`的类型**：

`candid::Reserved`

**相应的`javascript`的类型**：

任何值。

## `empty`（Type `empty`）

------

`empty`是没有值的类型，是任何其它类型的子类型。

`empty`的实际用例比较少。它可用于将方法标记为“从不成功返回”。例如：

``` candid
service : {
  always_fails () -> (empty)
}
```

**类型语法**：

`empty`

**文本语法**：

没有，因为这个类型没有值。

**超类型**：

所有类型。

**相应的`Motoko`的类型**：

`None`。

**相应的`Rust`的类型**：

`candid::Empty`。

**相应的`javascript`的类型**：

没有，因为这个类型没有值。