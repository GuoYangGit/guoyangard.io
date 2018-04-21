---
title: kotlin学习笔记-基本数据类型
date: 2018-02-24 12:23:24
categories: kotlin
---
> 自从2017年中旬Google大会将kotlin作为Android的官方语言，至今也有将近一年的时间了，趁这段时间正好整理了下我学习kotlin的一些见解与笔记，将它以文章的形式发布出来。
<!--more-->

这段时间我将以八篇文章的形式将所学的kotlin分享出来，这是kotlin的第一篇。

- [基本数据类型](https://www.jianshu.com/p/3378502de524)
- [类、成员、表达式](https://www.jianshu.com/p/4077359516dd)
- [继承、单例、数据类](https://www.jianshu.com/p/6f1d960eb683)
- [高阶函数]()
- [DSL]()
- [协程]()
- [kotlin中的反射]()
- [kotlin中的泛型]()

#### 什么是kotlin

  首先我们来介绍下kotlin:
  
- 来自于著名的IDE IntelliJ IDEA(Android Studio基于此开发) 软件开发公司 JetBrains(位于东欧捷克)
- 起源来自JetBrains的圣彼得堡团队，名称取自圣彼得堡附近的一个小岛(Kotlin Island)
- 一种基于JVM的静态类型编程语言  

 基于JVM的静态类型语言，所以我们可以得出，kotlin是兼容java的，但是kotlin又比java多出来很多优势：
 
- 不写分号
- 支持方法扩展
- 支持lambda表达式
- 支持函数编程
- 对null的判断

#### kotlin的基本类型

 我们上一张表格来对比下kotlin的类型与java的类型

|kotlin|java|
|:---:|:---:|
|Double|double|
|Float|float|
|Long|long|
|Int|int|
|Short|short|

> 在kotlin中的基本数据类型的看起来就是java的装箱类型，在这里我们需要注意，kotlin中已经没有来装箱类型，区分装箱还是基本类型，kotlin已经交给了编译器去处理。

kotlin中类型声明赋值的语法：val 变量名 : 类型 = 值

``` kotlin
    val double: Double = 3.00
    val float: Float = 3.0F
    val long: Long = 3L
    val int: Int = 3
    val aBoolean: Boolean = true
```

#### 不可隐式转换

我们平时在java中可以将声明的int类型赋值给Long类型(java会自动拆箱装箱)，但是kotlin并不可以，必须进行类型转换

``` kotlin
    val anInt: Int = 5 
    val aLong: Long = anInt.toLong()
```

#### 字符串对比

|kotlin|java|
|:---:|:---:|
|==|equal|
|===|==|
在java中我们判断两个字符串是否相等，我们会用equal()的方法，但是在kotlin中我们用==来判断，如果想要判断两个字符串的地址值我们使用===来进行判断。

``` kotlin
    val string: String = "kotlin"
    val fromChars: String = String(charArrayOf('k', 'o', 't', 'l', 'i', 'n'))
    println(string == fromChars)
    println(string === fromChars)
```

#### 字符串模版

在java中我们一般用+来拼接字符串，但是如果要拼接的字符串很长的话，我们是不是要写很多+号，这样会使我们的代码可读性很差，kotlin考虑到了这一点，所以出现了字符串模版这个概念。

- 可以使用在前面加$来引用变量
- 如果要进行运算的话使用${}
- 如果要输出带有""的语句，在每个"前面加上\转译符
- 如果要输出$符号,我们可以在$符号前面在加上一个$符号

``` kotlin
    val arg1: Int = 1
    val arg2: Int = 2
    println("" + arg1 + "+" + arg2 + "=" + (arg1 + arg2))
    println("$arg1 + $arg2 = ${arg1 + arg2}")
    println("Hello \"kotlin\"")
    val money = 1000
    println("$$money")
```

#### 类

- 类的写法：class 类名 {方法体}  
- 如果有构造方法  class 类名 (构造参数){方法体}
- 每个类有一个init()函数，init{}是在类被构建时调用的方法
- 所有的类最终都继承自Any,就像java中的Object类一样

``` kotlin
  //其实在构造参数列表前面还有一个constructor关键字
  //这个关键字可以省去不写,但是如果你要私有化构造函数时，就需要用到constructor关键字
  //例如private constructor(var 性格: String, var 长相: String)
  class People (var name: String, var age: Int) {
        init {
            println("new了一个人类，他叫:$name,他的年龄是:$age")
        }
    }
```

#### 空类型

我们在上面介绍kotlin的时候有提及到，kotlin对null空类型判断这一块非常严格。任意类型都有可空和不可空两种,如果表示该变量可以为null,必须在类型后面加上?符号，例如 

1.  var/val 变量名 : 类型 = null 编译器不通过  
2.  var/val 变量名 : 类型? = null 可空类型

当一个类型为可空类型，在我们使用时必须进行非空判断

1. 使用if(变量 != null) 来判断
2. 使用变量名加上?符号 例如 变量名?.length 如果该变量为null，直接true，后面语句不执行
3. 使用变量名加上!!符号 例如 变量名!!.length !!符号是用来告诉编译器该类型数据绝对不会为空，不过切记该方法最好不要使用，可能会导致null异常。

``` kotlin
    val string:String? = null
    println(string?.length)
    if (string != null) println(string.length)

    val stringMessage:String? = "HelloKotlin"
    println(stringMessage!!.length)
```

#### 智能类型推断

- as 关键字(类型转换)
var/val 变量名 : 类型 = 声明类型 as 类型 (类似于java类型转换，失败的话会抛出类型转换异常)
- is 关键字(类型判断)
- 我们在声明成员并赋值时,该变量后面的类型可以不写，kotlin可以帮我们推断出来

``` kotlin
    open class People

    class Man : People(){
        fun getName():String{
            return "guoyang"
        }
    }

    fun main(args: Array<String>) {
        var guoyang:People = Man()
        if (guoyang is Man){
            //这里我们判断了如果是这个类型，那么在if语句中我们就不用在做类型转换
            println(guoyang.getName())
        }
        
        val string:String? = "Hello"
        if (string != null) println(string.length)
        //这里我们所申明的类型与赋值的类型一样，kotlin会自动推断，所以前面的类型申明可以省去
        //val people:People = People()
        val people = People()
        //as 用来进行类型转换，但是如果这里进行转换失败会抛出class类型转换失败异常
        //如果我们不想让抛出异常，可以在as后面加上？，这样的话如果转换失败，所声明的man就是个null
        val man:Man? = people as? Man
        println(man)
    }
```

#### 区间

在kotlin中，区间表示一个范围，是ClosedRange的子类，IntRange是最常用的。

>写法：
>- 0..100 表示 [0,100]  包括100
>- 0 until 表示 [0,100) 不包括100
>- i in 0..100 判断i是否在区间[0,100]中  in

``` kotlin
    val range:IntRange = 0..100
    val range_exculsive:IntRange = 0 until 100

    println(range.contains(100))
    println(100 in range_exculsive)

    for (i in range) print("$i,")
    for (j in range_exculsive) print("$j,")
```

#### 数组

|kotlin|java|
|:---:|:---:|
|IntArray|int[]|
|ShortArray|short[]|
|LongArray|long[]|
|FloatArray|float[]|
|DoubleArray|double[]|
|CharArray|char[]|

> 基本写法:  
> - val array:Array<类型> = arrayOf(...)
> - print array[i] 输出第i个成员
> - array[i] = "guoyang" 给第i个成员赋值
> - array.length 数组的长度
> - array.joinToString() 将数组拼接成String语句,括号里面传入的参数类型为String,代表中间以什么形式隔开，默认不传为,符号
> - array.slice() 将数组中的从某个开始位置到结束位置的数据取出来,返回类型为List<数据类型>,括号里面的接受参数为区间。

``` kotlin
    val arrayOfInt:IntArray = intArrayOf(1,2,3,4)
    val arrayString:Array<String> = arrayOf("k","e","t","l","i","n")

    println(arrayOfInt[1])
    arrayString[1] = "o"
    println(arrayString[1])

    println(arrayOfInt.joinToString(""))
    println(arrayString.slice(2..3))
```
---

 关于kotlin的基本类型我们就讲解到这里，下一篇我们讲解kotlin中的类、成员、表达式。