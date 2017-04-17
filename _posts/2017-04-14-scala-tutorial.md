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
方法的定义可以嵌套，方法内部可以定义方法，但是内部方法只有在方法内部可见, 下面示例方法 `fact` 与 `factTail` 就是方法 `factorial` 的内部方法，在 `factorial` 外部不可见：
```scala
def factorial(i: Int): Long = {
  def fact(n: Int): Long = {
    case 1 => 1.toLong
    case n if n > 1 => n * f(n - 1)
  }
  def factTail(n: Int, acc: Long): Long = {
    n match {
      case 1 => 1.toLong
      case n if n > 1 => factTail(n - 1, n * acc )
    }
  }
  fact(i)
}
```
在示例中， `fact` 与 `factTail` 都调用了本身， 像这种方式定义的方法就是递归方法。
#### 尾递归
在上一节的示例中， `fact` 与 `factTail` 都是递归方法， 但是他们有点不同，这是一种使用递归的诡计， 让我们先从函数栈的角度来理解递归，然后再画一个简单的示意图来理解这种诡计。 要理解尾递归，先要理解函数调用栈。
> Java 栈是一块线程私有的内存空间。 java 堆和程序数据相关， java 栈就是和线程执行密切相关的，线程的执行的基本行为是函数调用，每次函数调用的数据都是通过 java 栈来传递的。 Java 栈与数据结构中的 stack 有着类似的含义，都是先进先出的数据结构，只支持出栈和入栈操作。 java 栈中保存的主要内容为栈帧。每一次函数调用都有一个对应的栈帧被压入 java 栈。 每一个函数调用结束，都会有一个栈帧被弹出 java 栈。当前正在执行的函所对应的栈帧位于当前栈的栈顶，它保存当前函数的局部变量，中间运算结果等数据。

![funcstack](/images/posts/2017_04/funcstack.png)

在示意图中，被调用函数执行结束后相应的栈帧就会被弹出，执行的结果放在调用函数栈帧的某个位置。 了解了函数栈帧后再来看递归函数，因为递归函数的参数在没有到达边界条件的时候，比如上节示例代码中 `fact` 参数大于 1 的时候， `fact` 就会一直调用自身， 相应的栈帧就会一直增长， 如果栈空间不足， 就会造成 `StackOverflowError` 错误。 这是使用递归函数需要非常小心的地方！

而尾递归是编译器对递归的一种优化措施(将递归编译成等效的循环)， 但并不是所有的递归函数都可以进行尾递归优化， 书本中这样定义尾递归： `表示调用递归函数是该函数中最后一个表达式，该表达式的返回值就是所调用的递归函数的返回值` 。 下面我们结合一些简单的示意图来理解这个定义：
![factTail](/images/posts/2017_04/factTail.png)

先对上图做一点简单的说明，灰色的块表示调用递归方法的栈帧， 蓝色的块表示递归方法的栈帧， 这个简图只是为了形象的描述递归方法，不一定完全满足真实的栈帧布局，欲了解详细情况请学习 JVM 相关知识！ 绿色的线表示栈帧销毁的方向， 栈增长的时候必要的信息向新生的栈帧中传递，一般栈增长时传递的信息表现为递归函数的参数， 而栈帧销毁的时候代表了信息的反向传递，在 `fact` 这种情况中， 前一个栈帧运行结束， 返回值会和当前栈帧其他的存储在内存中的值做运算，以运算结果作为当前栈帧的返回值， 然后再销毁当前栈帧， 图中上半部分的情况描述了这一过程， 在这种情况中， 一个栈帧的返回值依赖于上一个栈帧的返回值和当前栈帧中的一些信息。 在 `factTail` 这种情况中，前一个栈帧运行结束， 返回值不与当前栈帧中的信息发生运算， 直接将前一个栈帧的返回值作为当前栈帧的返回值， 相当于最终的结果在栈生长到最后一个栈帧的时候就已经确定了， 栈帧销毁的时候只是把这个固定的信息向栈帧销毁的方向移动， 每个栈帧之前保存的信息对于最终的信息没有任何影响！

基于上诉分析，我们发现图例中下面这种情况可以做一种优化： 既然栈帧生成的时候就已经确定了最终的信息，那么栈帧销毁就没有必要了， 我们可以省略掉这个反向的过程！ 但是函数调用必须要一个一个栈帧的销毁， 不能进行跳跃式的销毁， 所以我们有必要对这个过程做进一步的分析， 既然栈帧生成的时候就确定了最终的信息，也就是说最终的信息只与栈帧生成时传递的信息有关， 而栈帧每次传递的信息只有参数，而参数的数量是固定的， 并且生成下一个栈帧后这些参数就不会被使用了， 那么我们可以不再生成新的栈帧了， 直接把新的参数覆盖掉当前的参数， 对于 `factTail` 的例子就是在 `[4 1]` 的位置重新换上新的参数 `[3 4]` ， 而这就是循环： 一个位置控制结束，一个位置保存当前结果，每次循环更新控制状态、更新当前结果！  scala 中在方法定义前使用 `@tailrec` 注解可以提醒编译器检查是否满足尾递归并对尾递归进行优化。

### 类型推断
scala 是一个静态类型的语言， 但是 scala 的编译器又提供了一定能力的类型推断功能， 降低了代码的冗余度， 一些函数式编程语言， 如 Haskell ， 可以推断出几乎所有的类型， 因为他们可以执行全局类型推断。 但是 scala 因为支持了继承，使得全局推断困难得多。 下面这些情况需要显式的指明类型信息：
- 用 `var` 或 `val` 声明没有初始化的变量， 比如在类的抽象声明中 ```val book: String``` 。
- 所有的方法参数， 如 ```def deposit(amount: Money) = {...} ```
- 方法的返回值类型， 在以下情况中必须显示声明类型：
    - 在方法中明显使用了 `return` 。
    - 递归方法
    - 两个或多个方法重载， 其中一个方法调用了另一个重载方法， 调用者需要显示类型注解。
    - Scala 推断出的类型比你期望的更加宽泛。 

```scala
object StringUtil {
  def joiner(string: String*): String = string.mkString("-")
  def joiner(string: List[String]) = 
    joiner(string :_*)  //不能通过编译，需要显式指明方法返回类型
  
  def makeList(strings: String*) = {//期望类型推断为 List[String]
    if (strings.length == 0)
      List(0)                    //误用，类型为 List[Int]，本来该用 List.empty[String]
    else strings.toList          //类型为 List[String]
  }                              //类型将会被推断为 List[Any]
  
  def double(i: Int) { 2 * i}    //类型推断为 Unit
  def double2(i: Int) = { 2 * i} //类型推断为 Int
}
```
### 字面量
- 整数
    - 十进制： `0` 或 非 `0` 开头的数字
    - 十六进制： `0x` + `(0-9, A-F or a-f)`
    - 八进制： `0` + 数字， 从 scala 2.10 已经废弃
- 浮点
    - `Float` 小数后加 `f` 或者 `F`
    - `Double` 小数后加 `d` 或者 `D` ， 小数后不加默认 `Double`
    - 小数点后没有数字的浮点数例如 `3.` 这种写法在 `scala 2.10` 中废弃， 在 `scala 2.11` 中禁止！
- 布尔： 类型为 `Boolean` ， 只有两个值
    - `true`
    - `false`
- 字符
- 字符串
    - 双引号: 里面出现双引号需要转义字符 `\`
    - 三重引号: 可以跨行， 可以包含任意字符， 不会转义， 但是里面不能出现三个连续的引号

```scala
def hello(name: String) = s"""Welcome !
  Hello, $name !
  * (Gratuitous Star !)
  |We're glad you're here. 
  |  Have some extra whitespace.""".stripMargin() 
hello("yankun")
```
输出：
```
Welcome !
  Hello, yankun !
  * (Gratuitous Star !)
We're glad you're here. 
  Have some extra whitespace
```

- 符号: `'` + `数字` or `字母` or `下划线` ， 但是第一个字符不能为数字， 或者 `Symbol("id")` , 包含空格的符号 `Symbol("id ")`
- 函数: `val f: (Int, String) => String = (i, s) => s + i.toString()` , 对象 `f` 的类型为 `Function2[Int. String, String]` (简写为 `(Int, String) => String`)
- 元组: `TupleN` ， 例如 `val tup = ("www.yankun.tech", 100)` 类型为 `Tuple2[String, Int]` 简写为 `(String, Int)` 。 使用 `_n` 从元组中提取值， 例如 `tup._1` 。 使用箭头操作符产生二元组 `"www.yankun.tech" -> 100` 。

### 值、对象、类、类型
- 值与对象会在内存中占用一块内存， 一定意义上说， 值与对象是个等效的概念。
    - 伴生对象： 一个与类同名称的对象， 比如 `List` 对象。 这种对象里含有一个方法 `apply` ， 当任意一个对象包含这个方法的时候， 调用这个方法可以不用写成 `对象.apply(参数)` 可以简写为 `对象(参数)` 。 比如 `List` 的 `apply` 方法， 调用时直接使用 `List(参数)` ， `List` 对象 `apply` 方法类型签名为：
    - ```scala
    def apply[T](args: T*): List[T] = {...}
    ```
    - 单例对象： 用 `object` 关键字定义的一个单独的对象。
    - 函数对象： 在 scala 中， 函数与方法是不同的， 函数是一个对象， 就跟字符串对象或者其他对象是一样概念的对象， 之所以我们可以使用 `函数(参数)` 是因为函数对象里面有一个 `apply` 方法， 通过上文我们知道 `函数(参数)` 是 `函数.apply(参数)` 的简写。 而方法是对象的一个字段， 跟一般的方法概念是一样的， 但是我们可以把方法变成一个函数对象， 方法提升为函数对象的方式一般有两种：
        - 部分应用： 调用方法， 有些参数有值，有点参数没有值，没有值的使用下划线占位， 将会产生一个以下划线为参数列表的函数。
        - `eta` 变换： `方法 _`
- 类是构造对象的模板。 一个类可能有多个类型， 比如类 `Array[T]` 的实际类型有 `Array[String]` 、 `Array[Int]` 等。
    - 伴生类： 与对象同名称的类。

### 传名参数、传值参数
- 传值参数： 通常所见的参数类型， 参数是一个值， 调用方法的时候需要先计算出参数的值，再进入方法内部执行。
    - 普通值： 普通的值与对象。
    - 函数对象： 参数为一个函数对象， 比如 `def test(code : ()=>Unit): Unit`
- 传名参数： 参数为一个代码块， 调用方法时不对代码块进行求值， 方法内部每次使用该参数都对参数进行一次求值。
例如：
```scala
def test1(code: ()=>Unit){ //参数为传值参数(一个函数对象)
    println("start")
    code() //其实是调用 code 对象的 apply 方法
    println("end")
    code()
  }
  test1 {//此代码块， 先对代码块求值(代码块的值为最后一个表达式的值)， 再进入方法， 方法内部可以直接使用这个值
    println("1111")
    ()=>{println("2222")}
  }
```
执行结果为：
```
1111
start
2222
end
2222
```
```scala
def test(code : => Unit){ //传名参数
  println("start")
  code // 可以理解为直接展开代码块， 然后对代码块求值， 每次使用都是这个过程， 也可以写成 code()
  println("end")
  code
}
test{// 不对代码块求值， 作为一个整体进入方法
  println("1111")
  println("2222")
} 
```
执行结果为：
```
start
1111 
2222
end
1111
2222
```

### Option 、 Some 和 None : 避免使用 null
当一个返回值可能产生 `null` 的时候， 不直接返回这个值， 而是使用 `Option[T]` 对象对返回值进行封装， 正常值为 `Some(value of type T)` ， null 变为 `None` , `Some` 与 `None` 都是 `Option[T]` 的子类， 对 `Option[T]` 对象使用 `get` 方法获取 `Option[T]` 里的值， 如果是 `None` 将会抛出 `NoSuchElementException` 异常。 可以使用更安全的 `getOrElse` 方法为 `None` 提供一个默认值。 更多知识可以点击 [这里](http://udn.yyuap.com/doc/guides-to-scala-book/chp5-the-option-type.html) 了解。 
### 封闭类的继承
使用关键字 `sealed` 告诉编译器所有子类必须在同一个源文件中声明。 比如类型 `Option` 只有两个子类 `Some` 和 `None`， 为了防止用户私自给 `Option` 派生其他子类， `Option` 声明时使用 `sealed` 关键字进行限制：
```scala
sealed abstract class Option[+T] extends ... {...}
```

### 代码组织：文件和名空间
Scala 沿用 Java 用包涞表示命名空间这一做法， 但是更加灵活， 文件名不必与类名一致， 包结构不一定要与目录结构一致。
- Java 常规写法：
```scala
//file: src/main/com/example/mypkg.scala
package com.example.mypkg
class MyClass {...}
```
- 嵌套结构语法定义包作用域
```scala
//file: src/main/com/example/mypkg.scala
package com {
  package example{
    package pkg1{
      class C11 {def m = "m11"}
      class C12 {def m = "m12"}
    }
    package pkg2{
      class C21 {def m = "m21"; def makeC11 = {new pkg1.C11}; }
    }
    package pkg3.pkg31.pkg311 {
      class C311 {def m = "m311"}
    }
  }
}
```

- 包不能在类或对象中定义
### 导入类型及成员
```scala
import java.awt._                //导入 awt 下所有成员
import java.io.File              //导入 File
import java.util.{Map, HashMap}  //导入 Map HashMap
import java.math.BigInteger.{
  ONE => _ ,                     //隐藏这个成员
  TEN ,                          //导入 TEN
  ZERO => JAVAZERO               //导入 ZERO 并重命名为 JAVAZERO
}
```
#### 导入是相对的
```scala
import collection.immutable._    
// collection 为 scala 包的子包， scala 包默认已导入
import _root_.scala.collection.parallel._ //从根开始导入
```

#### 包对象
```scala
// src/com/example/json/package.scala //文件名必须为 package.scala
package com.example //json 包的上层包作用域
package object json { //package 关键字给包对象命名， 此处为 json
  class JSONObject {...}  //暴露给 API 使用者的包对象的成员
  def formString(string: String): JSONObject = {...}
}
```
使用时通过 `import com.example.json._` 导入包对象中的成员

### 抽象类型与参数化类型
