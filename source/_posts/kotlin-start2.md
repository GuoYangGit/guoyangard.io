---
title: kotlin学习笔记-类、成员、表达式
date: 2018-02-28 09:23:24
categories: kotlin
---
> 自从2017年中旬Google大会将kotlin作为Android的官方语言，至今也有将近一年的时间了，趁这段时间正好整理了下我学习kotlin的一些见解与笔记，将它以文章的形式发布出来。
<!--more-->

 今天我们所讲的是关于kotlin中的常量与变量、函数、Lambda表达式、类成员、运算符、中缀表达式、分支表达式、循环语句、异常捕获、函数参数。可能这篇的篇幅有点长，希望大家耐心看完，或者可以收藏起来，以后方便查找。
 
- [基本数据类型](https://www.jianshu.com/p/3378502de524)
- [类、成员、表达式](https://www.jianshu.com/p/4077359516dd)
- [继承、单例、数据类](https://www.jianshu.com/p/6f1d960eb683)
- [高阶函数]()
- [DSL]()
- [协程]()
- [kotlin中的反射]()
- [kotlin中的泛型]()

#### 常量与变量

>变量：关键字var 
  写法:
 var 变量名= value 值类型    //(这里的类型我们一般可以不定义，原因在空上篇的只能类型转换中提及到)
  举例：
var x = "java"  //定义变量
x = "kotlin"  //再次赋值

>常量： 关键字val  不可能重复赋值
 写法:
val 变量名= value 值类型  //类似于java的final
例子:
val x = 2 //运行期常量
const val x = 2 // 编译器常量，等同于java中的final

关于上面我们所讲到val类似于java中的final，同样是不可以被更改，为啥还要类似呢，因为java中的常量属于编译期常量，而kotlin中的val修饰的成员是运行期常量。我们可以通过java的字节码和kotlin的字节码来查看下。

``` java
//我们先写一段java代码，声明一个常量，再声明一个变量指向这个常量
public class Text {
    private final String KOTLIN = "kotlin";
    private String kotlin = KOTLIN;
}
//我们这里可以查看一下这个类的字节码
public class Text {
  private final Ljava/lang/String; KOTLIN = "kotlin"
  private Ljava/lang/String; kotlin

  public void <init>() {
    ldc "kotlin"
    //这里将"kotlin"这个值赋给了Text.KOTLIN
    putfield 'Text.KOTLIN','Ljava/lang/String;'

    ldc "kotlin"
    //这里按照我们正常的逻辑应该是Text.kotlin的idc指向Text.KOTLIN
    //但是这里直接给了一个"kotlin"的值，所以我们得知java中的final是编译期常量
    putfield 'Text.kotlin','Ljava/lang/String;'
    return
  }
}
//如果不懂的话我们可以将上面的KOTLIN改成变量再查看下字节码
public class Text {
  private Ljava/lang/String; KOTLIN
  private Ljava/lang/String; kotlin

  public void <init>() {
    //同样这里将"kotlin"这个值赋给了Text.KOTLIN
    ldc "kotlin"
    putfield 'Text.KOTLIN','Ljava/lang/String;'
    //但是这里我们可以看到Text.kotlin的值是在getfield
    //而getfield所指向的就是Text.KOTLIN
    getfield 'Text.KOTLIN','Ljava/lang/String;'
    putfield 'Text.kotlin','Ljava/lang/String;'
    return
  }
}
```

由以上的代码我们推断出java中的常量是指编译期常量，接下来我们看看kotlin中val常量与java有何不同。

``` kotlin
//这里的逻辑同java一样
val KOTLIN = "kotlin"
var kotlin = KOTLIN
fun main(args: Array<String>) {

}
//我们这里将kotlin类反编译成java类来看一下
public final class StructureKt {
   private static final String KOTLIN = "kotlin";
   //这里我们可以看到这个kotlin并没有像上面我们所写的java程序一样
   //直接指向KOTLIN，而是在下面的static的静态代码块里面进行指向
   //所以此处我们得出结论kotlin中的val修饰的成员是运行期常量
   private static String kotlin; 
    
    //我们同样可以注意下面这几个get，set方法，我们在kotlin中并没有定义
    //KOTLIN，kotlin的get，set方法(final没有set方法)，但是kotlin会自动给我们生成该方法
   public static final String getKOTLIN() {
      return KOTLIN;
   }

   public static final String getKotlin() {
      return kotlin;
   }

   public static final void setKotlin(@NotNull String var0) {
      Intrinsics.checkParameterIsNotNull(var0, "<set-?>");
      kotlin = var0;
   }

   static {
      kotlin = KOTLIN;
   }
}
```

那我们这里可能就会有疑问了，kotlin中有没有编译期常量呢，答案肯定是有的，我们可以在val前面加上const关键字，同样我们反编译看一下。

``` java
public final class StructureKt {
   public static final String KOTLIN = "kotlin";
    //这里的kotlin直接赋值了"kotlin"
   private static String kotlin = "kotlin";

   public static final String getKotlin() {
      return kotlin;
   }

   public static final void setKotlin(@NotNull String var0) {
      Intrinsics.checkParameterIsNotNull(var0, "<set-?>");
      kotlin = var0;
   }

   public static final void main(@NotNull String[] args) {
      Intrinsics.checkParameterIsNotNull(args, "args");
   }
}

```

#### 函数

> 定义:
> - fun 函数名(参数列表):返回值类型{函数体}  // (kotlin中的默认返回值是Unit,就等同于java中的void)
> - fun 函数名(参数列表) = [表达式]  //(kotlin同样可以使用=符号来连接后面的函数体)
> - fun(参数列表)  //(匿名函数,但这里需要注意的一点是，必须有个变量来接受这个匿名函数)

``` kotlin
    fun getName(name:String):Unit{
        println("Hi,$name")
    }

    fun getName(name: String) = println("Hi,$name")

    val sayHi = fun (name:String) = println("Hi,$name")
```

#### Lambda表达式

> 其实就是匿名函数
写法:
{参数列表 -> 函数体,最后一行是返回值类型}
调用:
() 等价于invoke()

> lambda表达式的简化
> - 函数参数调用时最后一个参数如果是lambda可以移出去
> - 函数参数只有一个lambda,调用时小括号可以省略
> - lambda只有一个参数可默认为it
> - 入参的类型与形参的类型一致的函数可以用函数引用的方式作为实参传入

``` kotlin
    val sum = { a: Int, b: Int -> a + b }
    println(sum(1,2))
    println(sum.invoke(1,2))
    //我们这里定义一个函数，函数接受的参数是一个lambda表达式
    //lambda表达式接收String类型的参数，返回值是Unit
    fun lambda(action:(name:String) -> Unit){
        //我们在lambda函数中调用了传入的表达式，表达式中接收一个参数，我们传入"kotlin"
        action("kotlin")
    }
    lambda({ name -> println(name)})
    //函数参数只有一个lambda,调用时小括号可以省略
    lambda{ name -> println(name)}
    //lambda只有一个参数可默认为it
    lambda { println(it) }
    //入参的类型与形参的类型一致的函数可以用函数引用的方式作为实参传入
    //这里可能大家不太好理解，我们可以进行拆分一下
    //lambda接收的表达式中的参数是一个参数，而且是String类型
    //println(message: Any?)这个函数接收的是一个参数，类型是Any，
    //而Any类型是所有类的父类，就像java中的Object一样，同样println函数的返回值是Unit类型。
    //所以我们可以使用(类名::函数名)来简化，同样println是包级函数，所以前面的类名也可以省去
    lambda(::println)
    
    //我们这里再定义一个两个参数的函数，第一个参数是String类型，第二个参数是lambda表达式
    fun lambda(name:String,action:() -> Unit){
        action()
    }
    //函数参数调用时最后一个参数如果是lambda可以移出去
    lambda("kotlin"){println()}
```

#### 类成员

> 类的成员:
该类中被val/var修饰的为类的成员
类的方法:
被fun修饰的为类的方法
属性可以定义get/set方法，我们在上面反编译kotlin代码的时候就已经看到过kotlin默认帮我们实现了。
> - val是不可以定义set方法,因为val是不可变的成员
> - 延迟初始话,var用lateinit来实现,val用by lazy来实现 by关键字我们后面再讲，其实我们知道by关键字的意义后，我们同样可以自定义延迟初始化成员。

``` kotlin
  class people(){
        //类的成员
        var man = "guoyang"

        //这里的方法是默认kotlin帮我们实现的,我们可以进行重写
        set(value) {
            //我们可以在这里进行其他操作,比如输出该值
            println(value)
            //field为get/set方法中特有的属性,具体指我们定义好的man成员所指的真正的值
            field = value
        }
        //这里的方法是默认kotlin帮我们实现的,我们可以进行重写
        get() {
            //我们可以在这里进行其他操作,比如输出该值
            println(field)
            //field为get/set方法中特有的属性,具体指我们定义好的man成员所指的真正的值
            return field
        }

        val woman = "nicai"
        //val没有set方法
        get() {
            println(field)
            return field
        }

        lateinit var age:String

        val name:String by lazy {
            "guoyang"
        }

        fun sayHi(){
            println(name)
        }
    }
```

#### 运算符

关于kotlin中的运算符都是有相对应的**operator**修饰的函数，关于运算符与对应函数，这里我就不列举出来了，直接反手就是一个[链接](http://www.kotlindoc.cn/Other/Opetator-overloading.html)，大家自己去看好吧。

> 对于运算法需要有以下注意的几点
> - 任意类可以定义或者重载父类的基本运算符
> - 通过运算符对应的具名函数来定义
> - 对参数的个数做要求,对参数和返回值类型不做要求

``` kotlin
    class People(var real:Int){
        //定义运算符需要函数前面加上operator关键字
        operator fun plus(ohther:People):People{
            return People(real+ohther.real)
        }

        operator fun plus(ohther:Double):Double{
            return real+ohther
        }

        override fun toString(): String {
            return "$real"
        }
    }
    
    val p1 = People(1)
    val p2 = People(2)
    println(p1+p2)
    println(p1+3.0)
```

#### 中缀表达式

其实kotlin中的中缀表达式在我们代码中很少见到，但是在DSL语法中很常见(这个以后再说)，光这么说感觉很抽象，我们来看一段代码吧。

``` kotlin
fun main(args: Array<String>) {
    //我们这里可以看到定义了一个array的数组，而在后面赋值的时候使用了1 to "one"
    //有些人可能有点蒙了，那to是啥，这个数组的类型是啥，我们点击 to 进去看看方法
    val array = arrayOf(1 to "one", 2 to "two")
    println(array[0])
}
//我们这里发现to是一个infix修饰的扩展函数，返回值是个Pair类型 
//这个函数使用了泛型A.to()，里面的参数是个泛型B
//所以上面的  1 to "one"  -> 1.to("one")
//为什么能写成1 to "one" ，主要原因就在于前面的infix关键字
infix fun <A, B> A.to(that: B): Pair<A, B> = Pair(this, that)
//这里是Pair数据类
data class Pair<out A, out B>(val first: A,val second: B) : Serializable {
    public override fun toString(): String = "($first, $second)"
}
```
根据上面的代码我们照葫芦画瓢的写一个自己的中缀表达式

``` kotlin
//我们先定义一个人的类，构造方法中传入年龄
class People(val age:Int){
    //我们定义一个中缀表达式，参数传入人名，在函数中输出这个人名与年龄
    infix fun name(name:String){
        println("$name age is $age")
    }
}

fun main(args: Array<String>) {
    val p1 = People(1)
    p1 name "guoyang"
}
```

那我们这里可能会想，一个参数太少了吧，我们是不是可以多传几个呢。

![1.png](https://upload-images.jianshu.io/upload_images/3347923-8680513ac2fd548e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里的infix直接爆红了，说我们的参数只能有一个(single)，所以我们这里可以总结下：

>中缀表达式定义:
只有一个参数,并且用infix来修饰的函数,可以省略函数调用前面的.和后面的()

#### 分支表达式

> **if**表达式,这里需要注意
kotlin中的if,else是个表达式,而且是有返回值的,返回值就是分支表达式中的最后一行,并且if/else要同时出现。

> **when**表达式
kotlin中的when表达式相当于java中的switch,并且支持任意类型,而且是有返回值的,同时表达式也必须要完善。

``` kotlin
    val user1 = if (args[0] == "q"){
        "kotlin"
    }else{
        "java"
    }

    val user2 = when(args[0]){
        "q" -> "kotlin"
        else -> "java"
    }

    println(user1)
    println(user2)
```

#### 循环语句

> for循环:
基本写法
for(element in elements){函数体}  //这里的**in**也是一个运算符

``` kotlin
fun main(args: Array<String>) {
    val intArray = arrayOf(1,2,3)
    for (i in intArray){
        println(i)
    }
}
//我们这里可以通过in追踪源码进去这个in干了些什么
public class Array<T> {
    //我们看到这里有个operator运算符关键字，定义了一个iterator函数，返回了一个Iterator，我们再进去Iterator里面看做了写什么
    public operator fun iterator(): Iterator<T>
}

public interface Iterator<out T> {
    //这里也是个运算符函数，根据函数名和返回T类型，我们可以猜出这个是取下一个元素
    public operator fun next(): T

     //这里是判断还有没有下一个元素，返回值为Boolean
    public operator fun hasNext(): Boolean
}
```

>因为Array的源码看不到，但是到这里我们就可以大致猜到in在for循环中代表一个iterator()运算符函数，返回值为Iterator<T>，
这里应该是使用了Iterator里面的两个函数next()进行取下个函数，hasNext()判断有没有下个函数来进行遍历
注：这里的for循环中的in与if判断中的in可不是一个运算函数，大家没事可以去看看。
那到这里我们是不是也可以定义一个自己的类来实现for循环呢。

``` kotlin
fun main(args: Array<String>) {
    val list = MyList<Int>()
    list.add(1)
    list.add(2)
    list.add(3)
    for (i in list){
        println(i)
    }
}

class MyList<T>{
    private val list = ArrayList<T>()

    fun add(value: T){
        list.add(value)
    }
    //其实这里的Iterator我们也可以去自己定义，我比较懒- -！，就不写那么多了
    operator fun iterator(): Iterator<T> {
        return list.iterator()
    }
}
```

> While循环
>- 基本写法:
do ... while(...)...
while(...)...
>- 跳过和终止循环
continue 跳过当前循环
break 终止循环
多层循环嵌套的终止结合标签用法

``` kotlin
//多层循环嵌套的终止结合标签用法
Outter@for(...){
        Inner@while(i<0){
            if(...) break@Outter
        }
   }
```

#### 异常捕获

> 写法: 
try {代码执行体}catch (异常类型){代码执行体}finally {代码执行体}
> - catch分支匹配异常类型
> - 同样是个表达式,可以用来赋值
> - finally无论代码是否抛出异常都会执行，这里的finally如果不会用到可以不写

#### 参数

> 具名参数
定义：
给函数的实参附上形参,当然参数的位置不用固定。

``` kotlin
//这里我们定义一个两数相加的函数
fun sum(arg1:Int,arg2:Int) = arg1 + arg2
//这里我们可以指定具体的参数名
sum(arg1 = 2,arg2 = 3) 
//同样我们还可以变动参数的位置
sum(arg2 = 3,arg1 = 2)
```

> 变长参数
> - 某个参数可以接受多个值
> - 并且可以不是最后一个参数(因为java中没有具名参数,所以java中的变长参数只能是函数中最后一个参数)。
> - 如果传参时有歧义,需要使用具名参数来表示

``` kotlin
    fun array(vararg ints:Int,string: String){
        ints.forEach { println(it) }
        println(string)
    }
    array(1,2,3,4,5,string = "kotlin")
    //上面的函数是传入了5个Int参数，如果多了的话我们不可能一个个的传吧
    //这里我们会用到*运算符
    val ints = intArrayOf(1,2,3,4,5)
    array(*ints,string = "kotlin")
```

>关于Spread Operator kotlin中的*运算符我们需要注意以下几点
> - 只支持展开数组Array，不支持集合List
> - 不能像其他运算符号一样被重载

>默认参数
> - 为函数参数指定默认值
> - 可以为任意位置的参数指定默认值
> - 传参时,如果有歧义,需要使用具名参数

``` kotin
    fun array(double: Double = 3.0,string: String,int: Int = 1){
        println(double)
        println(string)
        println(int)
    }
    //此处传参时如果不使用具名参数，kotlin默认识别为第一个默认参数
    array(string = "kotlin")
    fun array(string: String,int: Int = 1){
        println(string)
        println(int)
    }
    array("kotlin")
```

---

到这里这篇文章的内容也就讲完了，下一篇我们继续讲解kotlin中的继承、单例、数据类。