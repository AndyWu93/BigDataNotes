

# 1 [Scala介绍](https://docs.scala-lang.org/)

Scala安装：2.12.10

REPL：Read-Eval-Print-Loop  交互式命令行

```shell
scala -version
scala  # 启动终端
```

## 编译

- 1 HelloWorld.scala 文件生成

- 2 用scalac 编译 生成2个class文件
- 3 scala 运行 HelloWorld.class

## val vs var

* val：值    
* var：变量

```scala
/**
 * String name = "若泽"
 * Int/int age = 30
 *
 * val 修饰的是"不可变"   final   val 名字[:类型] = xxx
 * var 修饰的是可变               var 名字[:类型] = xxx
 **/
object ValVarApp {
  def main(args: Array[String]): Unit = {
    var user = new User()  // 根据类创建一个对象/实例
    user.name = "若泽"
    println(user.name)   // ??  若泽
    user = null
    println(user.name)   // ??  NPE   null.name
  }

  class User {
    var name:String = "PK"
  }
}
```



# 2 基本数据类型

<img src="pictures/Scala/数据类型.png" style="zoom:75%;" />

* Byte/Char
* Short/Int/Long/Float/Double
* Boolean

```scala
/**
 * 浮点类型
 * 精度问题
 * Float使用不当可能就会有精度丢失问题   单精度
 * Double                           双精度
 */
val a:Float = 1.11f
val b:Double = 1.43

/**
 * 数据类型的转换问题
 *   数据类型自动提升
 *   数据类型从大到小时，可以使用toXXX
 */
val g = 10.asInstanceOf[Double]

// 类型判断
val h = 10.isInstanceOf[Int]
```
| Data Type | Possible Values                                              |
| --------- | ------------------------------------------------------------ |
| Boolean   | `true` or `false`                                            |
| Byte      | 8-bit signed two’s complement integer (-2^7 to 2^7-1, inclusive) -128 to 127 |
| Short     | 16-bit signed two’s complement integer (-2^15 to 2^15-1, inclusive) -32,768 to 32,767 |
| Int       | 32-bit two’s complement integer (-2^31 to 2^31-1, inclusive) -2,147,483,648 to 2,147,483,647 |
| Long      | 64-bit two’s complement integer (-2^63 to 2^63-1, inclusive) (-2^63 to 2^63-1, inclusive) |
| Float     | 32-bit IEEE 754 single-precision float 1.40129846432481707e-45 to 3.40282346638528860e+38 |
| Double    | 64-bit IEEE 754 double-precision float 4.94065645841246544e-324d to 1.79769313486231570e+308d |
| Char      | 16-bit unsigned Unicode character (0 to 2^16-1, inclusive) 0 to 65,535 |
| String    | a sequence of `Char`                                         |

## print

```scala
money = 25000
println("%d", money)
println("%f", math.Pi)
```

## 字符串插值

```scala
// 字符串插值  一定要掌握
val info2 = s"name=$name , age=$age, money=$money"
println(info2)

val info3 = s"name=$name , age=${age}, money=${money+10000}"
println(info3)

val welcome = "欢迎来到若泽数据," +
  "这里是大数据实战班"
println(welcome)

val welcome2 =
  """
    |欢迎来到若泽数据,
    |这里是大数据实战班,
    |我是若泽...
    |""".stripMargin
println(welcome2)

val sql =
  """
    |
    |select
    |e.empno, e.ename, e.deptno, d.dname
    |from
    |emp e join dept d
    |on e.deptno = d.deptno
    |
    |""".stripMargin
println(s"即将要执行的SQL语句是: $sql")
```

## lazy

```scala
# 在第一次使用时 才赋值加载（延迟加载）
lazy val a = 1
```

## 占位符

```scala
// _是一个占位符，在使用时要注意：var修饰
var d:Double= _  // d: Double = 0.0
var d:Int= _     // d: Int = 0
var d:Boolean= _ // d: Boolean = false
var d:String= _  // d: String = null
```



# 3 循环

## 3.1 while / do while

- 没有break、 continue

```scala
import util.control.Breaks._ 
// 改变作用域控制
// break
var x  = 1
breakable{
  while (x < 20) {
    x += 1
    if (x == 5) {
      println(5 + "..........")
      break
    }
  }
}

// continue
while (x < 20) {
  breakable{
    x += 1
    if (x == 5) {
      println(5 + "..........")
      break
    }
  }
}
```



## 3.2 for

### to

- 左闭右闭

```scala
// 使用频率高
// 1 to 10  ->  1.to(10)  语法糖
for (i <- 1 to 10) {  // to [ , ]
  println(i)
}
```

### until

- 左闭右开

```scala
for (i <- 1 until 10) {  // to [ , )
  println(i)
}
```

### by

```scala
// 取奇数
for (i <- 1.to(10, 2)) {
  println(i)
}

for (i <- 1 to 10 by 2) {

}
```

### reverse

```scala
for (i <- 1 to 10 reverse) {
  // 反转
  println(i)
}
```

### Range

```scala
// Range
println(Range(1, 10, 2))
```

### 加if

```scala
for (i <- 1 to 10 if i != 8) {
  println(i)
}
```

### 2层循环

```scala
//for (x <- 1 to 9) {
//    for (y <- 1 to x) {
//        print(s"$y * $x = ${x*y} \t")
//    }
//    println()
//}

for (x <- 1 to 9; y <- 1 to x) {
  print(s"$y * $x = ${x*y} \t")

  if (x == y ) println()
}
```

### 循环输出

- for循环中的 yield 会把当前的元素记下来，保存在集合中，循环结束后将返回该集合

```scala
val res: Seq[Int] = for (x <- 1 to 10) yield x * x
println(res)

val res1: String = for (x <- "Scala") yield x.toUpper
println(res1)

println("Scala".map(x => x.toUpper).mkString("")) // *****
```



# 4 函数/方法

## 4.1 基本定义

### 4.1.1 方法

```scala
/**
 *   Scala中方法或者函数 *****
 *   先把方法 等同于 函数     区别是什么 面试常考点
 *
 *  完整的定义： def 方法名(参数名:参数类型, .....):返回值类型 = { 方法体 }
 *  方法可以有返回值，也可以没有返回值
 *  方法体中不需要显示的去调用return进行返回，默认方法体内最后一行就是作为结果的返回值
 */
def 方法名(参数名:参数类型): 返回值类型 = {
	// 括号内的叫做方法体

	// 方法体内的最后一行为返回值，不需要使用return
}

// 案例
def add(x:Int, y:Int):Int = {
  x + y   // 最后一行，默认做为返回值
}
```

### 4.1.2 函数

```scala
/**
 * 函数
 * val/var 函数名称 = (入参列表) => {函数体/方法体}
 * val/var 函数名称:(入参类型) => 返回值类型 =（入参） => {函数体/方法体}
 **/

// 最最最最基础的定义
val f2:(Int,Int)  => Int = (a:Int, b:Int) => a + b
// 简写1
val f3:(Int,Int)  => Int = (a, b) => a + b
// 简写2
val f1 = (a:Int, b:Int) => a + b

println(f2(3, 19))
```



## 4.2 参数类型

### 无参数

```scala
def add2() = 1 + 2
println(add2())
println(add2)  // 方法没有入参，括号都可以省去

def sayHello(): Unit = {
    println("欢迎来到若泽数据...")
}
```

### 默认参数

Spark加载配置常用

```scala
def sayWorld(name:String = "PK哥"): Unit = {
  println(s"World: $name")
}

sayWorld("若泽")  // World: 若泽
sayWorld()       // World: PK哥
val x = sayWorld _  // 方法转函数
```

### 命名参数 

了解即可

```scala
def speed(distance:Float, time:Float) = distance / time

println(speed(100, 10))
println(speed(time = 10, distance = 100))
```

### 变长参数

- 变长参数只能放在最后那个入参中

```scala
 // * 表示可以传进来N个Int类型的参数
def sum(nums: Int*) = {  // 返回值类型不是我写的，是IDEA推导出来的
  var res = 0
  for(ele <- nums) {
    res += ele
  }
  res
}

println(sum(1, 2))
println(sum(1, 2, 3, 4, 5))
println(sum(1 to 100 :_*))  // 1 to 100 不能直接这么写 要加语法糖
```

### 函数参数

- 有参

```scala
def add(a:Int, b:Int) = a + b
// 函数可以是加法、减法......
def method(op:(Int, Int) => Int) = {
  op(10, 20)
}

method(add) // 参数是函数的函数：高阶函数

// 案例2
def foo2(add:(Int, Int) => Int): Unit = {
  println(add(10,20))
}
foo2(_+_)
foo2((a,b) => a+b)
```

- 无参

```scala
def method2(op:() =>Unit) = op()
def Print = println("xxxxx")

method2(Print _)
```



## 4.3 递归

```scala
/**
* 对于递归来说，一定要显示的写上返回值类型
*/
def test(x:Int): Long = {
  if(x == 1) 1
  else x * test(x-1)
}
```

### 尾递归

- 函数在函数体最后调用它本身，就被称为尾递归
- 尾递归的核心思想是通过参数来传递每一次的调用结果，达到不压栈。它维护着一个迭代器和一个累加器

```scala
import scala.annotation.tailrec
// 阶乘
@tailrec
def test2(x:Int, acc:BigInt): BigInt = {
  if(x == 1) acc
  else test2(x - 1, x * acc)
}
```



## 4.4 访问权限修饰符

|                        | 本类       | 本包                     |           子类           |          其他包          |
| :--------------------- | ---------- | ------------------------ | :----------------------: | :----------------------: |
| Java   public          | Y          | Y                        |            Y             |            Y             |
| Java   protected       | Y          | Y                        |            Y             | <font color=red>N</font> |
| Java   private         | Y          | <font color=red>N</font> |            Y             | <font color=red>N</font> |
| Java   default         | Y          | Y                        | <font color=red>N</font> | <font color=red>N</font> |
|                        |            |                          |                          |                          |
| Scala  public          | Y          | Y                        |            Y             |            Y             |
| Scala  private（默认） | Y+伴生对象 | <font color=red>N</font> | <font color=red>N</font> | <font color=red>N</font> |
| Scala  protected       | Y          | <font color=red>N</font> |            Y             | <font color=red>N</font> |

## 4.5 函数高级操作

### 字符串

* 多行
* 插值

```scala
val s = "PK"
val name = “Andy”

println(s”$s:$name”)

val b =
  “””Hi
    |this is
    |hello
    |world
    |”””.stripMargin

println(b)
```

### 匿名函数

```scala
// 匿名函数
// （参数名： 参数类型…）=> 函数体
val m = (x:Int) => x+1
m(1)

def add = (x:Int, y:Int) => {x + y}
add(1,3)
```

### curry函数 柯里化

curry化最大的意义在于**把多个参数的函数等价转化成多个单参数函数的级联**，这样所有的函数就都统一了，方便做lambda演算。 在scala里，curry化对类型推演也有帮助，scala的类型推演是局部的，在同一个参数列表中后面的参数不能借助前面的参数类型进行推演，curry化以后，放在两个参数列表里，后面一个参数列表里的参数可以借助前面一个参数列表里的参数类型进行推演。

上面的说法比较书面化，用更加口语化的一点来描述：
1.把多个参数转化为单参数函数的级联，达到了动态确定参数的目的。
2.当某些参数不确定时，可以先保留一个存根。剩余的参数确定以后，就可以通过存根调用剩下的参数。
3.通过类似于**建造者模式**(building)，把一个大的东西的构造过程，切成一个个的小模块来逐步构造。举个最简单的例子，Person.name("xxx").age(num).salary(count).phone(xxxx)。

```scala
// 将原来接受2个参数的一个函数，转换成2个
def sum(x:Int, y:Int) = x+y
println(sum(1,2))

// curry 函数
def sum2(x:Int)(y:Int) = x+y
println(sum2(1)(4))
```

### 链式编程

* map/flatMap

```scala
val l = List(1,2,3,4,5,6)

for(ele <- l) yield ele * 2
l.map(x=>x*2)
l.map(_ * 2)  // 每个占位符代表每个元素
l.take(3)  // 取前3个
```

- flatMap

```scala
val f = List(List(1,2),List(3,4))
f.flatten
f.map(_.map(_*2))
f.flatMap(_.map(_*2))
```

- filter

```scala
l.map(_ * 2).filter(_ > 8).foreach(println)
// l.filter(_ > 5 && _ % 2 == 1)  // 只能使用一次的可以用_
l.filter(x => x > 5 && x % 2 == 1)
```

- reduce

```scala
l.reduce(_+_)  // 1+2 =>3 + 3 =>6 + 4 .....

l.reduceLeft(_-_)
l.reduceRight((x,y) => {
  println(s"$x, $y")
  x - y
})

l.fold(0)(_-_)
```

- zip

```scala
val a = List(1,2,3,4)
val b = List("A","B","C","D")
val c = List("A","B","C","D","E")
a.zip(b)
a.zipAll(c, "-", "~")

val columns = List("empno", "ename", "job")
columns.zipWithIndex.foreach(println)
```

- groupBy

```scala
val l = List(1,2,3,4,5,6)
l.groupBy(x => if(x % 2 ==0) "偶数" else "奇数")

val array = Array(("pk",100),("pk",10),("ruoze",3),("xingxing",10))
array.groupBy(x => x._1)  // 相同的key放在一起
```

- sortBy/sortWith

- 综合案例

```scala
val lll = List(1,2,3,4,5,"PK",true)
// Int  * 10
lll.filter(_.isInstanceOf[Int]).map(_.asInstanceOf[Int]).map(_ * 10)

// wc
val lines: List[String] = Source.fromFile("data/rzdata.txt", "utf-8").getLines().toList
lines.flatMap(_.split(" ")).groupBy(x => x).mapValues(_.size).toList.sortBy(-_._2).take(2).foreach(println)
```

- 自定义map/filter

```scala
def dataFilter(list: List[Int], op:Int => Boolean) = {
  for(ele <- list if op(ele)) yield ele
}

def dataMap(list:List[Int], op:Int => Int) = {
  for(ele <- list) yield op(ele)
}

def dataForeach(list:List[Int], op:Int => Unit): Unit = {
  for(ele <- list) {
    op(ele)
  }
}


dataForeach(l, x => println(x))  // println
dataMap(l, x => x * 2).foreach(println)
dataFilter(l, _ >= 4).foreach(println)
```



### 偏函数

* 偏函数：被包在花括号内没有match的一组case语句

```scala
val names = Array("Ak", "Yu", "Bt")
val name = names(Random.nextInt(names.length))
name match {
  case "Ak" => println("Ak Dr")
  case "Yu" => println("Yu Dr")
  case _ => println("Do not kown")
}


//  偏函数改造
// A 输入参数类型  B 是输出参数类型
def sayChinese:PartialFunction[String, String] = {
  case "Ak" => "Ak Dr"
  case "Yu" => "Yu Dr"
  case _ => "Do not kown"
}

sayChinese("Ak")
```

```scala
// 找出Int类型对应的值, 乘以5
val array: Array[Any] = Array(1, 2, true, "PK", 10.0)

array.filter(_.isInstanceOf[Int]).map(_.asInstanceOf[Int]).map(_*5)

// collect 传偏函数
array.collect {
  case i:Int => i * 5
} .foreach(println)
```



# 5 面向对象

## 基本概念

[Java多态](https://blog.csdn.net/qq_41679818/article/details/90523285)

- 三大属性
  - 封装：属性、方法封装到类中
  - 继承：父类和子类之间的关系（属性、方法可以重写）
    User extends Person
  - 多态：***在使用多态后的父类引用变量调用方法时，会调用子类重写后的方法***
    - 多态成员变量：编译运行看左边
    - 多态成员方法：编译看左边，运行看右边
    - 定义格式：父类类型 变量名=new 子类类型();   Person person = new User();



## 构造器

- 主构造器
- 附属构造器

```scala
object ConstructorApp {
  def main(args: Array[String]): Unit = {
    val student = new Student(“Pk”, 33, “math”)
    println(student.name + “:” + student.major)
    println(student)
  }
}

// 主构造器（跟在类名后面即）==> Person(val name:String, val age:Int)
class Person(val name:String, val age:Int) {
  println("Person in ....")

  val school = "ustc"
  var gender:String = _
  
   /**
   * 附属构造器
   * 1) def this定义的叫做附属构造器
   * 2) 一个附属构造器中第一行必须要调用主构造器或者其他附属构造器
   */
  // 附属构造器  ==> this(name:String, age:Int, gender:String)
  def this(name:String, age:Int, gender:String) {
    this(name, age)  // 附属构造器的第一行代码一定要调用主构造器或者其他附属构造器
    this.gender = gender
  }

  println("Person out .....")
}
```

- 继承

```scala
class Student(name:String, age:Int, var major:String) extends Person(name, age) {
  println("Student in ….")

  /**
   * 重写来说，需要一个关键字override来修饰
   */
  override val school: String = “Peking”
  //  override def toString: String = super.toString
  override def toString: String = "to Student " + school
  
  println("Student out .....")
}
```

## 类

new 子类时，先调用老子，再调用自己

### 抽象类

```scala
object AbstractApp {
  def main(args: Array[String]): Unit = {
    val student = new Student2
    println(student.name)
    student.speak
  }
}

/**
 * Scala中的抽象类需要使用abstract关键字来修饰
 * 类中的一个或者多个方法/属性只有定义，没有实现
 * 抽象类是不能够直接使用的
 * 一般情况下抽象类是需要具体的子类来实现，然后使用
 *
 * 子类重写父类的抽象方法/属性时 不强制加override
 *
 **/

abstract class Person2 {
  def speak

  val name:String
  val age:Int
}

class Student2 extends Person2 {
  override def speak: Unit = {
    println("speak")
  }

  override val name: String = "PK"
  override val age: Int = 17
}
```

### 伴生类、伴生对象(apply方法)

* 类和它的伴生对象可以互相访问其私有成员
* 单例对象不能new，所以也没有构造参数
* 可以把单例对象当做java中可能会用到的静态方法工具类。
* 作为程序入口的方法必须是静态的，所以main方法必须处在一个单例对象中，而不能写在一个类中



- 伴生对象 目的

  - **弥补类中不能定义 static 属性的缺陷**：Scala 中没有 static 关键字，而 Scala 又运行与 JVM 之上，与 Java 类库完全兼容的编程语言，同时类中拥有静态属性又是如此的有必要，因此推出这个伴生对象机制就显得很有必要了。

  - **节省内存资源**：在 Java 中静态属性是属于类的，在整个 JVM 中只开辟一块内存空间，这种设定使得静态属性可以很节省内存资源，不用担心像实例属性那样有内存溢出的风险。在 Scala 中伴生对象本质上也是生成静态属性。
  - **资源共享**：既然静态属性在整个 JVM 中仅开辟一块内存空间，那就说明我们可以在所有实例当中共享这块内存里面的信息。

```scala
object ApplyApp {
  def main(args: Array[String]): Unit = {
    for (i <- 1 to 10) {
      println(ApplyTest.incr)
    }

    println(ApplyTest.count)  // 说明object本身就是一个单例对象
    
    val b = ApplyTest()      // 没有使用new xxx() ==> Object.apply   经常用   不用new就可以实例化  Array(1,2,3)

    val c = new ApplyTest()  // new xxx() ==> Class.apply           不经常使用
    c()  
  }
}

/*
 * 伴生类和伴生对象：
 * class是其object的伴生类
 */
class  ApplyTest {
   println(" ApplyTest class enter... ")

  def apply() = {
    println("Class ApplyTest apply ...")
    
    new ApplyTest  // 在class中的apply中new class
  }
  
  println(" ApplyTest class leave... ")
}

/**
 * object修饰的东西，调用时不需要new，直接 东西.属性/方法
 */
object ApplyTest {
  println(“Object ApplyTest in ….”)
  var count = 0
  def incr = {
    count += 1
    count
  }
  
  // 最佳实践：在Object的apply方法中去new class
  def apply() = {
    println("Object AplyTest apply ...")
    
    new ApplyTest  // 在object中的apply中new class
  }

  println("Object AplyTest out ....")
}
```

### 样例类(case class)

* 通常用在模式匹配

```scala
object CaseClassApp {
  def main(args: Array[String]): Unit = {
    println(Dog(“wangcai”).name)
  }
}

// case class不用new
case class Dog(name:String)
```

### Trait

类似接口

```scala
// 运行代码 
val memoryManager = new StaticMemoryManager
println(memoryManager.maxOnHeapStorageMemory)

// trait
trait MemoryManager {
  println("~~~~~~~MemoryManager~~~~~~")
  val name:String
  def maxOnHeapStorageMemory: Long
}

trait RuozedataException {
  println("~~~~~~~RuozedataException~~~~~~")
  def exception: Exception
}

class RuozedataLogger {
  println("~~~~~~~RuozedataLogger~~~~~~")
  def print(): Unit = {println("打印日志...")}
}

/**
 * 第一个用extends  后面的with
 * 继承时：普通类放在trait前面
 */
class StaticMemoryManager extends RuozedataLogger with MemoryManager with RuozedataException {
  println("~~~~~~~StaticMemoryManager~~~~~~")
  
  override val name: String = "静态内存管理"
  override def maxOnHeapStorageMemory: Long = {
    println(s"$name 获取内存...")
    100L
  }

  override def exception: Exception = {
    new RuntimeException(s"$name 抛出异常...")
  }
}

class UnifiedMemoryManager extends MemoryManager {
  println("~~~~~~~UnifiedMemoryManager~~~~~~")
  
  override val name: String = "统一内存管理"
  override def maxOnHeapStorageMemory: Long = {
    println(s"$name 获取内存...")
    200L
  }
}
```

- 先进入最老的老子，再从左往右运行(面试题)

```scala
// 运行代码 
val t = new AAABBBCCC
t.printInfo

// 测试1
class AAABBBCCC extends AAA with CCC with BBB {
  override def printInfo(): Unit = {
    super.printInfo() // BBB
    println("AAABBBCCC--------------printInfo")
  }
}

trait AAA {
  def printInfo(): Unit = {println("AAA")}
}

trait BBB extends AAA {
  override def printInfo(): Unit = {println("BBB")}
}

trait CCC extends AAA {
  override def printInfo(): Unit = {println("CCC")}
}

// 测试2
class AAABBBCCC extends AAA with BBB with CCC {
 override def printInfo(): Unit = {println("AAABBBCCC--------------printInfo")}
}
```

### 总结

#### case class vs class

- case class**初始化的时候可以不用new，当然你也可以加上，普通类一定需要加new**
- case class默认改写**toString、equals**、**hashCode**
- case class **默认是可以序列化的，也就是实现了Serializable**
- case class**支持模式匹配**

#### case class vs case object

- 有参用case class，无参用case object



## Package

引包方式

```scala
import com.ruozedata.scala.oo.pack.pack1._
val a2 = new AA
a2.method03()

import com.ruozedata.scala.oo.pack.pack1.{AA => RuozedataA}  // 别名
val a3 = new RuozedataA
a3.method03()
```

引用Java

```scala
import java.util.ArrayList

val list = new ArrayList[String]() // Scala中调用了Java SDK里面的API
list.add("pk")
list.add("若泽")

// 隐式转换
import scala.collection.JavaConverters._
for(ele <- list.asScala) {
  println(ele)
}
```

### Package Object

- Scala 可以在每一个包中定义一个包对象「package object」，作为在整个包中方便共享使用的容器。
  - 包对象中可以定义任何内容，而不仅仅是变量和方法。
  - 按照惯例，包对象的代码通常放在名为 package.scala 的源文件中。
  - 每个包都允许有一个包对象。在包对象中的任何定义都可以在包中的其他类里面使用。



## Type

```scala
// type 定义一个新的类型

type S = String  // 相当于给String起了一个名字
val name:S = "若泽"
println(name)

def  printInfo():S = "abc"
println(printInfo())
```

## 反射

```scala
// 用法
classOf[Pig]
pig.getClass  // Java pig.class

class Animal {
  override def toString: String = "这是一个动物..."
}

class Pig extends Animal {
  override def toString: String = "这是一只猪..."
}
```

## App

```scala
object AppApp extend App {
  // 不需要main
}
```

## 枚举

```scala
object EnumerationApp extends App  {

  println(WeekDay.isWorkingDay(WeekDay.Sun))

  def test(input : WeekDay) = {
    println(input)
  }

  test(WeekDay.Sun)
}

object WeekDay extends Enumeration {
  // 固定写法
  type WeekDay = Value
  val Mon, Tue, Wed, Thu, Fri, Sat, Sun = Value

  def isWorkingDay(d: WeekDay) = ! (d == Sat || d == Sun)
}
```



# 6 [集合(collection)](https://docs.scala-lang.org/overviews/index.html)

Scala中的集合是分成两大部分

- 不可变集合  scala.collection.immutable
- 可变集合    scala.collection.mutable



## Array

* 定长数组

```scala
val a = Array[String](5)
a(1) = “hello”

val b = Array(“hadoop”, “Spark”)

b.mkString(",")
b.toBuffer  // 转变长数组
```

* 可变数组

```scala
import scala.collection.mutable.ArrayBuffer
val d = ArrayBuffer[Int]()
d += 1
d += (2,3,4)
d ++= Array(6,7,8)
d.insert(0,0)
d :+ 7  // 加在末尾
7 +: d  // 加在头部

d.remove(1)
d.remove(0,3)
d.trimEnd(2)
d.toArray

for (ele <- d){
  println(ele)
}

for (ele <- (0 until d.length).reverse) {
  println(d(ele))
}
```

## List

- Nil

```scala
scala.collection.immutable.Nil.type = List()
```

- List

```scala
val l1 = List(1,2,3,4,5,6,7)
val l2 = List(4,5,6,7,8,9,10)
l1.union(l2)
l1.intersect(l2)
l1.diff(l2)
l1.sliding(2, 3)  // 窗口

1::2::Nil  // res9: List[Int] = List(1, 2)
```

- ListBuffer

```scala
import scala.collection.mutable.ListBuffer

val l = ListBuffer[Int]()
l += 2
l += (3,4,5)
l ++= List(6,7,8,9)
l -= 2
l -= 3
l -= (1, 4)
l --= List(5,6,7,8)
l ++= List(9,10,11,12)
l -= 9

l.toList
l.toArray
l.isEmpty
l.tail.tail
l.last
l.product
l.dropRight(1)
```

- 综合案例

```scala
def sum(nums:Int*):Int = {
  if(nums.length == 0) {
    0
  } else {
    nums.head + sum(nums.tail:_*)  // 递归   :_* ===> 固定写法，把seq转为array
  }
}

println(sum())
println(sum(1,2,3,4))
```

#### 排序

##### Ordering

- 类似Java的Comparator

##### Ordered

- 类似Java的Comparable

```scala
def main(args: Array[String]): Unit = {
  val s1 = new Student("a1", 1)
  val s2 = new Student("a2", 13)

  println(new CompareComm2(s1, s2).compare)
}


class Student(val name:String, val age:Int) extends Ordered[Student] {
    override def compare(that: Student): Int = this.age - that.age

    override def toString = s"Student($name, $age)"
}
```

- 使用隐式转换

```scala
implicit def s2OrderedS(stduent: Student)= new Ordered[Student] {
            override def compare(that: Student): Int = stduent.age - that.age
        }
```

- 柯里化与隐式转换

```scala
def main(args: Array[String]): Unit = {
  val s1 = new Student("a1", 1)
  val s2 = new Student("a2", 13)

  implicit val cmptor=new Ordering[Student]{
    override def compare(x: Student, y: Student): Int = x.age-y.age
  }
  println(new CompareComm3(s1, s2).compare)
}

// 柯里化
class CompareComm3[T : Ordering](x:T,y:T)(implicit comptor:Ordering[T]){
    def compare = if (comptor.compare(x,y)>0) x else y
}

class Student(val name:String, val age:Int) //extends Ordered[Student] {
  // override def compare(that: Student): Int = this.age - that.age
    override def toString = s"Student($name, $age)"
}
```



## Tuple

```scala
// 元组 最多Tuple22
val a = (1,2,3,4,5)
a._1

val t0 = 0 -> "0"

val t = (("ruozedata",9527), "PK", 30)
t._1
t._2
t._3

t._1._1
t._1._2

// 对偶Tuple
val b = ("ruozedata", "bigdata")
b.swap
```



## Set

```scala
// 无序 无重复
import scala.collection.mutable.Set

val s1 = Set(1,2,3,4,5,6,7)
val s2 = Set(4,5,6,7,8,9,10)

s1.union(s2)
s1.++(s2)
s1 ++ s2
s1 | s2
s1.intersect(s2)
s1 & s2
s1.diff(s2)
s1 -- s2
```

## Map

```scala
import scala.collection.mutable._

val a = Map("PK" -> 18, “Andy” -> 22)
val b = mutable.Map("PK" -> 18, "Andy" -> 22)
val c = mutable.HashMap[String, Int]()

b("PK") = 11
b.get("PK")  // key不存在，不会报错
b.getOrElse("PK", 9)
b.get("PK").getOrElse(9)

c += (“a” -> 1)

for ((key, value) <- b) {
  println(key + ";" + value)
}

for ((key,_) <- b) {
  println(key + ":" + b.getOrElse(key,0))
}

for (key <- b.keySet) {
  println(key + ":" + b.getOrElse(key, 0))
}

for (value <- b.values) {
  println(value)
}
```

## Option&Some&None

Option是一个抽象类
None 和 Some 继承Option

```scala
val m = Map(1 -> 2)
println(m(1))
println(m(2))
println(m.getOrElse(2, "None"))


case object None extends Option[Nothing] {
  def isEmpty = true
  def get = throw new NoSuchElementException("None.get")
}

final case class Some[+A](x: A) extends Option[A] {
  def isEmpty = false
  def get = x
}
```



# 7 模式匹配
- Java模式匹配（case）： 对一个值进行条件判断，返回针对不同的条件进行不同的处理

```scala
变量 match {
	case value1 => 代码1
	case value2 => 代码2
	.....
	case _ => 代码N  // 最后没有匹配上的
}
```

## 基本数据类型模式匹配
```scala
import scala.util.Random
val names = Array(“Ak”, “Yu”, “Bt”)
val name = names(Random.nextInt(names.length))

name match {
  case "Ak" => println("Ak Dr")
  case "Yu" => println("Yu Dr")
  case _ => println("Do not kown")
}


def judgeGrade(name:String, grade:String): Unit = {
  grade match {
    case "A" => println("Very good")
    case "B" => println("Good")
    case "C" => println("Just soso")
    case _ if(name == "lisi") => println("lisi .......")
    case _  => println(" .......")
  }
}

judgeGrade("lisi","B1")  // 多重过滤
judgeGrade("zs", "A")
```
## Array模式匹配
```scala
def greeting(array:Array[String]) = {
  array match {
    case Array("ls") => println("hi: ls")
    case Array(x, y) => println("hi: "+ x +" , " + y)
    case Array("ls", _*) => println("hi: ls and others …")
    case _ => println("hi  ......")
  }
}

greeting(Array("ls"))              // hi: ls
greeting(Array("A", "B"))          // hi: A , B
greeting(Array("zz", "ls", "aa"))  // hi  ......
```
```scala
Array(1,2,3,4,5,6,7,8,9) match {
 case Array(1,2, other@_*) => println(other.toSet)
 case _ => println("...")
}
// Set(5, 6, 9, 7, 3, 8, 4)
```

## List模式匹配

```scala
def greeting(list:List[String]) = {
  list match {
    case "zs"::Nil => println("hi zs")
    case x::y::Nil => println("hi "+x+" hi "+y)
    case "zs"::tail => println(s"hi zs and others …..$tail")
    case _ => println(".......")
  }
}

greeting(List("zs"))        // hi zs
greeting(List("zs","ls"))   // hi zs hi ls
```
## Tuple匹配

```scala
val tuple = ("PK", 31)   //  PK, 31
tuple match {
 case (name, age) => println(s" $name, $age")
 case _ => println("....")
}
```

## 类型匹配

```scala
def matchType(obj:Any) = {
  obj match {
    case x:Int => println("Int")
    case x:String => println("String")
    case x:Map[_,_] => x.foreach(println)
    case _ => println("other")
  }
}

matchType(1)
matchType(1f)
matchType(Map("a"->1))
```
## Scala异常处理
```scala
// IO
val file = "test.txt"

try {
  // open file
  // use file
  val I = 10/0
} catch {
  case e:ArithmeticException => println("不能为0")
  case e:Exception => println(e.getMessage)
} finally {
  // 释放资源, 一定执行：close file
}
```
## case class模式匹配
```scala
class Person
case class CTO(name:String, floor:String) extends Person
case class Employee(name:String, floor:String) extends Person
case class Other(name:String) extends Person

def caseclassMatch(person: Person) = person match {
    case CTO(name, floor) => println("CTO "+name+"floor " + floor)
    case Employee(name, floor) => println("employee "+name+"floor " + floor)
    case _ => println(".......")
}

caseclassMatch(CTO("PK", "12"))  // CTO PKfloor 12
```
## Some&None模式匹配

```scala
val grades = Map("pk"->"A", "zs"->"D")

def getGrade(name:String) = {
  val grade = grades.get(name)
  grade match {
    case Some(grade) => println(name+"  your grade   "+grade)
    case None => println("...")
  }
}

getGrade("zs")  // zs  your grade   D
getGrade("ls")  // ...
```



# 8 Scala隐式转换（课程26）

## 概述
* 需求：为一个已存在的类添加一个新的方法
	* Java：动态代理（LP）
	* Scala：隐式转换
		* 双刃剑
* 使用前提
  * 知道具体使用，不要滥用，会影响其他地方

## Example

- 例子1

```scala
class Man(val name:String) {
  def eat() = {
    println(s"man[ $name ] eat ……")
  }
}

class Superman(val name:String) {
  def fly() = {
    println(s"superman[ $name ] fly ……")
  }
}


// 定义隐式转换函数即可
implicit def man2superman(man:Man):Superman = new Superman(man.name)
val man = new Man("PK")
man.fly()
```

- 例子2

```scala
object ImplicitApp extends App {

  implicit def file2RichFile(file: File):RichFile = new RichFile(file)

  val file = new File("/Users/Andy/IdeaProjects/scala-train/pom.xml")
  val txt = file.read
  println(txt)
}

class RichFile(val file:File) {
  def read() = {
    scala.io.Source.fromFile(file.getPath).mkString
  }
}
```

## 隐式类

- 为Int类型增强方法

```scala
implicit class Calculator(x:Int) {
  def add(a:Int) = a+x
}

1.add(2)
```



## 切面封装

```scala
/**
 * 统一定义隐式转换的切面
 */
object ImplicitAspect {
    implicit def man2SuperMan(man:Man):SuperMan=new SuperMan(man.name)
}
```

```scala
// toList返回Scala的List类型 而FlinkKafkaConsumer需要的是Java的List类型
import scala.collection.JavaConversions._
        val kafkaSource = new FlinkKafkaConsumer[String]("topic".split(",").toList, new SimpleStringSchema(), properties)
```



## 隐式参数
* 指的是在函数或方法中，定义一个用implicit修饰的参数，此时，Scala会尝试找到一个指定类型，用implicit修饰的对象，即隐式值，并注入参数
* 不推荐使用
  * 多个隐式参数会造成混乱
```scala
implicit val test = "test"

def testParam(implicit name:String) = {
  println(name + "——————")
}

testParam
```

## 作用域（26）隐式转换参数寻找

### 1 当前作用域

```scala
object ArrayApp {
    def main(args: Array[String]): Unit = {

        implicit def b2a(b:B) = {
            println("当前作用域")
            new A
        }

        val a:A = new B()
    }
}

class A
class B

object A {}
object B {}
```

### 2 去相关类型的伴生对象找

- A的伴生对象

```scala
class A
class B

object A {
     implicit def b2a(b:B) = {
            println("A的伴生对象作用域")
            new A
        }
}
object B {}

object ArrayApp {
    def main(args: Array[String]): Unit = {

        val a:A = new B()
    }
}
```

- B的伴生对象

```scala
class A
class B

object A {

}
object B {
    implicit def b2a(b:B) = {
            println("B的伴生对象作用域")
            new A
        }
}

object ArrayApp {
    def main(args: Array[String]): Unit = {

        val a:A = new B()
    }
}
```



# 9 Scala操作外部数据

## 操作文件
```scala
import scala.io.Source

object FileApp {
  def main(args: Array[String]): Unit = {

    val file = Source.fromFile("/Users/Andy/IdeaProjects/scala-train/input/d.txt")(scala.io.Codec.UTF8)

    readline()
    readChar()
    readNet()
    
    def readline() = {
      for (line <- file.getLines()) {
        println(line)
      }
    }

    def readChar() = {
      for (ele <- file) {
        println(ele)
      }
    }   

    def readNet() = {
      val file = Source.fromURL("http://www.baidu.com")
      for (line <- file.getLines()) {
        println(line)
      }
    }

  }
}
```
## 操作MySQL
```scala
import java.sql.{Connection, DriverManager, PreparedStatement, ResultSet}

object JDBCApp {

    def main(args: Array[String]): Unit = {

        val driver = "com.mysql.jdbc.Driver"
        val url = "jdbc:mysql://hadoop000:3306/hadoop_hive"
        val username = "root"
        val password = "xxx@123"

        Class.forName(driver)
        val connection: Connection = DriverManager.getConnection(url, username, password)
        val statement: PreparedStatement = connection.prepareStatement("select * from TBLS")
        val rs: ResultSet = statement.executeQuery()
        while (rs.next()) {
            val id: String = rs.getString("TBL_ID")

            println(s"$id")
        }
    }
}
```
#### ScalaLikeJDBC（课程26）

- http://scalikejdbc.org/

##### xml

```xml
<dependency>
   <groupId>org.scalikejdbc</groupId>
   <artifactId>scalikejdbc_2.12</artifactId>
   <version>3.4.0</version>
</dependency>
<dependency>
  <groupId>org.scalikejdbc</groupId>
  <artifactId>scalikejdbc-config_2.12</artifactId>
  <version>3.4.0</version>
</dependency>
```

##### application.conf

```bash
db.default.driver="com.mysql.jdbc.Driver"
db.default.url="jdbc:mysql://hadoop000:3306/test?useSSL=false"
db.default.user="root"
db.default.password="xxx@123"

# Connection Pool settings
db.default.poolInitialSize=5
db.default.poolMaxSize=7
```

##### code

```scala
import scalikejdbc.config.DBs
import scalikejdbc.{DB, SQL}

object ScalikeJDBCApp {
    def main(args: Array[String]): Unit = {
        DBs.setupAll()

        insert()
        transaction() // 事务
        query()
    }

    def transaction(): Unit = {
        DB.localTx(implicit session => {
            SQL("update person set name=? where id=?").bind("xxx2",2).update().apply()
            SQL("update person set name=? where id=?").bind("xxx1",1).update().apply()
        })
    }

    def insert(): Unit = {
        DB.autoCommit{implicit session => {
            SQL("insert into person(id,name) values(?,?)").bind(2,"bob").update().apply()
        }}
    }
  
    def query(): Unit = {
        val a = DB.readOnly(implicit session => {
            // 方法1
            SQL("select * from person").map(re => re.int("id")).list().apply()
            // 方法2
            SQL("select * from person").map(rs => Person(rs.int("id"), rs.string("name"))).list().apply()
        })

        a.foreach(println)
    }
}

case class Person(id:Int, name:String)
```



## 操作XML

```scala
object XMLApp {

  def main(args: Array[String]): Unit = {

    loadXML()
  }

  def loadXML() = {
    val xml1 = XML.load(this.getClass.getClassLoader.getResource(“test.xml”))

    println(xml1)
  }
}

// 方法2
val xml = XML.load(new FileInputStream(“/Users/Andy/IdeaProjects/scala-train/resources/books.xml”))

```
* XML属性操作
```scala
// 获取全部
val xml = XML.load(this.getClass.getClassLoader.getResource(“pk.xml”))


// header/field
val headerfield = xml \ “header” \ “field”
println(headerfield)


// all field
val fields = xml \\ “field”
for (field <- fields) {
  println(field)
}


// header/field/name
 val fieldAttributes = (xml \ “header” \ “field”).map(_ \ “@name”)
    val fieldAttributes = (xml \ “header” \ “field” \\ “@name”)
    for (fieldAttribute <- fieldAttributes) {
      println(fieldAttribute)
    }


//name=“Logon" message
  val filters = (xml \\ “message”)
    .filter(_.attribute(“name”).exists(_.text.equals(“Logon”)))

  val filters = (xml \\ “message”)
    .filter(x => ((x \ “@name”).text).equals(“Logon”))
  for (filter <- filters) {
    println(filter)
  }
}


// header/field  content
    (xml \ "header" \ "field")
      .map(x => (x \ "@name", x.text, x \ "@required"))
      .foreach(println)
```

## Scanner/StdIn

- Java

```scala
import java.util.Scanner

object ScannerApp {
    def main(args: Array[String]): Unit = {

        val scanner = new Scanner(System.in)
        println("------"+scanner.nextLine())
    }
}
```

- Scala

```scala
import scala.io.StdIn

object ScannerApp {
    def main(args: Array[String]): Unit = {

        val str: String = StdIn.readLine("pls:")
        println("    "+ str)
    }
}
```



# 10 泛型(课程26)

### 基础知识

- Java泛型

```java
<T extends XXX>   // XXX的子类型  上届  upper bounds
  
<? super XXX>    ==> // XXX的父类型  ==> 定义下届  lower bounds
```

- Scala

```scala
[? <: XXX]   // 上届  upper bounds

[? >: XXX]  // 定义下届  lower bounds
```

- 例子

```scala
object BoundsApp {
    def main(args: Array[String]): Unit = {
      
        val cmp = new CompareComm[Integer](1, 4)  // Int 没有实现任何比较方法 不能用
        println(cmp.compare)
      
      // 视图 有隐式转换 没有定义类型
       val c2 = new CompareComm2(2, 11)
        println(c2.compare)
    }
}

class CompareComm[T <: Comparable[T]](x:T, y:T){
    def compare = if(x.compareTo(y)>0) x else y
}

// 视图界定
class CompareComm2[T <% Comparable[T]](x:T, y:T){
    def compare = if(x.compareTo(y)>0) x else y
}
```

- 例子0-1

```scala
class Student(val name: String, val age:Integer) extends Ordered[Student]{
    override def compare(that: Student): Int = this.age - that.age
}

class CompareComm[T <: Comparable[T]](x:T, y:T){
    def compare = if(x.compareTo(y)>0) x else y
}

val s1 = new Student("a1", 1)
val s2 = new Student("a2", 2)
println(new CompareComm(s1, s2).compare)
```

- 例子0-2 隐式转换

```scala
class Student(val name: String, val age:Integer) {
  override def toString: String = name + "\t" + age
}

class CompareComm2[T <% Comparable[T]](x:T, y:T){
  def compare = if(x.compareTo(y)>0) x else y
}
// 赋予student类 ordered
implicit def student2OrderedStudent(student: Student) = new Ordered[Student] {
  override def compare(that: Student): Int = student.age - that.age
}


val s1 = new Student("a1", 1)
val s2 = new Student("a2", 2)
println(new CompareComm2(s1, s2).compare)
```

- 例子0-3 隐式参数

```scala
class Student(val name: String, val age:Integer) {
  override def toString: String = name + "\t" + age
}

class CompareComm3[T : Ordering](x:T, y:T)(implicit cmptor: Ordering[T]) {
  def compare = if(cmptor.compare(x,y)>0) x else y
}
implicit val cmptor = new Ordering[Student] {
  override def compare(x: Student, y: Student): Int = x.age - y.age
}


val s1 = new Student("a1", 1)
val s2 = new Student("a2", 2)
println(new CompareComm3(s1, s2).compare)
```



- 案例1

```scala
abstract class Msg[T](context:T)
class WebChatMsg[String](context:String) extends Msg[String](context)
class DigitMsg(context:Int) extends Msg(context)  // 简写

new WebChatMsg("aaaa")
new DigitMsg(1)
```

- 案例2

```scala
object GenericApp {
    def main(args: Array[String]): Unit = {
        val value = new MM[Int, CupEnum, Int](1, CupEnum.A, 2)
        println(value)
    }

    object CupEnum extends Enumeration {
        type CupEnum = Value
        val A,B,C,D,E,F = Value
    }

    class MM[A,B,C](val faceValue:A, val cup:B, val height:C) {
        override def toString = s"MM($faceValue, $cup, $height)"
    }
}
```

- 例子3 上下届

```scala
class Person
class User extends Person
class Child extends User

def test[T](t:T): Unit = {
  println(t)
}

// User是上届
def test2[T <: User](t:T): Unit = {
  println(t)
}
// User是下届
def test3[T >: User](t:T): Unit = {
  println(t)
}

test2(new Person)  // 会报错 最多传到User
test3(new Child)   // 会报错 至少是User
```



#### 协变、逆变

- 协变

  - 父->子

  ```scala
  class Test[+User]
  val value: Test[User] = new Test[Child]
  ```

- 逆变

  - 子->父

  ```scala
  class Test[-User]
  val value: Test[User] = new Test[Person]
  ```

  

