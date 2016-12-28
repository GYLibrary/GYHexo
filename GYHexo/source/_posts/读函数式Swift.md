---
title: 读函数式Swift(一)
date: 2016-04-12 19:35:16
categories: 书籍
tags: [Swift] 
description: 
---
# 函数式Swift

## 引言

> Swift是一门有着合适的语言特性来适配函数式编程方法的优秀语言
> 
> Swift函数式程序应有的特质:
> 
> - 模块化
> - 对可变状态的谨慎处理: 函数式编程又称“面向值编程”。面向对象编程专注于类和对象的设计，“函数式编程强调基于值编程的重要性，这能使我们免受可变状态或其他一些副作用的困扰。通过避免可变状态，函数式程序比其对应的命令式或者面向对象的程序更容易组合。
> - 类型:“Swift 有一个强大的类型系统，使用得当的话，它能够让你的代码更加安全和健壮。”

## 可选值
- 可选值绑定机制(optional binding可避免写!)

   ```
  if let madridPopulation = cities["Madrid"] {
	print("The population of Madrid is \(madridPopulation * 1000)")
	} else {
	print("Unknown city: Madrid")
	}”

   ```
 
   
- nil运算符(??)
 
 	```
 	//无论是否nil都会计算defalut的值
 	value ?? defalut
 	```
 	为防止value不为nil计算defalut导致无畏消耗
 	
 	```
 infix operator ?? { associativity right precedence 	110 }
func ??<T>(optional: T?, @autoclosure defaultValue: () 	-> T) -> T {
	if let x = optional {
	return x
	} else {
	return defaultValue()
	}
}

 	```
“Swift 的 autoclosure 类型标签来避开创建显式闭包的需求。“它会在所需要的闭包中隐式地将参数封装到 ?? 运算符。 ”

- 可选值检查

	```
extension Optional {
func flatMap<U>(f: Wrapped -> U?) -> U? {
guard let x = self else { return nil }
return f(x)
}
}
	```
借力于标准flatMap函数检查一个可选值是否为nil

- 为什么使用可选值
> 因为显示的可选类型更符合Swift增强静态安全的特性，有助于避免因为缺失值导致意外崩溃

## 不可变性的价值

### 值类型与引用类型

Swift类型分为**值类型和引用类型**。典型例子：结构体和类

定义下述结构体：

```
struct PointStruct {
var x: Int
var y: Int
}

var structPoint = PointStruct(x: 1, y: 2)
var sameStructPoint = structPoint
sameStructPoint.x = 3”

```
在执行这段代码之后,很明显 sameStructPoint 等于 (x: 3, y: 2)。然而 structPoint 仍然保持原始值。这就是值类型与引用类型之间的关键区别：**当被赋以一个新值或是作为参数传递给函数时，值类型会被复制**。之所以给 sameStructPoint.x 赋值并不更新原来的 structPoint，正是因为先前的 sameStructPoint = structPoint 赋值过程中，发生了**值复制**。[Andy Matuschak](https://objccn.io/issue-16-2/)对值类型和引用类型之间的区别给出了直观的例子。

## 枚举





