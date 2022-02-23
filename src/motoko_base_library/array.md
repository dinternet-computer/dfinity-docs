# Array

**Array 上的函数：**

## equal

``` motoko
func equal<A>(a : [A], b : [A], eq : (A, A) -> Bool) : Bool
```

测试是否两个`array`包含相同的值。

## append

``` motoko
func append<A>(xs : [A], ys : [A]) : [A]
```

连接两个给定的`array`。`Array.append`有非常严重的性能问题，已被废弃，请使用`Buffer.append`。

## sort

``` motoko 
func sort<A>(xs : [A], cmp : (A, A) -> Order.Order) : [A]
```

通过`cmp`函数排序给定的`array`。这是一个稳定的排序。

``` motoko
import Array "mo:base/Array";
import Nat "mo:base/Nat";

let xs = [4, 2, 6, 1, 5];
assert(Array.sort(xs, Nat.compare) == [1, 2, 4, 5, 6])
```

## sortInPlace

``` motoko
func sortInPlace<A>(xs : [var A], cmp : (A, A) -> Order.Order)
```

通过`cmp`函数对给定`array`进行原地排序。这是一个稳定的排序。

``` motoko
import Array "mo:base/Array";
import Nat "mo:base/Nat";

let xs : [var Nat] = [4, 2, 6, 1, 5];
Array.sortInPlace(xs, Nat.compare);
assert(Array.freeze(xs) == [1, 2, 4, 5, 6])
```

## chain

``` motoko 
func chain<A, B>(xs : [A], f : A -> [B]) : [B]
```

## filter

``` motoko 
func filter<A>(xs : [A], f : A -> Bool) : [A]
```

## mapFilter

``` motoko 
func mapFilter<A, B>(xs : [A], f : A -> ?B) : [B]
```