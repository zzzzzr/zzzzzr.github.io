---
title: JAVA8特性学习
layout: default
categories: Java学习
tags: [Lambda, Stream, Optional, DateTime]
date: 2022-01-01
---

# JAVA8特性学习

> 在网上看到其他人总结的JAVA8特性文档，这里做一个总结学习，用于后面自己回忆。

JAVA8的特性可以分为4部分：

1. Lambda表达式
2. 流式API
3. Optional类
4. 新版日期时间API

下面分别做总结

## **Lambda表达式**

> 链接：<https://lw900925.github.io/java/java8-lambda-expression.html>

**行为参数化**：行为参数化是指预先定义一个代码块但不去执行它，而是把它当做参数传递给另一个方法。

**Lambda表达式（lambda expression）**：lambda表达式是一个匿名函数，在Java 8中可以把Lambda表达式理解为匿名函数，它没有名称，但是有参数列表、函数主体、返回类型等。

**Lambda表达式的语法**：`(parameters) -> { statements; }` 。

**方法引用**：用来直接访问类或者实例的已经存在的方法或者构造方法，如果某个方法签名和接口恰好一致，就可以直接传入方法引用。

**方法引用的语法**：方法引用使用:: 分隔符，语法为A::B 。分隔符的前半部分表示引用类型，后面半部分表示引用的方法名称。例如：`Integer::compareTo` 表示引用类型为`Integer` ，引用名称为`compareTo` 的方法。

**函数式接口**：在Java 8中，把那些仅有一个抽象方法的接口称为函数式接口。如果一个接口被`@FunctionalInterface` 注解标注，表示这个接口被设计成函数式接口，只能有一个抽象方法，如果你添加多个抽象方法，编译时会提示`“Multiple non-overriding abstract methods found in interface XXX”` 之类的错误。

**函数式接口举例**：Runnble、Callable、Consumer、Function......

**Consumer接口**：`java.util.function.Consumer<T>` 定义了一个名叫`accept()` 的抽象方法，它接受泛型T的对象，没有返回（void） 。如果你需要访问类型T的对象，并对其执行某些操作，就可以使用这个接口。

**Function接口**：`java.util.function.Function<T, R>` 接口定义了一个叫作`apply()` 的方法，它接受一个泛型T的对象，并返回一个泛型R的对象。可以看出这个接口主要用于将一个对象转化成另一个类型的对象。

---

## **流式API**

> 链接：<https://lw900925.github.io/java/java8-stream-api.html>

StreamAPI：Stream API是对Java中集合操作的增强，可以利用它进行各种过滤、排序、分组、聚合等操作

流与集合的区别：遍历方式不同，遍历集合通常使用`for-each` 方式，这种方式称为外部迭代，而流使用内部迭代方式，也就是说它帮你把迭代的工作做了，你只需要给出一个函数来告诉它接下来要干什么

StreamAPI的并发能力：`Stream API`将迭代操作封装到了内部，它会自动的选择最优的迭代方式，并且使用并行方式处理时，将集合分成多段，每一段分别使用不同的线程处理，最后将处理结果合并输出，有点类似于`Fork-Join`操作。

使用流的限制：流只能遍历一次，遍历结束后，这个流就被关闭掉了。如果要重新遍历，可以从数据源（集合）中重新获取一个流。如果对一个流遍历两次，就会抛出`java.lang.IllegalStateException`异常

```java
List<String> list = Arrays.asList("A", "B", "C", "D");
Stream<String> stream = list.stream();
stream.forEach(System.out::println);
stream.forEach(System.out::println); // 这里会抛出java.lang.IllegalStateException异常，因为流已经被关闭
```

流的构成：流通常由三部分构成

1. 数据源：数据源一般用于流的获取，比如`users.stream()` 。
2. 中间处理：中间处理包括对流中元素的一系列处理，如：过滤`filter()` ，映射`map()` ，排序`sorted()` 。
3. 终端处理：终端处理会生成结果，结果可以是任何不是流值，如`List<String>` ；也可以不返回结果，如`stream.forEach(System.out::println)` 就是将结果打印到控制台中，并没有返回。

---

## **Optional类**

> 链接：<https://lw900925.github.io/java/java8-optional.html>

Optional对象通过泛型包装了一个内部对象，该Optional对象用于对内部对象进行判空判断。

`java.util.Optional<T>` 类 是一个封装了 `Optional` 值的容器对象，`Optional` 值可以为`null` ，如果值存在，调用`isPresent()` 方法返回true ，调用`get()` 方法可以获取值。

Optional对象的创建：`Optional` 类提供类三个方法用于实例化一个Optional 对象，它们分别为`empty()`  `of()`、`ofNullable()`，这三个方法都是静态方法，可以直接调用。

`empty`方法用于创建一个空`Optional`对象；`of`方法通过传入一个非空对象创建一个`Optional`对象；`ofNullable`方法通过传入一个可以为空的对象创建一个`Optional`对象。

获取Optional对象的值：

- `get()` 获取Optional内部的值，如果为null则抛出NPE

- `orElse()` 如果有值就返回，否则返回一个给定的值作为默认值；

- `orElseGet()` 与orElse() 方法作用类似，区别在于生成默认值的方式不同。该方法接受一个Supplier<? extends T> 函数式接口参数，用于生成默认值；

- `orElseThrow()` 与前面介绍的get() 方法类似，当值为null时调用这两个方法都会抛出NullPointerException 异常，区别在于该方法可以指定抛出的异常类型。

---

## **新版日期时间API**

> 链接：<https://lw900925.github.io/java/java8-newtime-api.html>

日期类、时间类、日期时间类：`LocalDate`  `LocalTime`  `LocalDateTime`

时间戳类：`Instant`

时间区间类：`Duration` （时间区间更细，最细以诺秒作为时间单位） `Period` （以日作为时间单位）

时间操作类：`TemporalAdjusters`

时间格式化类：`DateTimeFormatter` 将时间转换为对应字符串，或者将字符串转换为时间

时区类：`ZoneId` `ZonedDateTime` `ZoneOffset` `OffsetDateTime`
