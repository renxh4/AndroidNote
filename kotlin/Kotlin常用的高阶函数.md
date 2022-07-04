# 函数的集合操作

## filter

filter作用是：遍历集合并把每个元素放入lambda中，如果符合表达式则加入新集合，否则遗弃该元素

类似的还有`filterIndexed` 带index的过滤器，`filterNot`，过滤所有不满足条件的过滤器

```kotlin
        val test = listOf(1, 3, 5, 7, 9)
        // filter函数遍历集合并选出应用给定lambda后会返回true的那些元素
        println("大于5的数 ${test.filter { it > 5 }}")
```
输出

```kotlin
大于5的数 [7, 9]
```
## map

map的作用是：我理解对应的应该是变换，遍历集合中的每个数据，然后通过lambda进行数据变换，加入新的集合然后返回

```kotlin
        val test = listOf(1, 3, 5, 7, 9)

        //map函数,我理解对应的应该是变换，遍历集合中的每个数据，然后通过lambda进行数据变换，加入新的集合然后返回

        println("把集合中所有数据变为字符串${test.map { "haha$it" }}")
```
输出
```kotlin
把集合中所有数据变为字符串[haha1, haha3, haha5, haha7, haha9]
```
## all & any & count & find

```kotlin
        val test = listOf(1, 3, 5, 7, 9)

        // all判断是否全部符合lambda表达式的条件
        println("是否全部符合>10 ${test.all { it > 10 }}")

        // any判断是否存在有符合lambda表达式的条件的数据
        println("是否存在>8 ${test.any { it > 8 }}")

        // count获取符合lambda表达式条件的数据个数
        println("大于5的个数 ${test.count { it > 5 }}")

        // find获取符合lambda表达式条件的第一个数据
        println("第一个大于5 ${test.find { it > 5 }}")
        println("最后一个大于5 ${test.findLast { it > 5 }}")
```

输出

```kotlin
是否全部符合>10 false
是否存在>8 true
大于5的个数 2
第一个大于5 7
最后一个大于5 9
```
## groupBy

groupBy 作用：先遍历每个元素，然后把元素返给lambda，根据lambda的规则生成key，然后元素作为value组成新的map返回
```kotlin
        val test1 = listOf("a", "ab", "b", "bc")
        // groupBy 先遍历每个元素，然后把元素返给lambda，根据lambda的规则生成key，然后元素作为value组成新的map返回
        println("按首字母分组 ${test1.groupBy{it[0]}}")
```
输出
```kotlin
按首字母分组 {a=[a, ab], b=[b, bc]}
```

## partition

partition 的作用：按照条件进行分组，该条件只支持Boolean类型条件，first为满足条件的，second为不满足的

```kotlin
        val test1 = listOf("a", "ab", "b", "bc")
        println("满足条件的")
        test1.partition { it.length > 1 }.first.forEach { print("$it、") }
        println()
        println("不满足条件的")
        test1.partition { it.length > 1 }.second.forEach { print("$it、") }
```
输出

```kotlin
满足条件的
ab、bc、
不满足条件的
a、b、
```
## flatMap

flatMap 作用：首先遍历每个元素 按照lambda表达式对元素进行变换，再将变换后的列表合并成一个新列表
```kotlin
        // flatMap首先遍历每个元素 按照lambda表达式对元素进行变换，再将变换后的列表合并成一个新列表
        println(test1.flatMap { it.toList() })
```
输出
```kotlin
[a, a, b, b, b, c]
```
## sortedBy

sortedBy 作用就是排序   `有时间看下源码`

```kotlin
        val test2 = listOf(3, 2, 4, 6, 7, 1)
        //排序 低-->高
        println(test2.sortedBy { it })
        //排序 高-->低
        println(test2.sortedByDescending { it })
```
输出
```kotlin
[1, 2, 3, 4, 6, 7]
[7, 6, 4, 3, 2, 1]
```
## take & slice

```kotlin
        val test3 = listOf(3, 2, 4, 6, 7, 1)
        // 获取前3个元素,形成新的集合返回
        println(test3.take(3))
        // 获取指定index，拿到元素，重新组成集合返回
        println(test3.slice(IntRange(2, 4)))
```

## reduce

reduce的作用：将一个集合的所有元素通过传入的操作函数实现数据集合的累积操作效果。
```kotlin
        val test4 = listOf("a", "ab", "b", "bc")
        // reduce函数将一个集合的所有元素通过传入的操作函数实现数据集合的累积操作效果。
        println(test4.reduce { acc, name -> "$acc$name" })
```

输出

```kotlin
aabbbc
```

## 惰性序列操作

一些集合函数进行链式调用的时候，每个函数都将调用结果保存为一个新的临时列表，因此大量的链式操作会产生大量的中间变量，会造成性能问题，为了解决性能问题，可以把链式操作改成序列


`调用扩展函数asSequence把任意集合转换成序列，调用toList做反向翻转`

```kotlin
        // 函数的链式调用
        println("集合调用 展示age大于10的name ${
            testList.filter { it.age > 10 }
                .map { it.name }}")


        //函数的序列操作
        println("集合调用 展示age大于10的name1 ${
            testList.asSequence()
            .filter { it.age > 10 }
            .map { it.name }.toList()}")
```

```kotlin
                       //中间操作            //末端操作
testList.asSequence(). filter {..}.map {..}.toList() 
```
通过序列方式可以避免大量的中间集合，从而提高了性能

## 延时计算

```kotlin
fun lateAdd(a: Int, b: Int): ()->Int {
    fun add(): Int {
        return a + b
    }
    return ::add
}

fun main() {

    val add = lateAdd(1,2)

    println(add.invoke())
}
```
在lateAdd定义了一个局部函数，最后返回嘎函数的引用，对结果使用invoke,达到延迟使用

# fold

fold 作用：你可以设置一个初始值，然后遍历每一个元素然后放入lambda中操作后返回
```kotlin
val test5 = listOf("ab", "abc", "bd", "bcs")

        println(
            "fold 的使用==${
                test5.fold("初始值") { acc, s ->
                    acc + s
                }
            }"
        )
```
输出
```koltin
fold 的使用==初始值ababcbdbcs
```





