---
layout: post
title: scala 教程
categories: scala
description: scala 程序设计第二版读书笔记
keywords: 程序语言
---

> 此博客为 《 Scala 程序设计 (第 2 版) 》 读书笔记，转载请保留原始地址。 需要更加深入学习建议购买正版图书 [图灵社区](http://www.ituring.com.cn/book/1593)

## 用 scala 简洁有效的完成工作
### 分号
Scala 可以省略分号，多个表达式处于一行时用分号分隔。
### 变量声明
* `val` 声明不可变变量
* `var` 声明可变变量

_可变与不可变是指引用是否可变，不是引用指向的堆内存是否可变，注意不要与 scala 的 `immutable` 与 `mutable` 搞混淆，例如：_
```scala
val array: Array[String] = new Array(5)
array(0) = "hello"        //可以正确运行
array = new Array(2)      //不可以正确运行
```
用 `val` 与 `var` 声明变量必须初始化，当 `val` 与 `var` 用于类的构造函数的参数中的时候，这时候变量是类的一个属性。 `val` 属性不可变， `var` 属性可变。为了减少可变性引起的 bug ，应该尽可能少使用 `var` 。
#### 偏函数
```scala
val pf: PartialFunction[String, String] = {
  case "one" => "ONE"
  case "two" => "TWO"
}
```
示例代码中 `pf` 的类型是 `String => String` ,但是真实的函数 `pf` 只能处理 `“one”` 和 `“two”` 两种输入，除此之外的任何字符串都不能处理，函数 `pf` 就是一个偏函数，偏函数只能处理与 `case` 匹配的输入参数。如果偏函数被调用，而函数的输入与所有的语句都不匹配，系统会抛出 `MatchError` 运行时错误。可以使用 `isDefineAt` 方法测试输入参数是否与偏函数匹配。

偏函数可以链式链接， `pf1 orElse pf2 orElse pf3 ...` ， 如果 `pf1` 不匹配 ， 就会尝试 `pf2` ，以此类推， 都不匹配才会抛出 `MatchError` 。
#### 方法声明
##### 命名参数列表
```scala
case class Point(x: Double = 0.0, y: Double = 0.0) {
  def shift(deltax: Double = 0.0, deltay: Double = 0.0) =
    copy (x + deltax, y + deltay)
}
abstract class Shape() {
    def draw(offset: Point = Point(0.0, 0.0))(f: String => Unit): Unit =
    f(s"draw(offset = $offset), ${this.toString}")
}
case class Circle(center: Point, radius: Double) extends Shape
case class Rectangle(lowerLeft: Point, height: Double, width: Double) extends Shape
```
##### 多个参数列表
##### Future 简介
##### 嵌套方法与递归
#### 类型推断
#### 保留字
#### 字面量
##### 整数
##### 浮点
##### 布尔
##### 字符
##### 字符串
##### 符号
##### 函数
##### 元组
#### Option 、 Some 和 None : 避免使用 null
#### 封闭类代继承
#### 代码组织：文件和名空间
#### 导入类型及成员
##### 导入是相对的
##### 包对象
#### 抽象类型与参数化类型
