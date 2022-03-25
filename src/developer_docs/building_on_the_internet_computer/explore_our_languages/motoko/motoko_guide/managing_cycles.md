# 管理燃料

`IC`的使用是按照单位燃料衡量和付费的。`IC`对每个罐头都维护一个`balance`属性。燃料可以在罐头之间转移。

在`motoko`程序中，每个`actor`都代表一个罐头，罐头有一定量的燃料。燃料的所有权可以在罐头之间转移。燃料通过消息选择性地发送和接收，即共享函数调用。调用者可以选择通过调用传输燃料，而被调用者可以选择接受调用者提供的燃料。除非明确指示，否则调用者不会传输任何燃料或被调用者接受任何燃料。

被调用者可以接受所有、部分、完全不接受燃料，直至达到调用者提供的燃料限制。任何剩余的燃料都将退还给调用者。如果该调用被捕获，其携带的所有燃料都会退还给调用者，没有任何损失。

将来，我们可能会看到`motoko`采用专用的语法和类型来支持更安全的燃料编程。目前，我们通过库中的`ExperimentalCycles`库提供的低级命令式`API`提供了一种临时方法来管理燃料。

> 注意，该库可能会发生变化，并且可能会被更高级别的`motoko`版本中的功能替代。

## `ExperimentalCycles`库

------

`ExperimentalCycles`库提供了用于观察`actor`当前燃料余额、转移燃料和观察燃料退款的命令式操作。

该库提供以下操作：

``` motoko
func balance() : (amount : Nat)

func available() : (amount : Nat)

func accept(amount : Nat) : (accepted : Nat)

func add(amount : Nat) : ()

func refunded() : (amount : Nat)
```

函数`balance()`以金额的形式返回`actor`当前的燃料余额。函数`balance()`是有状态的，在调用`accept(n)`、添加燃料后调用函数或从`await`恢复（反映退款）后可能会返回不同的值。

> 注意，由于燃料余额所花费的计算资源，因此`balance()`的值通常会从一个共享函数调用到下一个共享函数调用之间减小。

函数`available()`，返回当前可用的燃料数。这是从当前调用者收到的金额，减去到目前为止此调用接受的累积金额。在通过`return`或`throw`退出当前共享函数或异步表达式时，任何剩余的可用金额都会自动退还给调用者。

函数接受从`available()`到`balance()`的转账金额。它返回实际转移的金额，该金额可能少于请求的金额，例如，可用的金额较少，或者达到罐头余额限制。

函数`add(amount)`指示要在下一次远程调用中传输的额外燃料量，即评估共享函数调用或异步表达式。在调用时，但不是之前，自上次调用以来添加的总单位数将从`balance()`中扣除。如果这个总数超过`balance()`，调用者会被捕获，终止调用。

> 注意，添加量的隐式寄存器在每次调用时递增，在进入共享函数时以及在每次共享函数调用之后或从等待恢复时重置为零。

函数`refunded()`报告在当前上下文的最后一个等待中退款的燃料数量，如果还没有等待，则返回零。调用`refunded()`仅供参考，不会影响`balance()`。相反，退款会自动添加到当前余额中，无论是否使用`refunded()`。

### 示例

为了说明，我们现在将使用`ExperimentalCycles`库来实现一个玩具存钱罐来保存燃料。

我们的存钱罐有一个隐含的`owner`、一个`benefit`回调和一个固定的`capacity`，所有这些都在创建时就提供好了。回调用于转移提取的金额。

``` motoko
import Cycles "mo:base/ExperimentalCycles";

shared(msg) actor class PiggyBank(
  benefit : shared () -> async (),
  capacity: Nat
  ) {

  let owner = msg.caller;

  var savings = 0;

  public shared(msg) func getSavings() : async Nat {
    assert (msg.caller == owner);
    return savings;
  };

  public func deposit() : async () {
    let amount = Cycles.available();
    let limit : Nat = capacity - savings;
    let acceptable =
      if (amount <= limit) amount
      else limit;
    let accepted = Cycles.accept(acceptable);
    assert (accepted == acceptable);
    savings += acceptable;
  };

  public shared(msg) func withdraw(amount : Nat)
    : async () {
    assert (msg.caller == owner);
    assert (amount <= savings);
    Cycles.add(amount);
    await benefit();
    let refund = Cycles.refunded();
    savings -= amount - refund;
  };

}
```

银行的`owner`由构造函数`PiggyBank()`的（隐式）调用者标识，使用共享模式`shared (msg)`。字段`msg.caller`是一个`Principal`并且存储在私有变量`owner`中（供将来参考）。有关此语法的更多说明，请参阅主体和调用者标识。

存钱罐最初是空的，当前储蓄为零。

只有来自所有者的调用可以执行：

- 查询存钱罐的当前储蓄（函数`getSavings()`），或
- 从储蓄中提取金额（函数（`withdraw(amount)`））。

对调用者的限制由语句`assert(msg.caller == owner)`进行，其失败会导致封闭函数被捕获，而不会显示余额或转移任何燃料。

任何调用者都可以存入一定数量的燃料，前提是存储不会超过容量，从而打破存钱罐。由于存款功能仅接受可用金额的一部分，因此存款超过限制的调用者将收到任何未接受的燃料的隐式退款。退款是自动的，并由`IC`基础设施确保执行。

由于燃料的传输是单向的（从调用者到被调用者），检索燃料需要使用显式回调（`benefit`函数，由构造函数作为参数）。在这里，`withdraw`函数调用了`benefit`，但仅在将调用者验证为所有者之后，在回调中反转调用者和被调用者的关系，允许燃料“上游”流动。

请注意，`PiggkBank`的所有者实际上可以提供一个回调来奖励不同于所有者的受益人。

下面是所有者`Alice`可能如何使用`PiggyBank`的实例：

``` motoko
import Cycles = "mo:base/ExperimentalCycles";
import Lib = "PiggyBank";

actor Alice {

  public func test() : async () {

    Cycles.add(10_000_000_000_000);
    let porky = await Lib.PiggyBank(Alice.credit, 1_000_000_000);

    assert (0 == (await porky.getSavings()));

    Cycles.add(1_000_000);
    await porky.deposit();
    assert (1_000_000 == (await porky.getSavings()));

    await porky.withdraw(500_000);
    assert (500_000 == (await porky.getSavings()));

    await porky.withdraw(500_000);
    assert (0 == (await porky.getSavings()));

    Cycles.add(2_000_000_000);
    await porky.deposit();
    let refund = Cycles.refunded();
    assert (1_000_000_000 == refund);
    assert (1_000_000_000 == (await porky.getSavings()));

  };

  // Callback for accepting cycles from PiggyBank
  public func credit() : async () {
    let available = Cycles.available();
    let accepted = Cycles.accept(available);
    assert (accepted == available);
  }

}
```

让我们剖析`Alice`的代码：

`Alice`将`PiggyBank`的`actor`类作为库导入，因此她可以按需创建新的`PiggyBank`的`actor`。

大部分操作发生在`Alice`的`test()`函数中：

`Alice`通过调用`Cycles.add(10_000_000_000_000)`将自己的`10_000_000_000_000`个单位的燃料专用于运行储钱罐，然后创建`PiggyBank`的新实例`porky`之前，传递回调`Alice.credit`和容量`(1_000_000_000)`。通过`Alice.credit`指定`Alice`作为提款的受益人。这`10_000_000_000_000`个单位的燃料，减去一小笔安装费，将记入`pigky`的余额，而无需通过`pigky`初始化代码进行任何进一步的操作。你可以把它想象成一个电动存钱罐，它消耗自己的资源。由于构建`PiggyBank`是异步的，`Alice`需要等待结果。

创建`porky`之后，她首先使用断言验证`porky.getSavings()`是否为零。

`Alice`将她的`1_000_000`个单位的燃料`Cycles.add(1_000_000)`用于在下一次调用`porky.deposit()`时转移到`porky`。仅当对`porky.deposit()`的调用成功（它应该）时，才会从`Alice`的余额中扣除这些燃料。

`Alice`现在提取了一半的金额，即`500_000`，并验证了`pigky`的储蓄减少了一半。`Alice`最终通过对`Alice.credit()`的回调来接收燃料，该回调在`porky.withdraw()`中启动。请注意，接收到的燃料正是在`pigky.withdraw()`中添加的燃料，在它调用其它`benifit`回调之前，即`Alice.credit`。