# 理解lambda

## 定义一个完整的匿名函数

###  首先看下定义普通变量

```koltin
var a :String = "qq"
```
这是一个普通的变量，`变量名为a`，`类型为String` `值为"qq"`

### 然后我们看下一个匿名函数变量

```koltin
    var fun1 : (Int,Int)->String =fun(a:Int,b:Int):String {
        return "测试$a+$b"
    }
```
*  `变量名为fun1` 
*  `类型为(Int,Int)->String（参数是（int,int）返回值为String的匿名函数）`
*  `值为:匿名函数，因为他没有函数名`
```koltin
fun(a:Int,b:Int):String {
        return "测试$a+$b"
    }
```

### 最后看下调用

```koltin
fun main() {
    println(fun1(100,200))
}
```
输出
```koltin
测试100+200
```

## 然后我们对上面的fun1匿名函数变量进行优化删减

**这是一个完整的匿名函数**

```kotlin
   var fun1 : (Int,Int)->String =fun(a:Int,b:Int):String {
        return "测试$a+$b"
    }
```
**第一次优化**

把`fun(a:Int,b:Int):String`删除参数放入大括号内，`return`删除，自动推断最后一句话为返回值

```kotlin
var fun2 : (Int,Int)->String = {a:Int,b:Int->
     "测试$a+$b"
}
```
**接下来有俩种优化分类**

* 第一种优化，既然前面已经游客参数类型，那么后面的表达式不如直接删除参数类型
```kotlin
var fun4 : (Int,Int)->String = {a,b->
    "测试$a+$b"
}
```
* 第二种优化：既然从后面的表达式可以推断出参数和返回值，那么不如直接直接把`: (Int,Int)->String`删除
  ```kotlin
  var fun3 = {a:Int,b:Int->
    "测试$a+$b"
    } 
  ```
  这种也是常用的形式


这就是一个lambda表达式的理解，当我们看到一个lambda表达式的时候完全可以反向推导他的完整表达式是什么，然后就很好理解了

# 高阶函数

定义：在参数或者返回值含有函数类型的就是高阶函数

```kotlin
fun test(ation: (a: Int, b: Int) -> String): String {
    return ation(1, 2)
}
```
比如这个函数test,参数是一个函数类型

调用

```kotlin
test({ a, b -> "haha$a+$b" })
//简化写法，直接把小括号省略
test { a, b -> "haha$a+$b" }
```

```kotlin
fun test1(a:Int):()->String{
    return {
        "返回了$a"
    }
}
···
这个返回值为函数类型

调用

```kotlin
 val bb  = test1(1)
println(bb())
```

## 常用的几个高阶函数

### let
直接上源码

`T.let`  这个就是扩展函数

```kotlin
public inline fun <T, R> T.let(block: (T) -> R): R {
    ...
    return block(this)
}
```
* 接收泛型<T,R>
* 参数是一个函数类型，参数为泛型T，返回值为泛型R
* let的返回值为泛型R
* let方法内部其实是直接调用block函数，并返回

看下使用

```kotlin
    val ff = FF()

    ff.let {
        "sdas"
    }
```
首先看下上面这段代码中泛型<T，R>分别代表什么

* ff调用的let方法，所以T的类型为FF
* lambda中最终返回是String 所以R的类型为String
  
#### let的作用

let其实有俩个作用

* 定义一些变量在特定的作用域内
```kotlin
class FF {
    var  aa :Int = 2
}
```
```kotlin
    ff.let {
        //这个变量的作用域就是在let的范围内
        var i :String = "22"
        "sdas"
    }
```
* 避免一些判空操作
```kotlin
ff?.let {
        //这里的it值得就是ff
        //利用?.操作符，如果ff为空就不会进入let内，也就是说let内的it永远不会为空
        "${it.aa}"
    }
```

### with

同样的操作直接上源码

```kotlin
//这里特别注意block 参数 T.() -> R,这里的this代表的是T类型
public inline fun <T, R> with(receiver: T, block: T.() -> R): R {
    ...
    return receiver.block()
}
```
* 接收泛型T,R
* 接收参数T,函数类型block（无参数且返回值为R）
* with的返回值为R
* with内部直接调用block匿名函数

调用

```kotlin
 with(ff,{
        this?.aa
    })
```
* 首先T泛型为FF类型
* this值得是FF类型
* R泛型表示lambda中的返回值this?.aa，也就是Int类型

#### 作用
适用于一个类，有多个方法和属性的情况，可以省去类名重复，直接调用方法即可

```kotlin
class FF {
    var  aa :Int = 2
    
    fun ffa(){
        
    }
    
    fun ffb(){
        
    }
    
    fun ffc(){
        
    }
}

fun main() {
    val ff :FF =FF()
    with(ff,{
        ffa()
        ffb()
        ffc()
    })
}
```

### run

上源码

```kotlin
/扩展函数
//参数是一个带接收者的函数类型，T.()->R 里的 this 代表T
public inline fun <T, R> T.run(block: T.() -> R): R {
    ...
    return block()
}
//普通函数
public inline fun <R> run(block: () -> R): R {
    ...
    return block()
}
```
#### 作用

扩展函数其实是let和with的结合体
普通run函数，作用就是单独的作用域

### also

先看源码
```kotlin
public inline fun <T> T.also(block: (T) -> Unit): T {
    ...
    block(this)
    return this
}
```
这个和let很像，区别是，这个的返回值是T,返回原来的对象

### apply

```kotlin
public inline fun <T> T.apply(block: T.() -> Unit): T {
    ...
    block()
    return this
}
```
和run函数很像，区别就是这个返回的对象本省T






## 多个参数

```kotlin 
fun test2(a:Int,ation: (a: Int, b: Int) -> String): String {
    return ation(1, 2)
}
```

当多个参数时，lambda可以在括号外部也可用在内部

```kotlin
    test2 (1){a: Int, b: Int -> "" }

    test2(1,{a: Int, b: Int -> "" })
```

## ::的作用

::的作用是获取引用，如下
```kotlin
fun test2(a: Int, ation: (a: Int, b: Int) -> String): String {
    return ation(1, 2)
}
//匿名函数
var fun5 = { a: Int, b: Int -> "" }

//具名函数，正常的函数
fun fun6(a: Int, b: Int): String {
    return "11$a+$b"
}
```
这个test2方法有一个函数参数，那么调用test2的时候可以这样

```kotlin
  //获取fun6函数的引用
    test2(22, ::fun6)
    //直接传入匿名函数变量
    test2(22, fun5)
    //直接使用lambda表达式
    test2(1) { a: Int, b: Int -> "" }
```

## lambda表达式的return

除非使用标签，否则return从最近使用的fun关键字出返回

## SAM转换

SAM = 唯一抽象方法

SAM转换是为了在kotlin代码中调用Java代码提供的语法糖，是为了Java的单一方法接口，提供的lambda形式的实现，例如Android中的onClick的实现

```kotlin
// SAM convert in KT
view.setOnClickListener{ 
    view -> doSomething
}

// Java接口
public interface OnClickListener { 
     void onClick(View v); 
}

```
## 带接收者的lambda

普通的lambda  `()->R`,这个表示无参返回值为R类型

待接收者的lambda `T.()->R`，这个表示T类型的扩展函数是`lambda函数`

这俩个的主要区别
* 普通的lambda 的this值得是外部实例
* 待接收者的lambda，由于这个lambda是T类型的扩展函数，所以this值得是T类型

## 闭包

在Kotlin中变量的作用域

* 全局变量：在函数内外都可以访问的属性
* 局部变量：只能在函数内部访问的变量

那么问题来了，有没有办法在函数的外部访问，函数的局部变量，这个就需要闭包了，闭包就可以让函数的外部访问函数内部的变量

比如

```kotlin 
fun text33(): () -> Int {
    var a: Int = 1
    return { a++ }
}
```
调用

```kotlin
    val result = text33()

    println(result())
    println(result())
```
打印日志
```kotlin
1
2
```

可以看到这样就可以访问到局部变量的a了

## kotlin的接口回调

java思想的回调写法

```kotlin
interface CallBack {
    fun success(msg: String)
    fun fail(msg: String)
}


class GG {
    var mycallBack: CallBack? = null
    fun setCallBack(callBack: CallBack) {
        mycallBack = callBack
    }

    fun init() {
        mycallBack?.success("success")
        mycallBack?.fail("fail")
    }
}

fun main() {
    val gg = GG()

    gg.setCallBack(object : CallBack {
        override fun success(msg: String) {
            println(msg)
        }

        override fun fail(msg: String) {
            println(msg)
        }
    })

    gg.init()
    
}
```
利用kotlin中的lambda实现callback

```kotlin
class HH {
    var successCallback:((String)->Unit)?=null
    var failCallback:((String)->Unit)?=null

    fun setCallback(success:(String)->Unit,fail:(String)->Unit){
        successCallback=success
        failCallback=fail
    }

    fun init(){
        successCallback?.let {
            it("success11")
        }

        failCallback?.invoke("fail11")
    }
}

```
调用
```kotlin
fun main() {
    val hh = HH()

    hh.setCallback({
       println(it)
    }, {
        println(it)

    })

    hh.init()
}
```
打印

```kotlin
success11
fail11
```





