---
title: kotlin学习笔记-继承、单例、数据类
date: 2018-03-02 10:23:24
categories: kotlin
---
> 自从2017年中旬Google大会将kotlin作为Android的官方语言，至今也有将近一年的时间了，趁这段时间正好整理了下我学习kotlin的一些见解与笔记，将它以文章的形式发布出来。
<!--more-->

 今天我们所讲的是关于kotlin中的接口与抽象类、继承、接口代理、接口方法冲突、类以及成员的可见性、object(单例)、伴生对象、静态成员、方法重载、扩展成员，数据类，内部类，枚举。可能这篇的篇幅有点长，希望大家耐心看完，或者可以收藏起来，以后方便查找。
 
- [基本数据类型](https://www.jianshu.com/p/3378502de524)
- [类、成员、表达式](https://www.jianshu.com/p/4077359516dd)
- [继承、单例、数据类](https://www.jianshu.com/p/6f1d960eb683)
- [高阶函数]()
- [DSL]()
- [协程]()
- [kotlin中的反射]()
- [kotlin中的泛型]()

#### 接口与抽象类
> 接口
写法：
interface 接口名{}
> - 不能有状态
> - 必须由类对其进行实现后使用

``` kotlin
interface IView{
    fun onClick(View view)
}
```

> 抽象类
写法：
abstract class 类名(参数){}
> - 可以有状态,可以有方法实现
> - 必须由子类继承后使用

``` kotlin
abstract class BaseActivity(){
    ...
}
```

>共性
> - 比较抽象,不能直接实例化
> - 有需要子类(实现类)实现的方法
> - 父类(接口)变量可以接受子类(实现类)的实例赋值

>不同
> - 抽象类有状态,接口没有状态
> - 抽象类有方法实现,接口只能有无状态的默认实现
> - 抽象类只能单继承,接口可以有多实现

#### 继承

> 这里的被继承类与抽象类不同，抽象类是class前面被abstract修饰，这里的继承类可以不用abstract关键字，可以在class 前面加上open关键字。
> - 父类需要open才可以被继承
> - 父类方法,属性需要open才能被覆写
> - 接口,接口方法,抽象类默认为open
> - 覆写父类(接口)成员需要overrive关键字

``` kotlin
open class People{
    //属性如果要被重写需要加上open关键字
    open var name:String = "kotlin"

    //方法如果要被重写需要加上open关键字
    open fun sayName(){
        println(name)
    }
}

class Man:People(){
    //重写父类的属性需要加上override关键字
    override var name:String = "java"
    
    //重写父类的方法需要加上override关键字
    override fun sayName() {
        super.sayName()
    }
}
```

#### 接口代理

> 如果该类传入参数的接口与其实现的接口一致，我们可以不用再去实现该接口，而可以使用接口代理的方法。举个例子

``` kotlin
//我们这边定义一个接口
interface OnClickListener{
    fun onclick()
}

//这边定义一个类，参数为我们定义的接口，然后我们继承该接口
class View(var onClickListener: OnClickListener):OnClickListener{

    //实现该接口调用传入接口的onclick方法
    override fun onclick() {
        onClickListener.onclick()
    }
}

fun main(args: Array<String>) {
    //我们这边定义一个匿名内部类实现OnClickListener里面的方法，并输出一个"onclick"
    val onClickListener = object :OnClickListener{
        override fun onclick() {
            println("onclick")
        }
    }
    //我们这边创建一个view并将匿名内部类作为参数传入
    val view = View(onClickListener)
    //调用这个view实现的onclick方法，最终会输出"onclick"
    view.onclick()
}

```

通过以上代码我们会发现View这个类传入的参数这个接口与其实现的接口一致，我们还需要实现该接口的方法，然后再去调用参数这个接口的方法，是不是感觉有些麻烦呢，这里就可以用到我们的接口代理来实现。我们稍加改动下View这个类的代码。

``` kotlin
interface OnClickListener{
    fun onclick()
}

//我们在这里就可以不用实现OnClickListener这个接口的方法，直接在接口后面使用by关键字
//by后面指定的是使用哪个接口去代理
class View(var onClickListener: OnClickListener):OnClickListener by onClickListener

fun main(args: Array<String>) {
    val onClickListener = object :OnClickListener{
        override fun onclick() {
            println("onclick")
        }

    }
    val view = View(onClickListener)
    view.onclick()
}
```

#### 接口方法冲突

> - 接口方法可以有默认实现
> - 签名一致并且返回值相同的冲突
> - 子类(实现类)必须覆写冲突的方法
> - super<父类/接口名>.方法名(参数列表)

``` kotlin
interface Cat{
    fun call() = println("cat")
}

interface Dog{
    fun call() = println("dog")
}
//我们这边定义一个Zoo类实现了Cat和Dog接口，因为这两个接口的函数名，参数与返回值都相同，所以产生了接口方法冲突。
class Zoo(val name:String):Cat,Dog {
    //我们这里可以实现一个接口
    override fun call() {
        //通过传入的不同name来判断调用哪个接口的方法
        if (name == "cat") super<Cat>.call()
        else super<Dog>.call()
    }
}
```

#### 类以及成员的可见性

|kotlin|java|说明|
|:--:|:--:|:--:|
|private|private|私有(当前类中可见)|
|protected|protected|子类可见|
|-|default|包内可见|
|internal|-|模块内可见|
|public|public|公有(所有类中可见)|

``` kotlin
//kotlin中默认的修饰都是public,这里的可见修饰符的位置和java中没有太大区别
//private constructor() 私有化构造方法
class Text private constructor(){
    val name:String = "kotlin"
    //子类可见的属性
    protected val age:Int = 5
    //模块内可见的方法
    internal fun say(){
        println("$name age is $age")
    }
}
```

#### object(单例)

> - 只有一个实例的类
> - 不能自定义构造方法
> - 可以实现接口,继承父类
> - 本质上就是单例模式最基本的实现

``` kotlin
//定义一个接口
interface Object{
    fun getName()
}

//创建一个单例的People类实现Object接口的方法
object People:Object{
    override fun getName() {

    }

    fun say(){
        println("kotlin")
    }
}
//我们可以反编译下kotlin代码来看下object修饰的类是如何实现单例的
public final class People implements Object {
   //声明了一个本类类型的静态常量
   public static final People INSTANCE;

   public void getName() {
   }

   public final void say() {
      String var1 = "kotlin";
      System.out.println(var1);
   }
  
  //在类的静态代码块中new了一个本类，并赋值给INSTANCE
   static {
      People var0 = new People();
      INSTANCE = var0;
   }
}
//kotlin中调用
People.say()
//java中调用
People.INSTANCE.say();
```

#### 伴生对象与静态成员

>伴生对象
> - 每个类可以对应一个伴生对象
> - 伴生对象的成员全局独一份
> - 伴生对象的成员类似与java的静态成员

>静态成员
> - 静态成员考虑用包级函数,变量代替
> - 在java中使用静态成员与方法使用@JvmField和@JvmStatic

``` kotlin
    class People private constructor(){
        companion object {
            //@JvmField用于java调用该类的静态成员
            @JvmField
            val TAG = "kotlin"

            //@JvmStatic用于java调用该类的静态方法
            @JvmStatic
            fun say(){
                println("kotlin")
            }
        }
    }

    println(People.TAG)
    People.say()
```

#### 方法重载与默认参数

> 方法重载
> - 等同于java中的多态
> - 方法名相同,参数不同的方法
> - 返回值不能不同

``` kotlin
class People{

    fun getName(name:String):String{
        return name
    }
    
    fun getName():String{
        return "kotlin"
    }
}
```

> 默认参数
> - 为函数参数设定一个默认值
> - 可以为任意位置的参数设置默认值
> - 函数调用时产生混淆时用具名参数

比如说上面这段代码，我们定义了两个getName函数，只是参数的长度不同，那我们可以使用默认参数进行优化

``` kotlin
class People{
    //我们可以在参数里面给它一个默认值
    fun getName(name:String = "kotlin"):String{
        return name
    }
}
//调用
People().getName()
People().getName("java")
```

既然说kotlin与java代码相通，那我们在java中试试来调用这段代码

``` kotlin
public class Text {
    public static void main(String[] args) {
        //我们会发现这里的getName方法中需要传入参数
        //那我们怎么可以实现像kotlin一样不传参数也能调用呢
        new People().getName();
    }
}

//我们可以这样改动一下kotlin中的代码
class People{
    //在这个默认参数函数上面加个@JvmOverloads注解，这样我们就会发现，java那边的调用编译器就不提示错误了
    @JvmOverloads
    fun getName(name:String = "kotlin"):String{
        return name
    }
}

```

> 方法重载与默认参数
> - 二者的相关性以及@JvmOverloads
> - 避免定义关系不大的重载

#### 扩展成员

> 为现有的类添加方法,属性
写法
fun X.y():Z{...}
val X.m //这里需要注意的时扩展属性不能初始化,类似于接口属性

``` kotlin
fun String.toSay(){
    println(this)
}

val String.kotlin: String
    get() = "kotlin"

fun main(args: Array<String>) {
    "kotlin".toSay()
    "java".kotlin
}
```

关于这个是不是有点懵，其实我们可以再次反编译一下这个代码。

``` kotlin
// 这个类是我在kotlin中的类，大家不必介意
public final class ObjectKt {
   public static final void main(@NotNull String[] args) {
      Intrinsics.checkParameterIsNotNull(args, "args");
      toSay("kotlin");
      getKotlin("java");
   }
  
    //这里定义了一个静态toSay的方法，我们会发现String $receiver这个参数
    //我们大概可以猜测出来这个参数就是我们扩展的那个String类
   public static final void toSay(@NotNull String $receiver) {
      Intrinsics.checkParameterIsNotNull($receiver, "$receiver");
      System.out.println($receiver);
   }

   //val String.kotlin: String  get() = "kotlin"
   //我们发现我们定义的String.kotlin常量在这里也变成了一个静态方法。
   public static final String getKotlin(@NotNull String $receiver) {
      Intrinsics.checkParameterIsNotNull($receiver, "$receiver");
      return "kotlin";
   }
}

//其实仔细看这个类，是不是有点像我们平时在java中定义的Utils工具类呢。
//所以说想要在java中调用的话，其实和我们平时调用工具类一样
ObjectKt.toSay("kotlin");
ObjectKt.getKotlin("java");
```

#### 数据类

> 关键字data
> - 默认的实现copy,toString等方法
> - componentN方法
> - allOpen和noArg插件(因为java中的Bean默认会有无参构造方法,而kotlin使用data来修饰的class类并没有无参构造方法,并且该类名是final修饰的,为了解决这些问题,kotlin出了两个插件来解决此问题,allOpen是为了去除final修饰,noArg是为了添加无参构造方法,但我们这里需要注意的是,这两个插件的生成是在编译后有效,所以我们在编码期间,并不能调用该类的无参构造方法,如果要调用,只能通过反射去调用,反射的话后面会讲解)

``` kotlin
data class People(var name:String,var age: Int)

fun main(args: Array<String>) {
    var people = People("kotlin",15)
    println(people)
    //这里是不是感觉很奇怪呢，为啥我们还可以这样定义val成员
    //因为kotlin编译器在这里给我们自动生成了相对应的component方法
    val (name,age) = people
    println("people name is $name,age is $age")
}

//我们可以反编译来看一下
public final class People {
   @NotNull
   private String name;
   private int age;

   @NotNull
   public final String getName() {
      return this.name;
   }

   public final void setName(@NotNull String var1) {
      Intrinsics.checkParameterIsNotNull(var1, "<set-?>");
      this.name = var1;
   }

   public final int getAge() {
      return this.age;
   }

   public final void setAge(int var1) {
      this.age = var1;
   }

   public People(@NotNull String name, int age) {
      Intrinsics.checkParameterIsNotNull(name, "name");
      super();
      this.name = name;
      this.age = age;
   }

   //这里就是kotlin给我们生成的component1方法，返回值为name
   public final String component1() {
      return this.name;
   }

   //这里就是kotlin给我们生成的component2方法，返回值为age
   public final int component2() {
      return this.age;
   }

   @NotNull
   public final People copy(@NotNull String name, int age) {
      Intrinsics.checkParameterIsNotNull(name, "name");
      return new People(name, age);
   }

   //这里kotlin给我们自动生成了copy()方法
   public static People copy$default(People var0, String var1, int var2, int var3, Object var4) {
      if ((var3 & 1) != 0) {
         var1 = var0.name;
      }

      if ((var3 & 2) != 0) {
         var2 = var0.age;
      }

      return var0.copy(var1, var2);
   }
    
   //这里kotlin给我们自动生成了toString()方法
   public String toString() {
      return "People(name=" + this.name + ", age=" + this.age + ")";
   }
  
   //这里kotlin给我们自动生成了hashCode()方法
   public int hashCode() {
      return (this.name != null ? this.name.hashCode() : 0) * 31 + this.age;
   }
    
   //这里kotlin给我们自动生成了equals()方法
   public boolean equals(Object var1) {
      if (this != var1) {
         if (var1 instanceof People) {
            People var2 = (People)var1;
            if (Intrinsics.areEqual(this.name, var2.name) && this.age == var2.age) {
               return true;
            }
         }

         return false;
      } else {
         return true;
      }
   }
}
```

从上面的反编译代码中我们可以看出来，使用data来修饰的数据类，kotlin会默认的帮我们生成copy，toString，hashCode，equals等方法，还帮我们生成了两个component的方法，这里需要注意的是，它的顺序就是我们定义这个数据类构造参数里面的参数列表顺序，所以我们可以在上面写出val (name,age) = people这样的代码，而且你会发现这个类是被final修饰的，并且没有无参构造方法，那我们是不是就没有办法去修改这个类和使用无参方法初始化这个类呢，答案当然是有的，我们需要使用到allOpen和noArg插件，这两个插件会给我们去除final关键字和添加一个默认的无参构造方法。

``` kotlin
//新建一个注解类，类名随意
annotation class AllOpen

//我们在build.gradle里面进行配置
buildscript {
    ...
    dependencies {
        ...
        classpath "org.jetbrains.kotlin:kotlin-noarg:$kotlin_version"
        classpath "org.jetbrains.kotlin:kotlin-allopen:$kotlin_version"
    }
}
...
apply plugin: 'kotlin-noarg'
apply plugin: 'kotlin-allopen'

//这里进行noArg的配置
noArg{
    //配置路径就是咱们先开始定义的那个注解类
    annotation("animation.AllOpen")
}
//这里进行allOpen的配置
allOpen{
    //配置路径就是咱们先开始定义的那个注解类
    annotation("animation.AllOpen")
}

//我们进行build一下
//完事后在我们的data类上加入@AllOpen注解
@AllOpen
data class People(var name:String,var age: Int)

//完事再反编译查看一下
//我们这里会发现这个class已经没有了final关键字
public class People {
   ...这里怕大家看到这么多代码头疼我先删了好吧，这里不是重点
    //这里也多出来一个无参构造方法
   public People() {
   }
}
```

#### 内部类
> - 定义在类内部的类
> - 与类成员有相似的访问控制
> - 默认是静态的内部类,非静态内部类用inner关键字来修饰. (java中默认的是非静态内部类)

``` kotlin
    class People{
        val age:Int = 0

        inner class Name{
            val age:Int = 10

            fun sayAge():Int{
                //如果此时内部类的属性名与外部类的属性名冲突
                //可以使用@外部类名来调用外部类的属性,前提是内部类是非静态内部类
                return this@People.age
            }
        }
    }
```

#### 匿名内部类

> - 没有定义名字的内部类
> - 类名编译时生成,类似于People$1.class
> - 可继承父类,实现多个接口,与java注意区别

``` kotlin
    interface OnClickListener{
        fun onClick()
        }
    }

    class View{
        var onClickListener:OnClickListener? = null
    }

    //这里的匿名内部类不仅仅可以实现接口,同样可以继承父类,记住继承的类要是抽象类或者open修饰的类
    view.onClickListener = object : People(),OnClickListener{
        override fun onClick() {
    }
```

#### 枚举

> - 实例可数的类,注意枚举也是类
> - 可以修改构造方法,添加成员
> - 可以提升代码的可读性,但是也有一定的开销.

``` kotlin
//这里的枚举定义其实查看字节码每个枚举都是定义了一个Log类
    enum class Log(val id:Int){
        DEBUG(0),INFO(1),WARN(2),ERROR(3);

        override fun toString(): String {
            return "$name,$id"
        }
    }

    println(Log.DEBUG)
//我们查看一下字节码好吧，这里我省去了很多无用的代码
public final enum Log extends java/lang/Enum  {
  public final static enum LLog; DEBUG
  public final static enum LLog; INFO
  public final static enum LLog; WARN
  public final static enum LLog; ERROR

  static <clinit>()V
    //DEBUG
    NEW Log //在这里new了一个Log类
    LDC "DEBUG" //指向了DEBUG
    INVOKESPECIAL Log.<init> (Ljava/lang/String;II)V //调用了Log.invoke方法
    PUTSTATIC Log.DEBUG : LLog; //指向了LLog类
    //INFO
    NEW Log
    LDC "INFO"
    INVOKESPECIAL Log.<init> (Ljava/lang/String;II)V
    PUTSTATIC Log.INFO : LLog;
    //WARN
    NEW Log
    LDC "WARN"
    INVOKESPECIAL Log.<init> (Ljava/lang/String;II)V
    PUTSTATIC Log.WARN : LLog;
    //ERROR
    NEW Log
    LDC "ERROR"
    INVOKESPECIAL Log.<init> (Ljava/lang/String;II)V
    PUTSTATIC Log.ERROR : LLog;
}
```
---

到这里这篇文章的内容也就讲完了，下一篇我们继续讲解kotlin中的高阶函数。