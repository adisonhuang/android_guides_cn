# 使用Kotlin进行Android开发

## 什么是Kotlin？

[Kotlin](https://kotlinlang.org/)是由[JetBrains](https://www.jetbrains.com/)开发的一种语言，后者是Android Studio基于的[IntelliJ IDEA](https://www.jetbrains.com/idea)背后的公司。Kotlin专门为大规模软件项目而构建，旨在改善Java的可读性，准确性和提高开发人员生产力。

该语言是针对 Java 的限制而创建的, 这些限制阻碍了 JetBrains 软件产品的开发, 并且在对所有其他 JVM 语言进行了评估后证明它们并不适合。由于Kotlin的目标是用于改进产品，所以它非常强调与Java代码和Java标准库的互操作性。

## 为什么Kotlin？

- 100％与Java可互操作 - Kotlin和Java可以共存于一个项目中。您可以继续使用Java中的现有库。

- 简洁 - 大大减少您需要编写的模板代码的数量。

- 安全 - 避免类型错误，如空指针异常。

- 支持函数式编程- Kotlin使用函数式编程的许多概念，例如lambda表达式。

## 语法概述

### 变量

定义局部变量

只读本地变量：

```kotlin
val a: Int = 1
val b = 1   // `Int` 类型会自动推断
```

可变变量

```kotlin
var x = 5 // `Int` 类型会自动推断
x += 1
```

### 函数

传递两个Int参数并返回Int的的函数：

```kotlin
fun sum(a: Int, b: Int) :Int {
	return a + b
}
```

上面函数也可以写成表达式形式

```kotlin
fun sum(a: Int, b: Int) = a + b
```

返回空的函数
```kotlin
fun printSum(a: Int, b: Int): Unit {
  print(a + b)
}
```

Unit 返回类型可以忽略

```kotlin
fun printSum(a: Int, b: Int) {
  print(a + b)
}
```

### 使用集合

迭代集合：

```kotlin
for (name in names)
  println(name)
```

检查集合是否包含使用某个对象：

```kotlin
if (text in names) // names.contains(text) is called
  print("Yes")
```

使用lambda表达式过滤和映射集合：

```kotlin
names
    .filter { it.startsWith("A") }
    .sortedBy { it }
    .map { it.toUpperCase() }
    .forEach { print(it) }
```

### 空安全

```kotlin
val x: String? = "Hi"
x.length // 编译失败
val y: String = null // 编译失败
```

处理null

```kotlin
// 使用安全操作符 ?.
x?.length // 如果 x 不为空，这会返回x.length，返回返回 null

// 二元云算符 ?:
val len = x?.length ?: -1 // 如果 x 为空，这会返回-1 
```





