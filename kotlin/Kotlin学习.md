# 概述
现在Kotlin很多开源项目都已经开始用了，再不学习就看不懂开源了，开始学起

#  变量定义

可变变量定义

```kotlin
var 变量名 ： 变量类型 = 变量值

var name :String = "小明"

//相当于java

String name = "小明"

var name = "小明" //编译器自动推到类型为String


//注意这种写法，只能是局部变量
var aa : String //先定义不初始化，必须要有类型
aa = "aa"


```

不可变变量定义

```kotlin
val 变量名 ：变量类型 = 变量值
val name ：String = "小红"

相当于java
final String name = "小红"
```

# 函数定义

函数定义的关键字为`fun`,参数格式为 `值 ：类型`,

```kotlin
整体格式为

fun 方法名(参数名 ：参数类型 ...) : 返回值类型{

    方法体

}

fun sum(a: Int, b: Int): Int {
   return a+b
}

```
## 表达式作为函数体，返回类型自动推断

```kotlin
fun sum1 (a : Int, b : Int) = a+b
```
## 无返回值的函数，类似java中的void

```kotlin
fun printMsg (msg : Int):Unit {
    print(msg)
}

//无返回值可省略
fun printMsg1 (msg : Int) {
    print(msg)
}
```

## 可变长参数

```kotlin
fun aa (vararg a: Int){
    for (v in a){
        print(v)
    }
}

//调用
fun  main(){
    aa(1,2,3,4,5)
}
```

## lambda(匿名函数)

```kotlin
fun bb() {
    val sunLambda: (a: Int, b: Int) -> Int = { x, y -> x + y }
    print(sunLambda(1, 2))
}


//调用
fun main() {
    bb()
}

这个后续会详细鞋

```

## 字符串模板

这里新增一个操作符 `$` ,`$变量名` 表示取这个变量的值  `${方法}` 表示取方法的返回值,如下： 

```kotlin
var a =1

var ss = "value = $a"

var sss = "value =  ${sum(3,4)} "
```

输出

```kotlin
value = 1
value = 7
```

## NULL检查机制

kotlin 增加了空安全，再java中如果传递一个参数，我们需要判断是否为空，不然就会报`空指针异常NullPointerException`,比如：

```java
public void text(String msg){
        if (msg!=null){
            char c = msg.charAt(0);
        }
    }
```

kotlin中增加了空安全

```kotlin
//这样定义的变量默认就是非空的
var a1 : String = "123"
//给这个变量直接赋值null，会报错
var a1 : String = null
//在类型后加上？操作符，表示这个变量可为空,赋值null不会报错
var a2 :String?  = null
```

空安全如何再代码中调用

```kotlin
//默认情况下表示非空，可以任意调用变量的方法
var a1 : String = "123"
a1.length
//可为空的情况下,直接调用变量方法会报错
var a2 :String?  = "123"
a2.length//编译报错
```

可为空的变量有一下几种调用方式

* 显示的再代码中判读
  ```kotlin
    val b: String? = "Kotlin"
    if (b != null && b.length > 0) {
        print("String of length ${b.length}")
    } else {
        print("Empty string")
    }
  ```
* 使用`?.`操作符
  ```kotlin
    //如果 a2 不为空则返回a2 的长度，如果为空 则返回null，不会抛出空指针异常
    var length :Int? = a2?.length
  ```
* 使用`!!`操作符
  ```kotlin
  //如果不为空则正常返回，如果为空则抛出空指针异常
  var length1 : Int = a2!!.length
  ```
* Elvis 操作符
  ```kotlin
    //如果a2不为空则走?:左边，如果为空 则走?:右边
    var length2: Int = a2?.length ?: -1
  ```

## 类型检测及自动类型转换

我们可以用`is`操作符来判断类型 相当于java中的`instanceof`

```kotlin
 if (aa is String){
        //此时已经自动转换类型为String
        println(aa.length)
    }
//或者
 if (aa !is String){
        return
    }
    //这里会自动转换String
    println(aa.length)
```

## 区间

kotlin可以通过`kotlin.ranges`包中的rangeTo() 函数及其操作符形式的`..`创建俩个值的区间，通常rangeTo会辅助以 `in和!in`


```kotlin
   var i= 1
    if (i in 1..4){
        //相当于 1<=i<=4
        println(i)
    }

    for (j in 1..4 ){
        //输出1，2，3，4
        println(j)
    }
```
### 反向迭代数字

```kotlin
    for (k in 4 downTo 1){
        //输出4，3，2，1
        println(k)
    }
```

### 自定义迭代步长

```kotlin
//每俩次输出一次
 for (l in 1..5 step 2){
        //输出1，3，5
        println(l)
    }
```

### 迭代不包含结束区间

```kotlin
for (g in 1 until 4){
        //输出1，2，3不包含4
        println(g)
    }
```

# 基本数据类型

这里只记录和java不同的地方

## 类型转换

由于不同的表达方式，较小类型并不是较大类型的子类，所以不能隐式转换

```kotlin
var d : Byte = 1

//这种会报错
var e :Int =d

//可以通过这种来进行转换
var f :Int =d.toInt()
```
一些常用的转换方法

```kotlin
toByte(): Byte
toShort(): Short
toInt(): Int
toLong(): Long
toFloat(): Float
toDouble(): Double
toChar(): Char
```

有些情况也是可以自动转化的

```kotlin 
var l = 1L+4 //Long + Int  =》Long
```

## 位操作符

对于位运算没有使用特殊字符来表示，而只可以使用中缀方式调用具名函数，例如

```kotlin
val x = (1 shl 2) and 0x000FF000
```
这是完整的位运算列表(只适用于 Int Long)

```kotlin
shl(bits) – 有符号左移
shr(bits) – 有符号右移
ushr(bits) – 无符号右移
and(bits) – 位与
or(bits) – 位或
xor(bits) – 位异或
inv() – 位非
```

## 数组

在Kotlin中 数组用`Array`来表示,它定义了`get个set`函数(这个会转变成[]),以及`size`属性

```kotlin
    //定义数组[1,2,3]
    var array  = arrayOf(1,2.3)


    //定义数组 [0,2,4]
    var array1 = Array(3,{i->i*2})

    //创建一个容量为6的Int 空数组
    var arrayOfNulls = arrayOfNulls<Int>(6)

    array.forEach {
        it->
        println(it)
    }

    array1.forEach {
        println(it)
    }

    arrayOfNulls.forEach {
        println(it)
    }
```

kotlin中的数组是不型变的，也就是说我们不可以把`Array<String> 赋值给 Array<Any>`以防止可能的运行时失败（但是你可以使用 Array<out Any>, 参见类型投影,泛型）。

### 原生类型数组

Ktlion也有无装箱开销的专门来表示基本数据类型的数组，`ByteArray、 ShortArray、IntArra`，这些跟Array类并没有继承关系，但他们有相同的方法集，其用法和Array一样



## 字符串

kotlin支持多行字符串，`"""`表示

```kotlin
//这样输出会有空格
var text1 = """
        多行字符串
        多行字符串
        多行字符串
        """

//可以用trimMargin去除空格，|作为边界前缀
var text1 = """
       | 多行字符串
       |多行字符串
       | 多行字符串
        """.trimMargin()
```

输出

```kotlin
        多行字符串
        多行字符串
        多行字符串
        
 多行字符串
 多行字符串
 多行字符串

```

### 字符串模板

字符串字面值可以包含模板表达式，就是一些小代码，会求值并合并到字符串中，模板表达式可以用`$`开头

```kotlin
val i = 10
println("i = $i") // 输出“i = 10”
```

```kotlin
val s = "abc"
println("$s.length is ${s.length}") // 输出“abc.length is 3”
```
原始字符串和转移字符串都支持模板，如果你需要再模板字符串中使用$符号，可以用一下语法

```kotlin
val price = """
${'$'}9.99
"""
```

# 条件控制

## If表达式

```kotlin
  //普通使用
    if (a > b) max = a
    
    //带有else使用
    if (a > b) {
        max = a
    } else {
        max = b
    }
    
    //if表达式返回值返回值
    var min  = if (a>b) b else a
```
基本用法和java差不多，但是需要注意的是，kotlin的If表达式是有返回值的的，可以看第三个例子，也就是说可以替换java中的三元表达式

### 使用区间

```kotlin
if (a in 0..3){
        println(a)
    }
```
可以判断`a`变量是否再`0..3`区间内


## when

这个是正常的使用，相当于`java中的switch，else相当于defult`

```kotlin
 when (a) {
        1 -> println("输入1")
        2 -> println("输入2")
        else -> println("不是12")
    }
```

也可以多个条件放在一个分支

```kotlin
 when(a){
       1,2-> println("输入12")
       else-> println("不是12") 
    }
```

也可以判断是否在区间或者集合中

```kotlin
  var items = setOf<Int>(1, 2, 11)
    when(a){
        in 0..10 -> println("在区间0-10内")
        in items-> println("在集合内")
        else-> println("不在区间内")
    }
```

可以判断类型，有返回值,可以作为方法的返回值

```kotlin
fun text(a : Any)= when(a){
    is String -> true
    else -> false
}
```

也可以替换简单的if表达式

```java
when {
    x.isOdd() -> print("x is odd")
    y.isEven() -> print("y is even")
    else -> print("x+y is odd.")
}
```

## For循环

for循环可以对任何提供迭代器的(iterator)对象进行遍历，语法如下

```kotlin
for (item in collection) print(item)
```
for 可以循环遍历任何提供了迭代器的对象。即：

* 有一个成员函数或者扩展函数 iterator()，它的返回类型
* 有一个成员函数或者扩展函数 next()，并且
* 有一个成员函数或者扩展函数 hasNext() 返回 Boolean。

普通的遍历

```kotlin
 for (a in array){
        println(a)
    }
```

通过索引遍历

```kotlin
for (a in array.indices){
        println(array[a])
    }
```

使用`withIndex`遍历

```kotlin
for ((index,value) in array.withIndex()){
        println("the element at $index is $value")
    }
```

遍历区间

```kotlin 
  for (a in 1..4){
        println(a)
    }

    for (a in 6 downTo 0 step 2){
        println(a)
    }
```

## while 循环和java一样

# 返回和跳转

##  标签

标签的作用就是，可以指定跳转的位置，可以结合`break continue,return`来自定义跳转位置

### 定义标签

* 声明标签：loop@
* 使用标签:@loop


### 结合 continue使用

```kotlin
fun text1() {
    for (i in 1..3) {
        for (j in 1..3) {
            if (i == 2 && j == 2) {
                continue
            }
            println("$i -> $j")
        }
    }
}
```
输出

```kotlin
1 -> 1
1 -> 2
1 -> 3
2 -> 1
2 -> 3
3 -> 1
3 -> 2
3 -> 3
```
加上标签改造

```kotlin
fun text2() {
    look@ for (i in 1..3) {
        for (j in 1..3) {
            if (i == 2 && j == 2) {
                continue@look
            }
            println("$i -> $j")
        }
    }
}
```
输出

```kotlin
1 -> 1
1 -> 2
1 -> 3
2 -> 1
3 -> 1
3 -> 2
3 -> 3
```

不加标签跳出了本次内部循环，自行标签后跳出了本次的最外部循环

### 结合break使用

改造前

```kotlin
fun text3 (){
    for (i in 1..3) {
        for (j in 1..3) {
            if (i == 2 && j == 2) {
                break
            }
            println("$i -> $j")
        }
    }
}
```
输出
```kotlin
1 -> 1
1 -> 2
1 -> 3
2 -> 1
3 -> 1
3 -> 2
3 -> 3
```

改造后

```java
1 -> 1
1 -> 2
1 -> 3
2 -> 1
```

改造前跳出内部循环，改造后跳出外部煦暖

### 结合return使用


没有改造之前

```kotlin
fun text5() {
    var arr = arrayOf(1, 2, 3, 0, 4, 5, 6)

    arr.forEach {
        if (it == 0) return
        println(it)
    }
}
```
输出

```kotlin 
1
2
3
```

改造之后

```kotlin 
fun text6() {
    var arr = arrayOf(1, 2, 3, 0, 4, 5, 6)

    arr.forEach hhh@{
        if (it == 0) return@hhh
        println(it)
    }
}

//隐士式法
fun text7() {
    var arr = arrayOf(1, 2, 3, 0, 4, 5, 6)

    arr.forEach {
        if (it == 0) return@forEach
        println(it)
    }
}

// Lambda 法 -> 相当于递归
fun text8(){
    var arr = arrayOf(1, 2, 3, 0, 4, 5, 6)

    arr.forEach(fun(value: Int) {
        if (value == 0) return
        println(value)
    })
}
```
输出

```kotlin
1
2
3
4
5
6
```
没改造之前直接退出方法，改造之后，退出然后继续循环遍历



# 类和对象

这里只说和java不太一样的地方

## 构造函数

kotlin类具有俩个构造函数，一个`主构造函数`，一个`次构造函数`，

### 主构造函数

主构造函数是类头的一部分他跟在类名后,构造函数的关键字为`constructor`

```kotlin
class Person constructor(firstName: String) { /*……*/ }
```

如果主构造函数没有任何的注解，或者可见修饰符，可以省略这个`constructor`关键字

```kotlin
class Person(firstName: String) { /*……*/ }
```

主构造函数不能包含任何的代码快，初始化代码可以放到`init关键字`的代码块中进行初始化，主构造函数的参数可以再init代码块中使用，也可以应用于类的成员属性的赋值

```kotlin
//带有关键字constructor
class A constructor(a: String) {
    var a: String = "shsh"
    val b: String = "ccc"
    var c: String
    val d: String

    //初始化代码块，可以初始化主构造函数
    init {
        c = a
        d = a
        println(a)
    }

    fun text(a :String) {
        println(a)
    }
}

//不带有关键字constructor
class B(b : String){
    var lastName :String  = "hahha"
        get() {
            if (field.equals("hahha")){
                field="333"
            }
            return field
        }
        set(value) {
            if (value.equals("111")){
                    field="222"
            }else{
                field=value
            }
        }

    var hah :String  = "222"
    init {
        println(b)
    }

//注释1
     fun text(){
        println(b)//报错，注意这里访问不到主构造函数参数，报错
    }
}

```

这里重点看下注释1，方法里访问不到主构造函数的参数，如何可以访问呢？,下面这俩种都可以访问

```kotlin
class C(c: String) {
    var c1 = c;

    fun text(){
        println(c1)
    }
}

class  D(var d :String){
    fun text(){
        println(d)
    }
}
```

如果主构造函数有修饰符或者注解那么关键字不能隐藏

```kotlin
class Customer public @Inject constructor(name: String) { /*……*/ }
```
### 次构造函数

类也可以声明前缀有 `constructor`关键字的次构造函数

```kotlin 
class E {

    constructor( a1111: String, b: Int) {
            println(a1111+b)
    }
}
```
如果类有一个主构造函数，每一个次构造函数都需要委托给主构造函数，可以直接委托，后者通过其他构造函数间接委托，委托到同一个类的另一个构造函数，需要用到`this`关键字

```koltin
class F(a: String) {

    constructor(a: String, b: Int) : this(a) {

    }
    
    constructor(a:String,b: Int,c:Int):this(a,b){
        
    }
}
```

需要注意的是`init`代码块会`在次构造函数之前`执行

如果一个非抽象类没有任何的主次构造函数，会自动生成一个没有参数的主构造函数，构造函数是public，如果你不希望有一个公有的构造函数，可以主动创建一个构造函数，自己定义范围
```kotlin
class DontCreateMe private constructor () { /*……*/ }
```

### 创建类的实例

和java的区别是，没有`new`关键字
```kotlin
class F(a: String) {

    constructor(a: String, b: Int) : this(a) {

    }

    constructor(a:String,b: Int,c:Int):this(a,b){

    }
}

fun main() {

    //没有new关键字
   var a  = F("11",1)
}
```

## 属性的Getters 与 Setters

声明一个属性的完整语法是

```kotlin
var <propertyName>[: <PropertyType>] [= <property_initializer>]
    [<getter>]
    [<setter>]
```

其实就是对应于`java的get和set`方法

```java
public class EE {
    
    public String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```
这个是java中的正常写法,kotlin中会自动生成get，set

```kotlin
class FF {
    var name: String = "name"

    var age: String = "age"
        get//默认实现
        set//默认实现

}
```
当然kotlin中也可以自定义get，set方法，其中`filed`字段表示本属性

```java
class FF {
    var name: String = "name"
        get() = "不管你设置什么都返回这个"



    var age: String = "age"
        get() {
            if (field.equals("18")){
                field="18岁可真好"
            }
            return field
        }
}

fun  main(){
    var f = FF()


    println(f.age)
    f.age="18"
    println(f.age)

    println(f.name)
    f.name="xiaomign"
    println(f.name)

}
```
这个就是重写了get方法，当你使用属性值得时候就相当于调用了get方法，会返回你自定义内容

```kotlin
age
18岁可真好
不管你设置什么都返回这个
不管你设置什么都返回这个
```

```kotlin
class FF {
    var name: String = "name"
        //这个就是返回原值
        get() = field
        set(value) {
            if (value.equals("小明")){
                field="原来是小明啊"
            }
        }
}

fun  main(){
    var f = FF()

    

    println(f.name)
    f.name="小明"
    println(f.name)

}
```
这就是重写了set方法，当你给属性赋值就会自动调用set方法

输出

```kotlin
name
原来是小明啊
```

val属性相当于java中的final，不可改变，所以val属性默认没有set方法，因为他不可改变，我们也可以给get set加上访问修饰符privite,public

```kotlin
class C(c: String) {
    var c1 = c
        private set

    fun text(){
        println(c1)
    }
}
```
c1 属性的set是privite的，所以外部不能重新给c1赋值

### 编译期常量

```kotlin
//要位于顶层
const val aaa :String  = "aa"
//相当于java
public static final  String aaa = "aa"
```

### 延时初始化属性变量

普通变量都必须再构造函数中初始化，这种就很不方便，我们可以选择延时初始化，使用关键字`lateinit`

```kotlin
class C(c: String) {

    lateinit var abc: String

}
```
这样就可以延迟初始化

## 嵌套类

相当于java中的 static 内部类，他不依赖外部类的对象
```kotlin
class outer { //外部类
    var a: Int = 1

    class Nested { //嵌套类
        fun text() = 2
    }
}

fun main() {
    var a = outer.Nested().text() //调用方式

    println(a)
}
```

## 内部类

内部类用`inner`关键字来表示,内部类会带，带有外部类的引用，相当于java中的普通内部类

```kotlin
class outer { //外部类
    private var a111: Int = 1


    class Nested { //嵌套类
        fun text() = 2
    }

    inner class inner { //内部类
        fun text() = a111 //访问外部类成员变量

        fun text1() {
            var out = this@outer//拿到外部类对象
            println(out.a111)//访问外部类成员
        }
    }
}


fun main() {
    var a = outer.Nested().text() //嵌套类调用方式

    println(a)

    var b = outer().inner().text() //内部类调用方式
    var c = outer().inner().text1()
}
```

## 匿名内部类

```kotlin
interface Face {
    fun tetx() :String
}

class outer {
    fun text3 (face: Face){
        var a = face.tetx()
        println(a)
    }
}

fun main() {
    //使用
    outer().text3(object : Face{
        override fun tetx(): String {
            return "匿名内部类"
        }
    })
}
```
其实根java差不多

## 访问修饰符

### 类属性修饰符，为了标识类的本身特性

* abstract //抽象类
* final  //类不可继承
* enum   //枚举类
* open   //类可继承。(类默认是final，不可继承)
* annotation //注解类

## 访问权限修饰符，标识可以访问范围

* public //所有地方都可访问
* private //只有本类可以访问
* protected //本类和子类可以访问（这个不能修饰类，可修饰成员变量，方法等）
* internal //只有本模块可以访问






  



 


















































