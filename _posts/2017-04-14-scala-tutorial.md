---
layout: post
title: scala 教程
categories: scala
description: scala 程序设计第二版读书笔记
keywords: 程序语言
---

> 此博客主要内容为 《 Scala 程序设计 (第 2 版) 》 读书笔记，转载请保留原始地址。 需要更加深入学习建议购买正版图书 [图灵社区](http://www.ituring.com.cn/book/1593)

## 快速入门
[scala 课堂](https://twitter.github.io/scala_school/zh_cn/)
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
### 偏函数
```scala
val pf: PartialFunction[String, String] = {
  case "one" => "ONE"
  case "two" => "TWO"
}
```
示例代码中 `pf` 的类型是 `String => String` ,但是真实的函数 `pf` 只能处理 `“one”` 和 `“two”` 两种输入，除此之外的任何字符串都不能处理，函数 `pf` 就是一个偏函数，偏函数只能处理与 `case` 匹配的输入参数。如果偏函数被调用，而函数的输入与所有的语句都不匹配，系统会抛出 `MatchError` 运行时错误。可以使用 `isDefineAt` 方法测试输入参数是否与偏函数匹配。

偏函数可以链式链接， `pf1 orElse pf2 orElse pf3 ...` ， 如果 `pf1` 不匹配 ， 就会尝试 `pf2` ，以此类推， 都不匹配才会抛出 `MatchError` 。
### 方法声明
#### 命名参数列表
```scala
case class Point(x: Double = 0.0, y: Double = 0.0) {
  def shift(deltax: Double = 0.0, deltay: Double = 0.0) =
    copy (x + deltax, y + deltay)
}
```
可行的构造方式有：
```scala
val p = new Point(x=3.0, y=4.0)
val p1 = Point(3.0,4.0)
val p2 = Point(3.0)                          //p2 = Point(3.0, 0.0)
val p3 = Point(y=4.0)                        //p3 = Point(0.0, 4.0)
```
#### 多个参数列表
```scala
def drawPoint(p: Point = Point(0.0, 0.0))(drawfunc: String => Unit = println) =
  drawfunc(p.toString)
```
可以任意指定参数列表的个数， 但是实际上很少有人使用两个以上的参数列表。当最后一个参数列表包含一个表示函数的参数时， 多个参数列表的形式拥有整齐的块结构语法， 还允许将参数列表两边的圆括号替换为花括号， 例如：
```scala
drawPoint(Point(3.0, 4.0)) {
  str => {
    println("drawfunc !!!")
    println(str)  //{...} 代码快的返回值为最后一个表达式的值，在这儿就是 println(str) 
  }
}
```
这种方式很像其他语言中的 `if` 和 `for` 表达式， 只不过在 scala 的这种写法中， `{...}` 块表示的是传给 `drawPoint` 方法的参数， `str => {...}`其本质是一个类型为 `String => Unit` 的匿名函数。 示例代码中第一个参数使用默认值时第一个括号不能省略。

多参数列表第二个优势是类型推断：
```scala
def m1[A](a: A, f: A => String) = f(a)
def m2[A](a: A)(f: A => String) = f(a)

m1(100, i => i.toString)                           //error: missing parameter type
m2(100)(i => i.toString)                           //可以正常运行
```
多参数列表第三个优势是最后一个参数列表使用 `implicit` 关键字声明隐式参数，当相应方法被调用时，可以显示指定这个参数，当没有指定时，编译器会在当前作用域找到一个合适的值作为参数，可以让代码更精简。下节的 `Future` 就是这样一个例子。
#### Future 简介
```scala
object Future {
  def apply[T](body: => T)(implicit execctx: ExecutionContext): Future[T]
}
```
`Future` 将一段耗时的代码 `body` 进行封装，封装后返回一个 `Future[T]` 类型的对象，这段代码会在 `ExecutionContext` 中进行执行，只有在 `ExecutionContext` 中执行结束， `Future[T]` 类型的对象才能获取到 `body` 的运行结果。 示例代码中的 `body` 为 `传名参数` ， 后文会有详细介绍。 更多关于 `Future` 的介绍可以 [点击](http://udn.yyuap.com/doc/guides-to-scala-book/chp8-welcome-to-the-future.html) 进行更多了解。
![Future](/images/posts/2017_04/future.png) 
#### 嵌套方法与递归
### 类型推断
### 保留字
### 字面量
#### 整数
#### 浮点
#### 布尔
#### 字符
#### 字符串
#### 符号
#### 函数
#### 元组
### Option 、 Some 和 None : 避免使用 null
### 封闭类代继承
### 代码组织：文件和名空间
### 导入类型及成员
#### 导入是相对的
#### 包对象
### 抽象类型与参数化类型
