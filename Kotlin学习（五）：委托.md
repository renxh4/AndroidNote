

# 类委托

类委托其实对应于Java中的代理模式

```kotlin 
interface Base{
    fun text()
}

//被委托的类(真实的类)
class BaseImpl(val x :String ) : Base{
    override fun text() {
        println(x)
    }
}

//委托类（代理类）
class Devices (b :Base) :Base by b


fun main(){
    var b = BaseImpl("我是真实的类")
    Devices(b).text()
}
```

输出

```kotlin
 我是真实的类
```

可以看到，`委托类(代理类)`持有`真实类`的对象，然后`委托类（代理类）`调用`真实类`的同名方法，最终真正实现的是方法的是`真实类`，这其实就是`代理模式`

kotlin中委托实现借助于by关键字，by关键字后面就是被委托类，可以是一个表达式

## 反编译成java代码看下

```java
public final class Devices implements Base {
   // $FF: synthetic field
   private final Base $$delegate_0;

   public Devices(@NotNull Base b) {
      Intrinsics.checkNotNullParameter(b, "b");
      super();
      this.$$delegate_0 = b;
   }

   public void text() {
      this.$$delegate_0.text();
   }
}



public final class BaseImpl implements Base {
   @NotNull
   private final String x;

   public void text() {
      String var1 = this.x;
      boolean var2 = false;
      System.out.println(var1);
   }

   @NotNull
   public final String getX() {
      return this.x;
   }

   public BaseImpl(@NotNull String x) {
      Intrinsics.checkNotNullParameter(x, "x");
      super();
      this.x = x;
   }
}


public interface Base {
   void text();
}


public final class BaseKt {
   public static final void main() {
      BaseImpl b = new BaseImpl("我是真实的类");
      (new Devices((Base)b)).text();
   }

   // $FF: synthetic method
   public static void main(String[] var0) {
      main();
   }
}
```

可以看到其实就是代理模式，`Devices`持有`BaseImpl`对象，重写`text`方法，`text`方法内部调用`BaseImpl.text()`

# 属性委托

属性委托和类委托一样，属性的委托其实是对属性的`set/get`方法的委托,把`set/get`方法委托给`setValue/getValue`方法,因此被委托类（真实类）需要提供`setValue/getValue`方法，val属性只需要提供`setValue`方法

属性委托语法：

`val/var <属性名>: <类型> by <表达式>`


```kotlin 

class B {
    //委托属性
    var a : String by Text()
}

//被委托类（真实类）
class Text {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "属性拥有者 = $thisRef, 属性的名字 = '${property.name}' 属性的值 "
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("属性的值 = $value 属性的名字 =  '${property.name}' 属性拥有者 =  $thisRef")
    }
}

fun main(){
    var b = B()

    println(b.a)

    b.a = "ahaha"
    
}
```

输出

```kotlin 
属性拥有者 = com.example.lib.weituo.B@27fa135a, 属性的名字 = 'a' 属性的值 
属性的值 = ahaha 属性的名字 =  'a' 属性拥有者 =  com.example.lib.weituo.B@27fa135a
```

上面的例子可以看到 ，属性a 委托给了Text,而且Text类中有`setValue和getValue`,所以当我们调用属性`a`的`get/set`方法时候，会委托到`Text`的`setValue/getValue`上,上面的例子可以看出来,里面有几个参数介绍一下：

`thisRef：`属性的拥有者

`property：`对属性的描述，是KProperty<*>类型或者父类

`value：`属性的值

## 反编译成Java代码看下

```java
public final class B {
   // $FF: synthetic field
   static final KProperty[] $$delegatedProperties = new KProperty[]{(KProperty)Reflection.mutableProperty1(new MutablePropertyReference1Impl(B.class, "a", "getA()Ljava/lang/String;", 0))};
   @NotNull
   private final Text a$delegate = new Text();

   @NotNull
   public final String getA() {
      return this.a$delegate.getValue(this, $$delegatedProperties[0]);
   }

   public final void setA(@NotNull String var1) {
      Intrinsics.checkNotNullParameter(var1, "<set-?>");
      this.a$delegate.setValue(this, $$delegatedProperties[0], var1);
   }
}


public final class BKt {
   public static final void main() {
      B b = new B();
      String var1 = b.getA();
      boolean var2 = false;
      System.out.println(var1);
      b.setA("ahaha");
   }

   // $FF: synthetic method
   public static void main(String[] var0) {
      main();
   }
}


public final class Text {
   @NotNull
   public final String getValue(@Nullable Object thisRef, @NotNull KProperty property) {
      Intrinsics.checkNotNullParameter(property, "property");
      return "属性拥有者 = " + thisRef + ", 属性的名字 = '" + property.getName() + "' 属性的值 ";
   }

   public final void setValue(@Nullable Object thisRef, @NotNull KProperty property, @NotNull String value) {
      Intrinsics.checkNotNullParameter(property, "property");
      Intrinsics.checkNotNullParameter(value, "value");
      String var4 = "属性的值 = " + value + " 属性的名字 =  '" + property.getName() + "' 属性拥有者 =  " + thisRef;
      boolean var5 = false;
      System.out.println(var4);
   }
}

```

可以看到`B`类持有`Text`对象,当调用`B.get()`方法，内部调用了`Text.getValue()`,`B`中创建了`KProperty`来保存属性的各种参数


## 更加简单的实现属性委托

每次实现委托都要写`getValue/setValue`方法,这就比较麻烦了，系统为我们提供了接口，方便我们重写这些方法,`ReadOnlyProperty和ReadWriteProperty`

```kotlin 
interface ReadOnlyProperty<in R, out T> {
    operator fun getValue(thisRef: R, property: KProperty<*>): T
}

interface ReadWriteProperty<in R, T> {
    operator fun getValue(thisRef: R, property: KProperty<*>): T
    operator fun setValue(thisRef: R, property: KProperty<*>, value: T)
}

```

被委托类只需要实现接口重写方法就行，`val`继承`ReadOnlyProperty`

```kotlin 
class Text1 : ReadOnlyProperty<Any, String> {
    override fun getValue(thisRef: Any, property: KProperty<*>): String {
        return "属性拥有者 = $thisRef, 属性的名字 = '${property.name}' 属性的值 "
    }
}

class Text2 : ReadWriteProperty<Any, String> {
    override fun getValue(thisRef: Any, property: KProperty<*>): String {
        return "属性拥有者 = $thisRef, 属性的名字 = '${property.name}' 属性的值 "
    }

    override fun setValue(thisRef: Any, property: KProperty<*>, value: String) {
        println("属性的值 = $value 属性的名字 =  '${property.name}' 属性拥有者 =  $thisRef")

    }

}

class B {

    val b :String by Text1()

    val c :String by Text2()

}
```

# Koltin 标准库中提供的几个委托

* 延迟属性（lazy properties）：其值只在访问时计算
* 可观察属性（observable properties）：监听器会收到此属性的变更通知
* 把多个属性映射到(Map)中，而不是存在单个字段

## 延迟属性lazy

lazy()接收一个lambda，返回Lazy实例，返回的实例可以作为实现延迟属性的委托，`仅在第一次调用属性进行初始化`

```kotlin 
class Lazy {
    val name :String by lazy {
        println("第一次调用初始化")
        "aa" }
}

fun main(){
    var lazy =Lazy()
    println(lazy.name)
    println(lazy.name)
    println(lazy.name)
}
```
输出

```koltin 
第一次调用初始化
aa
aa
aa
```

### 反编译Java代码

```java
public final class Lazy {
   @NotNull
   private final kotlin.Lazy name$delegate;

   @NotNull
   public final String getName() {
      kotlin.Lazy var1 = this.name$delegate;
      Object var3 = null;
      boolean var4 = false;
      return (String)var1.getValue();
   }

   public Lazy() {
      this.name$delegate = kotlin.LazyKt.lazy((Function0)null.INSTANCE);
   }
}
// LazyKt.java

public final class LazyKt {
   public static final void main() {
      Lazy lazy = new Lazy();
      String var1 = lazy.getName();
      boolean var2 = false;
      System.out.println(var1);
      var1 = lazy.getName();
      var2 = false;
      System.out.println(var1);
      var1 = lazy.getName();
      var2 = false;
      System.out.println(var1);
   }

   // $FF: synthetic method
   public static void main(String[] var0) {
      main();
   }
}
```

可以看到Lazy再初始化时生成了`name$delegate`变量是`kotlin.Lazy`类型的,而`getName()`方法返回的值其实就是`name$delegate.getValue()`

`name$delegate` 是由`kotlin.LazyKt.lazy((Function0)null.INSTANCE)`生成，我们看下源码

```kotlin 
public actual fun <T> lazy(initializer: () -> T): Lazy<T> = SynchronizedLazyImpl(initializer)
```
最终是由`SynchronizedLazyImpl`生成,继续跟下源码

```kotlin
private class SynchronizedLazyImpl<out T>(initializer: () -> T, lock: Any? = null) : Lazy<T>, Serializable {
    private var initializer: (() -> T)? = initializer
    @Volatile private var _value: Any? = UNINITIALIZED_VALUE
    // final field is required to enable safe publication of constructed instance
    private val lock = lock ?: this

    override val value: T
        get() {
            val _v1 = _value
            if (_v1 !== UNINITIALIZED_VALUE) {
                @Suppress("UNCHECKED_CAST")
                return _v1 as T
            }

            return synchronized(lock) {
                val _v2 = _value
                if (_v2 !== UNINITIALIZED_VALUE) {
                    @Suppress("UNCHECKED_CAST") (_v2 as T)
                } else {
                    val typedValue = initializer!!()
                    _value = typedValue
                    initializer = null
                    typedValue
                }
            }
        }

    override fun isInitialized(): Boolean = _value !== UNINITIALIZED_VALUE

    override fun toString(): String = if (isInitialized()) value.toString() else "Lazy value not initialized yet."

    private fun writeReplace(): Any = InitializedLazyImpl(value)
}
```

我们可以直接看 `value的get`方法,如果`_v1 !== UNINITIALIZED_VALUE`则表明已经初始化过了，就直接返回value，否则表明没有初始化过，调用`initializer`方法，也就是`lazy的lambda`表达式


其实跟自己实现的委托差不多，也是实现了`getValue`方法

### lazy委托参数

```kotlin
public enum class LazyThreadSafetyMode {

    /**
     * Locks are used to ensure that only a single thread can initialize the [Lazy] instance.
     */
    SYNCHRONIZED,

    /**
     * Initializer function can be called several times on concurrent access to uninitialized [Lazy] instance value,
     * but only the first returned value will be used as the value of [Lazy] instance.
     */
    PUBLICATION,

    /**
     * No locks are used to synchronize an access to the [Lazy] instance value; if the instance is accessed from multiple threads, its behavior is undefined.
     *
     * This mode should not be used unless the [Lazy] instance is guaranteed never to be initialized from more than one thread.
     */
    NONE,
}
```
* SYNCHRONIZED:添加同步锁，使lazy延迟初始化线程安全
* PUBLICATION：初始化的lambda表达式，可以在同一时间多次调用，但是只有第一次的返回值作为初始化值
* NONE：没有同步锁，非线程安全

```kotlin
    val name :String by lazy(LazyThreadSafetyMode.SYNCHRONIZED) {
        println("第一次调用初始化")
        "aa" }
}
```

## 可观察属性Observable委托

可以观察一个属性的变化过程

```kotlin 
class Lazy {
   
    var a : String by Delegates.observable("默认值"){
            property, oldValue, newValue ->

        println( "$oldValue -> $newValue ")
    }
}

fun main(){
    var lazy =Lazy()


    lazy.a = "第一次修改"

    lazy.a = "第二次修改"
}
```

输出

```koltin 
默认值 -> 第一次修改 
第一次修改 -> 第二次修改 
```

## vetoable委托

`vetoable`和`Observable`一样,可以观察属性的变化，不同的是`vetoable`可以决定是否使用新值

```kotlin 
class C {
    var age: Int by Delegates.vetoable(0) { property, oldValue, newValue ->
        println("oldValue = $oldValue -> oldValue = $newValue" )
        newValue > oldValue


    }
}

fun main() {
    var c = C()

    c.age = 5
    println(c.age)

    c.age = 10
    println(c.age)

    c.age = 8
    println(c.age)

    c.age = 20
    println(c.age)
}
```
输出

```kotlin 

oldValue = 0 -> oldValue = 5
5
oldValue = 5 -> oldValue = 10
10
oldValue = 10 -> oldValue = 8
10
oldValue = 10 -> oldValue = 20
20
```

当新值小于旧值，那么就不生效，可以看到第三次设的值是8，小于10就没有生效

## 属性储存在Map中

```kotlin 
class D(val map: Map<String, Any?>) {
    val name: String by map
    val age: Int by map
}

fun main(){
    var d = D(mapOf(
        "name" to "小明",
        "age" to 12
    ))


    println("name = ${d.name},age = ${d.age}")


}
```

输出

```kotlin 
name = 小明,age = 12
```

























