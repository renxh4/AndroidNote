# 概述
Kotlin 可以对一个类的属性和方法进行扩展,扩展不会对原有的类有影响

## 扩展方法

扩展方法可以在已有的类添加新的方法，不会对原有的类有影响

写法如下：

```kotlin 
fun receiverType.functionName(params){
    body
}
```
* receiverType：表示函数接收者，也就是要扩展方法的类名
* functionName：表示扩展函数的名称
* params：表示扩展函数的参数，可以为null
  
下面是个例子：

```kotlin 
class A(var name111: String) {

}

fun A.printMsg() {
    println("打印 $name111")
}


fun main() {
    A("小明").printMsg()

}
```

可以看到类`A`没有任何的方法，然后利用`kotlin`的特性，给A扩展方法`printMsg`，然后就可以通过A的对象去调用扩展方法了

### 看下反编译java代码

```java
public final class AKt {
   public static final void printMsg(@NotNull A $this$printMsg) {
      Intrinsics.checkNotNullParameter($this$printMsg, "$this$printMsg");
      String var1 = "打印 " + $this$printMsg.getName111();
      boolean var2 = false;
      System.out.println(var1);
   }

   public static final void main() {
      printMsg(new A("小明"));
   }

   // $FF: synthetic method
   public static void main(String[] var0) {
      main();
   }
}
// A.java


public final class A {
   @NotNull
   private String name111;

   @NotNull
   public final String getName111() {
      return this.name111;
   }

   public final void setName111(@NotNull String var1) {
      Intrinsics.checkNotNullParameter(var1, "<set-?>");
      this.name111 = var1;
   }

   public A(@NotNull String name111) {
      Intrinsics.checkNotNullParameter(name111, "name111");
      super();
      this.name111 = name111;
   }
}
```

可以看到其实就是加了一个静态方法，然后传入A的对象

## 扩展函数没有多态
直接上代码

```kotlin 
open class C

class D : C()

fun C.text(){
    println("我是c")
}

fun D.text(){
    println("我是d")
}

fun haha(c:C){
    c.text()
}

fun main(){
    haha(C())
    haha(D())
}
```
输出

```kotlin 
我是c
我是c
```

## 如果扩展函数和成员函数重名，优先使用成员函数

```kotlin 
class E {
    fun text(){
        println("成员函数")
    }
}

fun E.text(){
    println("扩展方法")
}

fun main(){
    E().text()
}
```
输出

```kotlin
成员函数
```

## 扩展空对象

```kotlin 
class F {
    companion object {
        fun get(): F? {
            return null
        }
    }
}

fun F?.text(): String {
    if (this == null) {
        return "空的"
    }

    return "不为空"
}

fun main() {

    var f = F.get()

    println(f.text())

}
```
打印

```kotlin 
空的 
```

## 扩展属性

```java

class E

var E.name: Int
    get() = 1
    set(value) {
        println(value)
    }



fun main() {
    E().name=111
    println(E().name)
}
```

需要注意的是，扩展属性没有幕后字段，所以需要实现get和set方法

### 反编译java代码

```java
public final class EKt {
   public static final int getName(@NotNull E $this$name) {
      Intrinsics.checkNotNullParameter($this$name, "$this$name");
      return 1;
   }

   public static final void setName(@NotNull E $this$name, int value) {
      Intrinsics.checkNotNullParameter($this$name, "$this$name");
      boolean var2 = false;
      System.out.println(value);
   }

   public static final void main() {
      setName(new E(), 111);
      int var0 = getName(new E());
      boolean var1 = false;
      System.out.println(var0);
   }

   // $FF: synthetic method
   public static void main(String[] var0) {
      main();
   }
}
// E.java



public final class E {
}
```

其实也是再顶层增加了set和get的静态方法，然后把E对象传入






