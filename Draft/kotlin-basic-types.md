---
title: kotlin 基本类型
date: 2017-07-30 11:49:24
tags: kotlin
---

在 Kotlin 中，所有东西都是对象。  

## 数字
数字没有隐式拓宽转换（如 Java 中 int 可以隐式转换为long——译者注)，另外有些情况的字面值略有不同。

|Type	|Bit width|
|:-----------------|:--------------------:|
|Double|	64|
|Float|	32|
|Long|	64|
|Int|	32|
|Short|	16|
|Byte|	8|


## 字面常量

数值常量字面值有以下几种:

十进制: 123
Long 类型用大写 L 标记: 123L
十六进制: 0x0F
二进制: 0b00001011

> 注意: 不支持八进制

Kotlin 同样支持浮点数的常规表示方法:

默认 double：123.5、123.5e10
Float 用 f 或者 F 标记: 123.5f

## 数字字面值中的下划线（自 1.1 起）

你可以使用下划线使数字常量更易读：

```
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val socialSecurityNumber = 999_99_9999L
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010
```

## 表示方式

在 Java 平台数字是物理存储为 JVM 的原生类型，除非我们需要一个可空的引用（如 Int?）或泛型。 后者情况下会把数字装箱。

注意数字装箱不必保留同一性:

```
val a: Int = 10000
print(a === a) // 输出“true”
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA === anotherBoxedA) // ！！！输出“false”！！！
```

对于数据相等的判断在kotlin中有===和==，它们的区别是什么呢？==判断是两个数据的值是否相等；但是===则判定的两个数据是否是同一个对象。也就是说===的判断条件更加苛刻一些。

```
val a: Int = 10000
print(a == a) // 输出“true”
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA == anotherBoxedA) // 输出“true”
print(boxedA === anotherBoxedA) // 输出“false”
```

## 显式转换
较小类型并不是较大类型的子类型，较小的类型不能隐式转换为较大的类型

```
val b: Byte = 1 // OK, 字面值是静态检测的
val i: Int = b // 错误
```

我们可以显式转换来拓宽数字

```
val i: Int = b.toInt() // OK: 显式拓宽
```

缺乏隐式类型转换并不显著，因为类型会从上下文推断出来，而算术运算会有重载做适当转换，例如：

```
val l = 1L + 3 // Long + Int => Long
```


## 字符

字符用 Char 类型表示。它们不能直接当作数字
```
fun check(c: Char) {
    if (c == 1) { // 错误：类型不兼容
        // ……
    }
}
```

字符字面值用单引号括起来: '1'。 特殊字符可以用反斜杠转义。 支持这几个转义序列：\t、 \b、\n、\r、\'、\"、\\ 和 \$。 编码其他字符要用 Unicode 转义序列语法：'\uFF00'。

我们可以显式把字符转换为 Int 数字：

```
fun decimalDigitValue(c: Char): Int {
    if (c !in '0'..'9')
        throw IllegalArgumentException("Out of range")
    return c.toInt() - '0'.toInt() // 显式转换为数字
}
```

当需要可空引用时，像数字、字符会被装箱。装箱操作不会保留同一性

## 数组

```
// 创建一个 Array<String> 初始化为 ["0", "1", "4", "9", "16"]
val asc = Array(5, { i -> (i * i).toString() })
```

> 注意: 与 Java 不同的是，Kotlin 中数组是不型变的（invariant）。这意味着 Kotlin 不让我们把 Array<String> 赋值给 Array<Any>，以防止可能的运行时失败（但是你可以使用 Array<out Any>, 参见类型投影）。

## 字符串

字符串的元素——字符可以使用索引运算符访问: s[i]

```
for (c in str) {
    println(c)
}
```

原生字符串 使用三个引号（"""）分界符括起来，内部没有转义并且可以包含换行和任何其他字符:

```
val text = """
    for (c in "foo")
        print(c)
"""
```


你可以通过 trimMargin() 函数去除前导空格：

```
val text = """
    |Tell me and I forget.
    |Teach me and I remember.
    |Involve me and I learn.
    |(Benjamin Franklin)
    """.trimMargin()
```

默认 | 用作边界前缀，但你可以选择其他字符并作为参数传入，比如 trimMargin(">")。


原生字符串和转义字符串内部都支持模板。 如果你需要在原生字符串中表示字面值 $ 字符（它不支持反斜杠转义），你可以用下列语法：

```
val price = """
	${'$'}9.99
	"""
```

> 但是经过代码运行和下面结果一样都是 $9.99

```
val price  = """
    $9.99
    """
```






