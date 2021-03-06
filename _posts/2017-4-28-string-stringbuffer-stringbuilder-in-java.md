---
layout: post
title: "String, StringBuffer and StringBuilder in Java"
date:   2017-04-27
excerpt: "Java 中的 String，StringBuffer，StringBuilder 详解"
tags: [Java]
comments: true
---

<center><h2>Java 中的 String，StringBuffer，StringBuilder 详解</h2></center>

<!--more-->

| :---------------: | :-------: |
| <font color="red">String</font> | <font color="blue">字符串常量</font> |
| **StringBuffer**  | **字符串变量** |
| **StringBuilder** | **字符串变量** |

以上三者都是用于字符串处理，不过各有千秋，那么该如何选择呢？

首先，看了上面的表格，可能会对 **String** 有一些疑问。既然是常量那么就不可改变，那为什可以这样写：

```java
String demo = "1 + 1 = ";

demo += "2";

System.out.print(demo);	//	result: 1 + 1 = 2
```

如此一来，我们不是改变了字符串常量 `demo` 吗？

其实在 **JVM** 内部大概是这样运行的：

1. 创建对象 `demo`
2. 将 `"1 + 1 = "` 赋给 `demo`
3. 创建一个新 `demo`
4. 将 `"1 + 1 = 2"` 赋给新 `demo`
5. 回收旧 `demo`

而另外两者是字符串变量，是可改变的。所有操作都是对于一个对象而言，无需创建额外的对象。

说了这么多，现在很容易就得出一个关于执行速度的结论：**StringBuffer > String**，**StringBuilder > String**。

不过凡事都有例外：

```java
String demoString = "这" + "是" + "个" + "例" + "外";
StringBuffer demoStringBuilder = new StringBuilder("这").append("是").append("个").append("例").append("外");
```

如果拼接大量字符串，写一个计时函数。你会发现，`demoString` 对象生成速度惊人的快，**StringBuilder** 类型根本比不上。

事实上 **JVM** 又耍了个花招：

1. 创建对象 `demoString`
2. 将 `"这是个例外"` 赋给 `demoString`

如果你这么做的话，它就原形毕露了：

```java
String demoString1 = "这";
String demoString2 = "是";
String demoString3 = "个";
String demoString4 = "例";
String demoString5 = "外";
String demoString = demoString1 + demoString2 + demoString3 + demoString4 + demoString5;
```

接下来比较一下 **StringBuffer** 和 **StringBuilder**：

|   StringBuffer    |   线程安全的    |
| :---------------: | :--------: |
| **StringBuilder** | **线程非安全的** |

如果字符串被多个线程使用，**JVM** 会保证对 **StringBuffer** 的操作是安全的，却不会对 **StringBuilder** 承诺。

由此可见，它们的执行速度：**StringBuilder > StringBuffer > String**。

所以一般根据下面三种情况来进行选择：

1. 操作少量数据 —— **String**
2. 操作大量数据且在单线程下 —— **StringBuilder**
3. 操作大量数据且在多线程下 —— **StringBuffer**

如果想了解更多，接着往下看。

### StringBuilder

#### 符号 “+” 的本质

当对 **String** 类型字符串进行 `+` 操作时，实际上发生了这些：

1. 编译器发现有对 **String** 进行拼接的操作
2. 编译器将 `+` 操作替换为 **StringBuilder** 的 `append()` 操作
3. **JVM** 初始化新的 **StringBuilder** 来进行 `append()` 操作
4. 将 **StringBuilder** 变为 **String**

- `特别注意：前两步是发生在编译过程中的!`

所以说下面两者执行效果是一样的：

```java
String demoString1 = "1 + 1 = ";
String demoString2 = "2";

String demoString = demoString1 + demoString2;
String demo = new StringBuilder().append(demoString1).append(demoString2).toString();
```

虽然最终效果一样，不过中间过程略不相同，这也是 **StringBuilder** 执行速度快于 **String** 的原因。

可以用 `for` 循环做个实验：

```java
public class Test {
  public static void main(String[] args){
  	testString();
  	testStringBuilder();
  }
  
  public static void testString() {
    long start = System.currentTimeMillis();
    String result = "";
    
    for (int i = 0; i < 100000; i++) {
      result += i;
    }
    
    System.out.println(System.currentTimeMillis() - start);
  }
  
  public static void testStringBuilder() {
    long start = System.currentTimeMillis();
    StringBuilder result = new StringBuilder();
    
    for (int i = 0; i < 100000; i++) {
      builder.append(i);
    }
    
    System.out.println(System.currentTimeMillis() - start);
  }
}
```

从输出结果可以看到，**String** 慢很多。同样都是用的 `append()`，为什么会有如此大的差别？

只需要用 `java -c` 命令查看 **class** 文件。答案一目了然：在 `testString()` 中，每次循环都要重新初始化 **StringBuilder** 对象，所以拉低了效率。

#### 关于性能

**StringBuilder** 内部实际上维护了一个 `char[]` 类型的 `value`，用来保存通过 `append()` 添加的内容，通过 `new StringBuilder()` 初始化时，`char[]` 的默认长度为 16，如果 `append()` 第 17 个字符，会发生什么？

```java
void expandCapacity(int minimumCapacity) {
  int newCapacity = value.length * 2 + 2;
  
  if (newCapacity - minimumCapacity < 0) {
    newCapacity = minimumCapacity;
  }
  if (newCapacity < 0) {
    if (minimumCapacity < 0) {
      throw new OutOfMemoryError();
    }
    
    newCapacity = integer.MAX_VALUE;
  }
  
  value = Arrays.copyOf(value, newCapacity);
}
```

如果 `value` 的剩余容量，无法添加全部内容，则通过 `expandSize(int minimumCapacity)()` 对 `value` 进行扩容，其中 `minimumCapacity = 原 value 长度 + append 添加的内容长度`。

1. 扩大容量为原来的两倍 + 2，为什么要 + 2，而不是刚好两倍？
2. 如果扩容之后，还是无法添加全部内容，则将 minimumCapacity 作为最终的容量大小；
3. 利用 `System.arraycopy` 方法对原value数据进行复制；

在使用StringBuilder时，如果给定一个合适的初始值，可以避免由于char[]数组多次复制而导致的性能问题。

#### 关于内存

**StringBuilder** 内部进行扩容时，会新建一个大小为原来两倍 + 2 的 `char` 数组，并复制原 `char` 数组到新数组，导致内存的消耗，增加垃圾回收的压力。

**StringBuilder** 的 `toString()`，也会造成 `char` 数组的浪费：

```java
public String toString() {
  return new String(value, 0, count);
}
```

`String` 的构造方法中，会新建一个大小相等的 `char` 数组，并使用 `System.arraycopy()` 复制 **StringBuilder** 中 `char` 数组的数据到其中，这样 **StringBuilder**的 `char` 数组就白白浪费了。

**重用StringBuilder**

```java
public class RStringBuilder {
  private final StringBuilder stringBuilder;
  
  public StringBuilderHolder(int size) {
    stringBuilder = new StringBuilder(size);
  }
  
  public StringBuilder reset() {
    stringBuilder.setLength(0);
    
    return stringBuilder;
  }
}
```

通过 `stringBuilder.setLength(0)` 可以把 `char` 数组的内存区域设置为 `0`，这样 `char` 数组重复使用，为了避免并发访问，可以在 `ThreadLocal` 中使用 **RStringBuilder**：

```java
private static final ThreadLocal<RStringBuilder> stringBuilder = new ThreadLocal<RStringBuilder>() {
  @Override
  protected RStringBuilder initialValue() {
    return new RStringBuilder(256);
  }
};

StringBuilder demoStringBuilder = stringBuilder.get().reset();
```

不过这种方式也存在一个弊端，**StringBuilder** 实例的内存空间一直不会被垃圾回收，如果 `char` 数组在某次操作中被扩容到一个很大的值，可能之后很长一段时间都不会用到如此大的空间，就会造成内存的浪费。