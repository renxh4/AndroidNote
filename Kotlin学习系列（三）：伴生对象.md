# 概述

# 如何反编译kt文件成java文件

## 1 点击Android Studio Tools

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/65bff9b2f136497f98289ebadc6b7a30~tplv-k3u1fbpfcp-watermark.image?)

## 点击Decompile

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2c92b4471b8647819b24e3e8b3ec83c1~tplv-k3u1fbpfcp-watermark.image?)

## kt文件

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5fa3dd27bc404702b07d9070685e5cf0~tplv-k3u1fbpfcp-watermark.image?)

## 反编译后的java文件

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/19707600804d44fcb5d862a26aa98ce1~tplv-k3u1fbpfcp-watermark.image?)

有了这个操作，就可以很快的理解kotlin中的概念了

# 包级函数，包级属性

kotlin和java有个地方不同，就是函数和属性可以不需要定义再类里面

## kotlin 定义一个包级函数,和包级属性

```kotlin 
const val name : String  = "hahha"

var age :Int = 12


fun saySome(){
    println("说些什么")
}



class AAA {
    fun A(){
        saySome()

        println(name+ age)
    }
}
```

反编译为java

```java
public final class AAA {
   public final void A() {
      AAAKt.saySome();
      String var1 = "hahha" + AAAKt.getAge();
      boolean var2 = false;
      System.out.println(var1);
   }
}
// AAAKt.java
package com.example.lib.hanshu;

import kotlin.Metadata;
import org.jetbrains.annotations.NotNull;


public final class AAAKt {
   @NotNull
   public static final String name = "hahha";
   private static int age = 12;

   public static final int getAge() {
      return age;
   }

   public static final void setAge(int var0) {
      age = var0;
   }

   public static final void saySome() {
      String var0 = "说些什么";
      boolean var1 = false;
      System.out.println(var0);
   }
}
```

可以看到编译后称为了俩个类 `AAA.java ` `AAAKt.java`,包级函数和属性，都编译为AAAKt.java的静态属性和静态方法了，外部表调用其实也是调用的 `AAAKt.saySome()`的静态方法

也就是说包级的属性和方法，最终被编译为静态方法和属性



# object对象

kotlin中对象指的是 使用` object`关键字定义的类型声明，一般用作`单例模式和伴生对象`，他让单例更加简单

```kotlin
object Text {
    var name: String = "xiaoming"

    fun get() {
        println("hhaha")
    }
}

fun main(){
    Text.get()
}
```

上面代码分编译成java，如下：

```java
public final class Text {
   @NotNull
   private static String name;
   @NotNull
   public static final Text INSTANCE;

   @NotNull
   public final String getName() {
      return name;
   }

   public final void setName(@NotNull String var1) {
      Intrinsics.checkNotNullParameter(var1, "<set-?>");
      name = var1;
   }

   public final void get() {
      String var1 = "hhaha";
      boolean var2 = false;
      System.out.println(var1);
   }

   private Text() {
   }

   static {
      Text var0 = new Text();
      INSTANCE = var0;
      name = "xiaoming";
   }
}
// TextKt.java

public final class TextKt {
   public static final void main() {
      Text.INSTANCE.get();
   }

   // $FF: synthetic method
   public static void main(String[] var0) {
      main();
   }
}

```
可以看待object关键字定义的类，其实就是一个单例类


# 伴生对象

伴生对象更合适的理解就是，用一个静态工厂创建对象

```kotlin
class Myclass {

    var name: String = "aaa"

    companion object Factory {
        fun create(): Myclass = Myclass()
    }

    fun text() {
        println("调用方法")
    }
    
}

fun main() {
    var myclass = Myclass.create()
    //上下俩个表达式等价
    var myclass1 = Myclass.Factory.create()

    println(myclass.name)

    myclass1.text()
}
```

输出

```java
aaa
调用方法
```

反编译成 java来看

```java
public final class MyclassKt {
   public static final void main() {
      Myclass myclass = Myclass.Factory.create();
      Myclass myclass1 = Myclass.Factory.create();
      String var2 = myclass.getName();
      boolean var3 = false;
      System.out.println(var2);
      myclass1.text();
   }

   // $FF: synthetic method
   public static void main(String[] var0) {
      main();
   }
}
// Myclass.java


public final class Myclass {
   @NotNull
   private String name = "aaa";
   @NotNull
   public static final Myclass.Factory Factory = new Myclass.Factory((DefaultConstructorMarker)null);

   @NotNull
   public final String getName() {
      return this.name;
   }

   public final void setName(@NotNull String var1) {
      Intrinsics.checkNotNullParameter(var1, "<set-?>");
      this.name = var1;
   }

   public final void text() {
      String var1 = "调用方法";
      boolean var2 = false;
      System.out.println(var1);
   }

   public static final class Factory {
      @NotNull
      public final Myclass create() {
         return new Myclass();
      }

      private Factory() {
      }

      // $FF: synthetic method
      public Factory(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}
```
其实就是再`Myclass`类中建立了一个`Factory`类，然后内部持有静态的`Facetory`对象，最后就是 Myclass.Factory.create()，来获取`Myclass`对象


## 省略伴生对象的名称

下面这个例子，就是伴生对象没有名称
```kotlin
class Myclass1 {
    companion object {
        fun text(){
            println("text")
        }
    }
}


fun main(){
    Myclass1.Companion.text()
    //上下俩个表达式等价
    Myclass1.text()
}
```

可以看到可以直接 `Myclass1.text()`类名调用方法，不需要对象，像极了Java的中的静态方法，其实他的作用也是要达到java中的静态方法的作用

### 反编译成java

```java
public final class Myclass1 {
   @NotNull
   public static final Myclass1.Companion Companion = new Myclass1.Companion((DefaultConstructorMarker)null);


   public static final class Companion {
      public final void text() {
         String var1 = "text";
         boolean var2 = false;
         System.out.println(var1);
      }

      private Companion() {
      }

      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}
// Myclass1Kt.java

public final class Myclass1Kt {
   public static final void main() {
      Myclass1.Companion.text();
      Myclass1.Companion.text();
   }

   // $FF: synthetic method
   public static void main(String[] var0) {
      main();
   }
}
```
看Java代码更加直观一些，应该都能看懂，不在赘述


## @JvmStatic

用这个标记会编译成静态方法

比如，我们上方的第一个例子，改造一下

```java
object Text {
    var name: String = "xiaoming"
    //这里加上@JvmStatic
   @JvmStatic
    fun get() {
        println("hhaha")
    }
}

fun main(){
    Text.get()
}
```

反编译成java


```java
public final class Text {
   @NotNull
   private static String name;
   @NotNull
   public static final Text INSTANCE;

   @NotNull
   public final String getName() {
      return name;
   }

   public final void setName(@NotNull String var1) {
      Intrinsics.checkNotNullParameter(var1, "<set-?>");
      name = var1;
   }

   //注释1 
   @JvmStatic
   public static final void get() {
      String var0 = "hhaha";
      boolean var1 = false;
      System.out.println(var0);
   }

   private Text() {
   }

   static {
      Text var0 = new Text();
      INSTANCE = var0;
      name = "xiaoming";
   }
}
// TextKt.java
package com.example.lib.danli;

import kotlin.Metadata;


public final class TextKt {
   public static final void main() {
      Text.get();
   }

   // $FF: synthetic method
   public static void main(String[] var0) {
      main();
   }
}
```

看注释一出，被`@JvmStatic`标记的方法变成了静态方法

## 替代java中的匿名内部类

首先看下java中的匿名内部类

```java
public class A {

    public void setClick(Click click){

    }

    public interface Click{
        void click();
    }


    public static void main(String[] args) {
        new A().setClick(new Click() {
            @Override
            public void click() {

            }
        });
    }
}
```

在Kotlin中借助object来实现

``` kotlin
interface B {
    fun click();
}

fun setClick(click: B) {

}

fun main() {
    setClick(object : B {
        override fun click() {

        }
    })
}
```








