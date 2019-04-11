---
title: kotlin 包和控制流
date: 2017-07-31 21:51:04
tags: kotlin
---

## 包

如果出现名字冲突，可以使用 as 关键字在本地重命名冲突项来消歧义：
```
import foo.Bar // Bar 可访问
import bar.Bar as bBar // bBar 代表“bar.Bar”
```

## 流程控制

### If表达式

在 Kotlin 中， if 是一个表达式，即它会返回一个值。 因此就不需要三元运算符（条件 ? 然后 : 否则），因为普通的 if 就能胜任这个角色。
```
// 传统用法
var max = a
if (a < b) max = b
// With else
var max: Int
if (a > b) {
  max = a
} else {
  max = b
}
// 作为表达式
val max = if (a > b) a else b
```

if 的分支可以是代码块，最后的表达式作为该块的值：
```
val max = if (a > b) {
  print("Choose a")
  a
} else {
  print("Choose b")
  b
}
```

### When 表达式

```
when (x) {
  1 -> print("x == 1")
  2 -> print("x == 2")
  else -> { // 注意这个块
  print("x is neither 1 nor 2")
  }
}
```

多分支需要相同的处理情况

```
when (x) {
  0, 1 -> print("x == 0 or x == 1")
  else -> print("otherwise")
}
```

我们也可以检测一个值在（ in ）或者不在（ !in ）一个区间或者集合中：

```
when (x) {
  in 1..10 -> print("x is in the range")
  in validNumbers -> print("x is valid")
  !in 10..20 -> print("x is outside the range")
  else -> print("none of the above")
}
```

检测一个值是（ is ）或者不是（ !is ）一个特定类型的值

```
fun hasPrefix(x: Any) = when(x) {
  is String -> x.startsWith("prefix")
  else -> false
}
```
when 也可以用来取代 if - else if 链。 如果不提供参数，所有的分支条件都是简单的布尔表达式，而当一个分支的条件为真时则执行该分支：

```
when {
  x.isOdd() -> print("x is odd")
  x.isEven() -> print("x is even")
  else -> print("x is funny")
}
```
