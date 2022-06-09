# 继承

kotlin中所有的类都有一个共同的超类 `Any`和java中的`object`一样的,默认情况下，kotlin类都是`public final`的不可继承的，如果让一个类可被继承咋需要用`open`关键字修饰

```kotlin
//表示可被继承
open class Base {
}
```
`kotlin` 中使用` ： `来表示继承,类似java中的`extends`

## 如果子类有主构造函数

如果子类有主构造函数，那么父类需要在主构造函数中立即初始化
```kotlin
open class Base(a :Int) {
    open fun text(){
        
    }
}

class MyBase(a : Int ) :Base(a) {

    override fun text() {

    }
}
```

## 如果子类没有主构造函数
如果子类中没有主构造函数，则需要在次构造函数中使用surper关键字初始化基类，或者委托=给另一个构造函数，注意在这种情况下，不同的次构造函数，可以调用基类的不同构造函数

```kotlin
class MyView : View {

    constructor(con: Context) : super(con){

    }

    constructor(con: Context,abb:AttributeSet) : super(con,abb){

    }
}
```

##  覆盖方法

方法默认都是`final`不可继承的,如果需要继承就需要使用`open`关键字，子类重写方法需要使用`override`修饰，不然会报错，`override`本身就是开放的，也就是说子类可以重写`override`标识的方法


```kotlin
open class Base(a :Int) {
    open fun text(){
        
    }
}

class MyBase(a : Int ) :Base(a) {

    override fun text() {

    }
}
```

如果在`final`类中的方法用`open`修饰，是不起作用的

## 覆盖属性

属性和方法类似，重写属性也需要用`override`来标识

```kotlin 
open class Base(a :Int) {
    open var text : Int =1
}

class MyBase(a: Int) : Base(a) {
    override var text: Int = 2
}
```
我们也可以用 `var` 重写`val` ，但是反之则不行，因为`val`其实是有`get`，没有`set`，用`var`重写只是增加了一个`set`方法

# 抽象类

类可以声明为抽象类`abstract`,抽象类作为子类的模板，可以作为公共方法的模板

* 抽象类，抽象属性，抽象方法都不需要`open`修饰,抽象方法默认就是open，open和abstract不能共存
* 抽象类不能实例化，但是可以有构造方法，构造方法是给子类用的
* 抽象类可以包含：属性（抽象属性或非抽象属性），方法（抽象方法和非抽象方法），构造器和初始化块，嵌套类（接口么枚举），5中成员
* abstract不能修饰局部变量
  
## 抽象成员
  * 抽象方法不能有方法体
  * 抽象成员不能初始化

## 定义抽象类
```kotlin
abstract class A {
    //抽象成员变量不能初始化
    abstract var A :Int
    //抽象方法不能有方法体
    abstract fun text()
    //普通属性也可以被重写，按时需要有open修饰
    open var B :Int  =1
    //普通方法也可以被重写，需要open修饰
    open fun text1(){

    }

    constructor(){
        //构造方法，抽象类不能实例化，构造方法是给子类用的
    }
}
```

# 密封类

密封类是一种特殊的抽象类，专门用来派生子类，使用`sealed`修饰

## 特点
密封类的子类是固定的，原因如下
* 密封类的子类必须和密封类在同一个文件夹
* 在其他文件夹中，不能为密封类的子类（但是密封# 类的子类可以被其他类继承）

```kotlin
sealed class B {
    abstract fun text()
}
```

# 接口

定义了系统与外界交互的规则，规定了实现者必须要向外界提供某些方法

## 特点
* 修饰符默认是public
* 接口只能继承接口，不能继承类
* 接口中可以定义抽象方法，和非抽象方法
* 接口中可以定义抽象属性和非抽象属性，但是接口中是没有幕后字段，所以需要提供get-set方法
* 接口中的非抽象成员（方法属性），可以用public|private俩种修饰符，（java自动为成员为public，如果手动指定也只能是public），抽象成员只能为public修饰
* 接口不包含构造方法和初始器，但是接口可以包括方法（抽象/非抽象方法）,属性（抽象/非抽象属性）嵌套类（嵌套将接口和枚举）
* kotlin接口抽象成员的`abstract`关键字可以省略(抽象类的中`abstract`不可以省略)
* 接口可以多继承
* 接口不能实例化，但是可以赋值变量
```kotlin
 interface A {
    //抽象属性 abstract可以省略
    abstract var a: Int
    //非抽象属性 ，没啥用
    var b : Int
        get() = 1
        set(value) = TODO()

    //抽象方法 abstract可以使用
    abstract fun text()
    
    //非抽象方法
    fun text1(){

    }
}
```
  
# 接口和抽象类的区别

## 语法上的区别
* 抽象类，有构造方法
* 接口没有构造方法
* 接口可以多继承
  
## 设计上的区别

*  抽象类表示 `是不是` 的关系
*  接口表示的 `有没有` 的关系 
* 接口是对行为的抽象
* 抽象类是对类的整体进行的抽象，比如我们把一些通用方法抽取到抽象类中
* 比如 `飞机` 和 `鸟 ` 他们都有飞行的能力，可以设计接口 `Fly`  ,如果是`战斗机`，`蜂鸟`就可以直接继承抽象类`飞机`，和`鸟`


