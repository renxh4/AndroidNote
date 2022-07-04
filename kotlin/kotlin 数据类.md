# 数据类

```kotlin 
data class A(var name: String, var age: Int)
```

这就是一个简单的数据类，看下反编译成java代码会是什么样

```java
public final class A {
   @NotNull
   private String name;
   private int age;

   @NotNull
   public final String getName() {
      return this.name;
   }

   public final void setName(@NotNull String var1) {
      Intrinsics.checkNotNullParameter(var1, "<set-?>");
      this.name = var1;
   }

   public final int getAge() {
      return this.age;
   }

   public final void setAge(int var1) {
      this.age = var1;
   }

   public A(@NotNull String name, int age) {
      Intrinsics.checkNotNullParameter(name, "name");
      super();
      this.name = name;
      this.age = age;
   }

   @NotNull
   public final String component1() {
      return this.name;
   }

   public final int component2() {
      return this.age;
   }

   @NotNull
   public final A copy(@NotNull String name, int age) {
      Intrinsics.checkNotNullParameter(name, "name");
      return new A(name, age);
   }

   // $FF: synthetic method
   public static A copy$default(A var0, String var1, int var2, int var3, Object var4) {
      if ((var3 & 1) != 0) {
         var1 = var0.name;
      }

      if ((var3 & 2) != 0) {
         var2 = var0.age;
      }

      return var0.copy(var1, var2);
   }

   @NotNull
   public String toString() {
      return "A(name=" + this.name + ", age=" + this.age + ")";
   }

   public int hashCode() {
      String var10000 = this.name;
      return (var10000 != null ? var10000.hashCode() : 0) * 31 + Integer.hashCode(this.age);
   }

   public boolean equals(@Nullable Object var1) {
      if (this != var1) {
         if (var1 instanceof A) {
            A var2 = (A)var1;
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
* 首先就是主动添加了参数的`get和set`方法
* 然后添加了`toString，hashCode，equals`方法
* 天机了`copy`方法，其实就是new了一个新的对象返回
* `componentN`方法就是按照顺序返回字段

生成这些其实是有要求的，要求如下：

* 主构造函数至少包含一个参数
* 所有的主构造函数必须是var或val
* 数据类不能声明为 abstract, open, sealed 或者 inner
* 数据类不能继承其他类，但是可以实现接口
  

## 使用copy方法

```kotlin
data class A(var name: String, var age: Int)


fun main() {
    var a: A = A("小明", 2)

    println(a)

    var newa = a.copy("小红")

    println(newa)
}

```
输出

```java
A(name=小明, age=2)
A(name=小红, age=2)
```

看下反编译java代码

```java
public final class AKt {
   public static final void main() {
      A a = new A("小明", 2);
      boolean var1 = false;
      System.out.println(a);
      A newa = A.copy$default(a, "小红", 0, 2, (Object)null);
      boolean var2 = false;
      System.out.println(newa);
   }

   // $FF: synthetic method
   public static void main(String[] var0) {
      main();
   }
}
```

## 解构声明

```koltin 

fun main() {
    var a: A = A("小明", 2)

    println("name = ${a.name}")

    println("age = ${a.age}")

    //这个就是解构声明
    var (name, age) = a

    println("name = $name")
    println("age = $age")
}
```
输出

```koltin 
name = 小明
age = 2
name = 小明
age = 2
```

看下java代码

```java
public final class AKt {
   public static final void main() {
      A a = new A("小明", 2);
      String name = "name = " + a.getName();
      boolean var2 = false;
      System.out.println(name);
      name = "age = " + a.getAge();
      var2 = false;
      System.out.println(name);
      name = a. ();
      int age = a.component2();
      String var3 = "name = " + name;
      boolean var4 = false;
      System.out.println(var3);
      var3 = "age = " + age;
      var4 = false;
      System.out.println(var3);
   }

   // $FF: synthetic method
   public static void main(String[] var0) {
      main();
   }
}
```
其实是利用了`componentN`方法


# 密封类

密封类是一种特殊的抽象类，专门用来派生子类，使用`sealed`修饰

## 特点
密封类的子类是固定的，原因如下
* 密封类的子类必须和密封类在同一个文件夹
* 在其他文件夹中，不能为密封类的子类（但是密封类的子类可以被其他类继承）

```kotlin
sealed class B {
    abstract fun text()
}
```

## 用途

```kotlin
interface A

class B : A

class C : A


fun text1(a:A) = when(a){
    is B -> println("is b")
    is C -> println("is c")
    else -> throw IllegalArgumentException("没有此类型")
}

```

由于kotlin编译器问题，这个必须有else分支，不然编译报错，这样写起来不是很优雅，这个时候我们可以借助密封类来简化

```kotlin
sealed class A

class B : A()

class C : A()


fun text1(a:A) = when(a){
    is B -> println("is b")
    is C -> println("is c")
}
```

这个时候就可以省去这个不必要的else分支




