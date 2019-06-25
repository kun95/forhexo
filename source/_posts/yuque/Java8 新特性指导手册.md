
---

title: Java8 新特性指导手册

date: 2019-01-22 11:30:52 +0800

tags: []

---
> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [https://juejin.im/post/5c3d7c8a51882525dd591ac7](https://juejin.im/post/5c3d7c8a51882525dd591ac7)


> 本教程翻译整理自 [github.com/winterbe/ja…](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fwinterbe%2Fjava8-tutorial)


<a name="cfeff30d"></a>
## 目录：

- [一、接口内允许添加默认实现的方法](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#%E6%8E%A5%E5%8F%A3%E5%86%85%E5%85%81%E8%AE%B8%E6%B7%BB%E5%8A%A0%E9%BB%98%E8%AE%A4%E5%AE%9E%E7%8E%B0%E7%9A%84%E6%96%B9%E6%B3%95)
- [二、Lambda 表达式](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#Lambda-%E8%A1%A8%E8%BE%BE%E5%BC%8F)
- [三、函数式接口 Functional Interface](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#%E5%87%BD%E6%95%B0%E5%BC%8F%E6%8E%A5%E5%8F%A3-Functional-Interface)
- [四、便捷的引用类的构造器及方法](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#%E4%BE%BF%E6%8D%B7%E7%9A%84%E5%BC%95%E7%94%A8%E7%B1%BB%E7%9A%84%E6%9E%84%E9%80%A0%E5%99%A8%E5%8F%8A%E6%96%B9%E6%B3%95)
- [五、Lambda 访问外部变量及接口默认方法](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#Lambda-%E8%AE%BF%E9%97%AE%E5%A4%96%E9%83%A8%E5%8F%98%E9%87%8F%E5%8F%8A%E6%8E%A5%E5%8F%A3%E9%BB%98%E8%AE%A4%E6%96%B9%E6%B3%95)
  - [5.1 访问局部变量](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#%E8%AE%BF%E9%97%AE%E5%B1%80%E9%83%A8%E5%8F%98%E9%87%8F)
  - [5.2 访问成员变量和静态变量](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#%E8%AE%BF%E9%97%AE%E6%88%90%E5%91%98%E5%8F%98%E9%87%8F%E5%92%8C%E9%9D%99%E6%80%81%E5%8F%98%E9%87%8F)
  - [5.3 访问接口的默认方法](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#%E8%AE%BF%E9%97%AE%E6%8E%A5%E5%8F%A3%E7%9A%84%E9%BB%98%E8%AE%A4%E6%96%B9%E6%B3%95)
- [六、内置的函数式接口](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#%E5%86%85%E7%BD%AE%E7%9A%84%E5%87%BD%E6%95%B0%E5%BC%8F%E6%8E%A5%E5%8F%A3)
  - [6.1 Predicate 断言](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#Predicate-%E6%96%AD%E8%A8%80)
  - [6.2 Function](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#Function)
  - [6.3 Supplier 生产者](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#Supplier-%E7%94%9F%E4%BA%A7%E8%80%85)
  - [6.4 Consumer 消费者](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#Consumer-%E6%B6%88%E8%B4%B9%E8%80%85)
  - [6.5 Comparator](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#Comparator)
- [七、Optional](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#Optional)
- [八、Streams 流](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#Stream-%E6%B5%81)
  - [8.1 Filter 过滤](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#Filter-%E8%BF%87%E6%BB%A4)
  - [8.2 Sorted 排序](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#Sorted-%E6%8E%92%E5%BA%8F)
  - [8.3 Map 转换](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#Map-%E8%BD%AC%E6%8D%A2)
  - [8.4 Match 匹配](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#Match-%E5%8C%B9%E9%85%8D)
  - [8.5 Count 计数](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#Count-%E8%AE%A1%E6%95%B0)
  - [8.6 Reduce](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#Reduce)
- [九、Parallel Streams 并行流](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#Parallel-Streams-%E5%B9%B6%E8%A1%8C%E6%B5%81)
  - [9.1 顺序流排序](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#%E9%A1%BA%E5%BA%8F%E6%B5%81%E6%8E%92%E5%BA%8F)
  - [9.2 并行流排序](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#%E5%B9%B6%E8%A1%8C%E6%B5%81%E6%8E%92%E5%BA%8F)
- [十、Map 集合](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#Map-%E9%9B%86%E5%90%88)
- [十一、新的日期 API](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#%E6%96%B0%E7%9A%84%E6%97%A5%E6%9C%9F-API)
  - [11.1 Clock](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#Clock)
  - [11.2 Timezones 时区](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#Timezones-%E6%97%B6%E5%8C%BA)
  - [11.3 LocalTime](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#LocalTime)
  - [11.4 LocalDate](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#LocalDate)
  - [11.4 LocalDateTime](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#LocalDateTime)
- [十二、Annotations 注解](https://souche.yuque.com/kun95/ynl730/ypoxo3/edit#Annotations-%E6%B3%A8%E8%A7%A3)

也希望学完本系列教程的小伙伴能够熟练掌握和应用 Java8 的各种特性，使其成为在工作中的一门利器。废话不多说，让我们一起开启 Java8 新特性之旅吧！

---


★★★ **如果此教程有帮助到你, 去小哈的 GitHub 帮忙 **Star** 一下吧, 谢谢啦！** [传送门](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fweiwosuoai%2Fjava8_guide) ★★★

---


<a name="90c0f18e"></a>
## 接口内允许添加默认实现的方法

Java 8 允许我们通过 `default` 关键字对接口中定义的抽象方法提供一个默认的实现。

请看下面示例代码：

```
// 定义一个公式接口
interface Formula {
    // 计算
    double calculate(int a);

    // 求平方根
    default double sqrt(int a) {
        return Math.sqrt(a);
    }
}
复制代码
```

在上面这个接口中，我们除了定义了一个抽象方法 `calculate`，还定义了一个带有默认实现的方法 `sqrt`。 我们在实现这个接口时，可以只需要实现 `calculate` 方法，默认方法 `sqrt` 可以直接调用即可，也就是说我们可以不必强制实现 `sqrt` 方法。

> 补充：通过 `default` 关键字这个新特性，可以非常方便地对之前的接口做拓展，而此接口的实现类不必做任何改动。


```
Formula formula = new Formula() {
    @Override
    public double calculate(int a) {
        return sqrt(a * 100);
    }
};

formula.calculate(100);     // 100.0
formula.sqrt(16);           // 4.0
复制代码
```

上面通过匿名对象实现了 `Formula` 接口。但是即使是这样，我们为了完成一个 `sqrt(a * 100)` 简单计算，就写了 6 行代码，很是冗余。

<a name="75bf3fb1"></a>
## Lambda 表达式

在学习 `Lambda` 表达式之前，我们先来看一段老版本的示例代码，其对一个含有字符串的集合进行排序：

```
List<String> names = Arrays.asList("peter", "anna", "mike", "xenia");

Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return b.compareTo(a);
    }
});
复制代码
```

`Collections` 工具类提供了静态方法 `sort` 方法，入参是一个 `List` 集合，和一个 `Comparator` 比较器，以便对给定的 `List` 集合进行 排序。上面的示例代码创建了一个匿名内部类作为入参，这种类似的操作在我们日常的工作中随处可见。

Java 8 中不再推荐这种写法，而是推荐使用 Lambda 表达：

```
Collections.sort(names, (String a, String b) -> {
    return b.compareTo(a);
});
复制代码
```

正如你看到的，上面这段代码变得简短很多而且易于阅读。但是我们还可以再精炼一点：

```
Collections.sort(names, (String a, String b) -> b.compareTo(a));
复制代码
```

对于只包含一行方法的代码块，我们可以省略大括号，直接 `return` 关键代码即可。追求极致，我们还可以让它再短点：

```
names.sort((a, b) -> b.compareTo(a));
复制代码
```

`List` 集合现在已经添加了 `sort` 方法。而且 Java 编译器能够根据**类型推断机制**判断出参数类型，这样，你连入参的类型都可以省略啦，怎么样，是不是感觉很强大呢！

<a name="160243f4"></a>
## 函数式接口 Functional Interface

抛出一个疑问：在我们书写一段 Lambda 表达式后（比如上一章节中匿名内部类的 Lambda 表达式缩写形式），Java 编译器是如何进行类型推断的，它又是怎么知道重写的哪个方法的？

需要说明的是，不是每个接口都可以缩写成 Lambda 表达式。只有那些函数式接口（Functional Interface）才能缩写成 Lambda 表示式。

那么什么是函数式接口（Functional Interface）呢？

所谓函数式接口（Functional Interface）就是只包含一个抽象方法的声明。针对该接口类型的所有 Lambda 表达式都会与这个抽象方法匹配。

> 注意：你可能会有疑问，Java 8 中不是允许通过 defualt 关键字来为接口添加默认方法吗？那它算不算抽象方法呢？答案是：不算。因此，你可以毫无顾忌的添加默认方法，它并不违反函数式接口（Functional Interface）的定义。


总结一下：只要接口中仅仅包含一个抽象方法，我们就可以将其改写为 Lambda 表达式。为了保证一个接口明确的被定义为一个函数式接口（Functional Interface），我们需要为该接口添加注解：`@FunctionalInterface`。这样，一旦你添加了第二个抽象方法，编译器会立刻抛出错误提示。

示例代码：

```
@FunctionalInterface
interface Converter<F, T> {
    T convert(F from);
}
复制代码
```

示例代码 2：

```
Converter<String, Integer> converter = (from) -> Integer.valueOf(from);
Integer converted = converter.convert("123");
System.out.println(converted);    // 123
复制代码
```

> 注意：上面的示例代码，即使去掉 `@FunctionalInterface` 也是好使的，它仅仅是一种约束而已。


<a name="62b92c2b"></a>
## 便捷的引用类的构造器及方法

小伙伴们，还记得上一个章节这段示例代码么：

```
@FunctionalInterface
interface Converter<F, T> {
    T convert(F from);
}
复制代码
```

```
Converter<String, Integer> converter = (from) -> Integer.valueOf(from);
Integer converted = converter.convert("123");
System.out.println(converted);    // 123
复制代码
```

上面这段代码，通过 Java 8 的新特性，进一步简化上面的代码：

```
Converter<String, Integer> converter = Integer::valueOf;
Integer converted = converter.convert("123");
System.out.println(converted);   // 123
复制代码
```

Java 8 中允许你通过 `::` 关键字来引用类的方法或构造器。上面的代码简单的示例了如何引用静态方法，当然，除了静态方法，我们还可以引用普通方法：

```
class Something {
    String startsWith(String s) {
        return String.valueOf(s.charAt(0));
    }
}
复制代码
```

```
Something something = new Something();
Converter<String, String> converter = something::startsWith;
String converted = converter.convert("Java");
System.out.println(converted);    // "J"
复制代码
```

接下来，我们再来看看如何通过 `::` 关键字来引用类的构造器。首先，我们先来定义一个示例类，在类中声明两个构造器：

```
class Person {
    String firstName;
    String lastName;

    Person() {}

    Person(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
}
复制代码
```

然后，我们再定义一个工厂接口，用来生成 `Person` 类：

```
// Person 工厂
interface PersonFactory<P extends Person> {
    P create(String firstName, String lastName);
}
复制代码
```

我们可以通过 `::` 关键字来引用 `Person` 类的构造器，来代替手动去实现这个工厂接口：

```
// 直接引用 Person 构造器
PersonFactory<Person> personFactory = Person::new;
Person person = personFactory.create("Peter", "Parker");
复制代码
```

`Person::new` 这段代码，能够直接引用 `Person` 类的构造器。然后 Java 编译器能够根据上下文选中正确的构造器去实现 `PersonFactory.create` 方法。

<a name="c81218bf"></a>
## Lambda 访问外部变量及接口默认方法

在本章节中，我们将会讨论如何在 lambda 表达式中访问外部变量（包括：局部变量，成员变量，静态变量，接口的默认方法.），它与匿名内部类访问外部变量很相似。

<a name="c2e51379"></a>
### 访问局部变量

在 Lambda 表达式中，我们可以访问外部的 `final` 类型变量，如下面的示例代码：

```
// 转换器
@FunctionalInterface
interface Converter<F, T> {
    T convert(F from);
}
复制代码
```

```
final int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);

stringConverter.convert(2);     // 3
复制代码
```

与匿名内部类不同的是，我们不必显式声明 `num` 变量为 `final` 类型，下面这段代码同样有效：

```
int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);

stringConverter.convert(2);     // 3
复制代码
```

但是 `num` 变量必须为隐式的 `final` 类型，何为隐式的 `final` 呢？就是说到编译期为止，`num` 对象是不能被改变的，如下面这段代码，就不能被编译通过：

```
int num = 1;
Converter<Integer, String> stringConverter =
        (from) -> String.valueOf(from + num);
num = 3;
复制代码
```

在 lambda 表达式内部改变 `num` 值同样编译不通过，需要注意, 比如下面的示例代码：

```
int num = 1;
Converter<Integer, String> converter = (from) -> {
	String value = String.valueOf(from + num);
	num = 3;
	return value;
};
复制代码
```

<a name="f0b07ba6"></a>
### 访问成员变量和静态变量

上一章节中，了解了如何在 Lambda 表达式中访问局部变量。与局部变量相比，在 Lambda 表达式中对成员变量和静态变量拥有读写权限：

```
@FunctionalInterface
    interface Converter<F, T> {
        T convert(F from);
    }
复制代码
```

```
class Lambda4 {
        // 静态变量
        static int outerStaticNum;
        // 成员变量
        int outerNum;

        void testScopes() {
            Converter<Integer, String> stringConverter1 = (from) -> {
                // 对成员变量赋值
                outerNum = 23;
                return String.valueOf(from);
            };

            Converter<Integer, String> stringConverter2 = (from) -> {
                // 对静态变量赋值
                outerStaticNum = 72;
                return String.valueOf(from);
            };
        }
    }
复制代码
```

<a name="e9841639"></a>
### 访问接口的默认方法

还记得第一章节中定义的那个 `Formula` (公式) 接口吗？

```
@FunctionalInterface
interface Formula {
	// 计算
	double calculate(int a);

	// 求平方根
	default double sqrt(int a) {
		return Math.sqrt(a);
	}
}
复制代码
```

当时，我们在接口中定义了一个带有默认实现的 `sqrt` 求平方根方法，在匿名内部类中我们可以很方便的访问此方法：

```
Formula formula = new Formula() {
	@Override
	public double calculate(int a) {
		return sqrt(a * 100);
	}
};
复制代码
```

但是在 lambda 表达式中可不行：

```
Formula formula = (a) -> sqrt(a * 100);
复制代码
```

带有默认实现的接口方法，是**不能**在 lambda 表达式中访问的，上面这段代码将无法被编译通过。

<a name="f5de65ac"></a>
## 内置的函数式接口

JDK 1.8 API 包含了很多内置的函数式接口。其中就包括我们在老版本中经常见到的 Comparator 和 Runnable，Java 8 为他们都添加了 [FunctionalInterface ]()注解，以用来支持 Lambda 表达式。

值得一提的是，除了 Comparator 和 Runnable 外，还有一些新的函数式接口，它们很多都借鉴于知名的 [Google Guava](https://link.juejin.im/?target=https%3A%2F%2Fgithub.com%2Fgoogle%2Fguava) 库。

对于它们，即使你已经非常熟悉了，还是最好了解一下的：

<a name="cca10afe"></a>
### Predicate 断言

`Predicate` 是一个可以指定入参类型，并返回 boolean 值的函数式接口。它内部提供了一些带有默认实现的方法，可以 被用来组合一个复杂的逻辑判断（`and`, `or`, `negate`）：

```
Predicate<String> predicate = (s) -> s.length() > 0;

predicate.test("foo");              // true
predicate.negate().test("foo");     // false

Predicate<Boolean> nonNull = Objects::nonNull;
Predicate<Boolean> isNull = Objects::isNull;

Predicate<String> isEmpty = String::isEmpty;
Predicate<String> isNotEmpty = isEmpty.negate();
复制代码
```

<a name="Function"></a>
### Function

`Function` 函数式接口的作用是，我们可以为其提供一个原料，他给生产一个最终的产品。通过它提供的默认方法，组合, 链行处理 (`compose`, `andThen`)：

```
Function<String, Integer> toInteger = Integer::valueOf;
Function<String, String> backToString = toInteger.andThen(String::valueOf);

backToString.apply("123");     // "123"
复制代码
```

<a name="8dda89e0"></a>
### Supplier 生产者

`Supplier` 与 `Function` 不同，它不接受入参，直接为我们生产一个指定的结果，有点像生产者模式：

```
class Person {
    String firstName;
    String lastName;

    Person() {}

    Person(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
}
复制代码
```

```
Supplier<Person> personSupplier = Person::new;
personSupplier.get();   // new Person
复制代码
```

<a name="37c62d93"></a>
### Consumer 消费者

对于 `Consumer`，我们需要提供入参，用来被消费，如下面这段示例代码：

```
class Person {
    String firstName;
    String lastName;

    Person() {}

    Person(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
}
复制代码
```

```
Consumer<Person> greeter = (p) -> System.out.println("Hello, " + p.firstName);
greeter.accept(new Person("Luke", "Skywalker"));
复制代码
```

<a name="Comparator"></a>
### Comparator

`Comparator` 在 Java 8 之前是使用比较普遍的。Java 8 中除了将其升级成了函数式接口，还为它拓展了一些默认方法：

```
Comparator<Person> comparator = (p1, p2) -> p1.firstName.compareTo(p2.firstName);

Person p1 = new Person("John", "Doe");
Person p2 = new Person("Alice", "Wonderland");

comparator.compare(p1, p2);             // > 0
comparator.reversed().compare(p1, p2);  // < 0
复制代码
```

<a name="Optional"></a>
## Optional

首先，`Optional` 它不是一个函数式接口，设计它的目的是为了防止空指针异常（`NullPointerException`），要知道在 Java 编程中， 空指针异常可是臭名昭著的。

让我们来快速了解一下 `Optional` 要如何使用！你可以将 `Optional` 看做是包装对象（可能是 `null`, 也有可能非 `null`）的容器。当你定义了 一个方法，这个方法返回的对象可能是空，也有可能非空的时候，你就可以考虑用 `Optional` 来包装它，这也是在 Java 8 被推荐使用的做法。

```
Optional<String> optional = Optional.of("bam");

optional.isPresent();           // true
optional.get();                 // "bam"
optional.orElse("fallback");    // "bam"

optional.ifPresent((s) -> System.out.println(s.charAt(0)));     // "b"
复制代码
```

<a name="6d24bd9f"></a>
## Stream 流

这一章节，我们开始步入学习 `Stream` 流。

_什么是 `Stream` 流？_

简单来说，我们可以使用 `java.util.Stream` 对一个包含一个或多个元素的集合做各种操作。这些操作可能是 _中间操作_ 亦或是 _终端操作_。 终端操作会返回一个结果，而中间操作会返回一个 `Stream` 流。

需要注意的是，你只能对实现了 `java.util.Collection` 接口的类做流的操作。

> `Map` 不支持 `Stream` 流。


`Stream` 流支持同步执行，也支持并发执行。

让我们开始步入学习的旅程吧！Go !

<a name="35f21c77"></a>
### Filter 过滤

首先，我们创建一个 `List` 集合：

```
List<String> stringCollection = new ArrayList<>();
stringCollection.add("ddd2");
stringCollection.add("aaa2");
stringCollection.add("bbb1");
stringCollection.add("aaa1");
stringCollection.add("bbb3");
stringCollection.add("ccc");
stringCollection.add("bbb2");
stringCollection.add("ddd1");
复制代码
```

`Filter` 的入参是一个 `Predicate`, 上面已经说到，`Predicate` 是一个断言的中间操作，它能够帮我们筛选出我们需要的集合元素。它的返参同样 是一个 `Stream` 流，我们可以通过 `foreach` 终端操作，来打印被筛选的元素：

```
stringCollection
    .stream()
    .filter((s) -> s.startsWith("a"))
    .forEach(System.out::println);

// "aaa2", "aaa1"
复制代码
```

> 注意：`foreach` 是一个终端操作，它的返参是 `void`, 我们无法对其再次进行流操作。


<a name="4ac3d996"></a>
### Sorted 排序

`Sorted` 同样是一个中间操作，它的返参是一个 `Stream` 流。另外，我们可以传入一个 `Comparator` 用来自定义排序，如果不传，则使用默认的排序规则。

```
stringCollection
    .stream()
    .sorted()
    .filter((s) -> s.startsWith("a"))
    .forEach(System.out::println);

// "aaa1", "aaa2"
复制代码
```

需要注意，`sorted` 不会对 `stringCollection` 做出任何改变，`stringCollection` 还是原有的那些个元素，且顺序不变：

```
System.out.println(stringCollection);
// ddd2, aaa2, bbb1, aaa1, bbb3, ccc, bbb2, ddd1
复制代码
```

<a name="78e2f144"></a>
### Map 转换

中间操作 `Map` 能够帮助我们将 `List` 中的每一个元素做功能处理。例如下面的示例，通过 `map` 我们将每一个 `string` 转成大写：

```
stringCollection
    .stream()
    .map(String::toUpperCase)
    .sorted((a, b) -> b.compareTo(a))
    .forEach(System.out::println);

// "DDD2", "DDD1", "CCC", "BBB3", "BBB2", "AAA2", "AAA1"
复制代码
```

另外，我们还可以做对象之间的转换，业务中比较常用的是将 `DO`（数据库对象） 转换成 `BO`（业务对象） 。

<a name="b1e9fab6"></a>
### Match 匹配

顾名思义，`match` 用来做匹配操作，它的返回值是一个 `boolean` 类型。通过 `match`, 我们可以方便的验证一个 `list` 中是否存在某个类型的元素。

```
// 验证 list 中 string 是否有以 a 开头的, 匹配到第一个，即返回 true
boolean anyStartsWithA =
    stringCollection
        .stream()
        .anyMatch((s) -> s.startsWith("a"));

System.out.println(anyStartsWithA);      // true

// 验证 list 中 string 是否都是以 a 开头的
boolean allStartsWithA =
    stringCollection
        .stream()
        .allMatch((s) -> s.startsWith("a"));

System.out.println(allStartsWithA);      // false

// 验证 list 中 string 是否都不是以 z 开头的,
boolean noneStartsWithZ =
    stringCollection
        .stream()
        .noneMatch((s) -> s.startsWith("z"));

System.out.println(noneStartsWithZ);      // true
复制代码
```

<a name="d46dce25"></a>
### Count 计数

`count` 是一个终端操作，它能够统计 `stream` 流中的元素总数，返回值是 `long` 类型。

```
// 先对 list 中字符串开头为 b 进行过滤，让后统计数量
long startsWithB =
    stringCollection
        .stream()
        .filter((s) -> s.startsWith("b"))
        .count();

System.out.println(startsWithB);    // 3
复制代码
```

<a name="Reduce"></a>
### Reduce

`Reduce` 中文翻译为：_减少、缩小_。通过入参的 `Function`，我们能够将 `list` 归约成一个值。它的返回类型是 `Optional` 类型。

```
Optional<String> reduced =
    stringCollection
        .stream()
        .sorted()
        .reduce((s1, s2) -> s1 + "#" + s2);

reduced.ifPresent(System.out::println);
// "aaa1#aaa2#bbb1#bbb2#bbb3#ccc#ddd1#ddd2"
复制代码
```

<a name="b2264d65"></a>
## Parallel-Streams 并行流

前面章节我们说过，`stream` 流是支持**顺序**和**并行**的。顺序流操作是单线程操作，而并行流是通过多线程来处理的，能够充分利用物理机 多核 CPU 的优势，同时处理速度更快。

首先，我们创建一个包含 1000000 UUID list 集合。

```
int max = 1000000;
List<String> values = new ArrayList<>(max);
for (int i = 0; i < max; i++) {
    UUID uuid = UUID.randomUUID();
    values.add(uuid.toString());
}
复制代码
```

分别通过顺序流和并行流，对这个 list 进行排序，测算耗时:

<a name="111628cc"></a>
### 顺序流排序

```
// 纳秒
long t0 = System.nanoTime();

long count = values.stream().sorted().count();
System.out.println(count);

long t1 = System.nanoTime();

// 纳秒转微秒
long millis = TimeUnit.NANOSECONDS.toMillis(t1 - t0);
System.out.println(String.format("顺序流排序耗时: %d ms", millis));

// 顺序流排序耗时: 899 ms

复制代码
```

<a name="eb499c12"></a>
### 并行流排序

```
// 纳秒
long t0 = System.nanoTime();

long count = values.parallelStream().sorted().count();
System.out.println(count);

long t1 = System.nanoTime();

// 纳秒转微秒
long millis = TimeUnit.NANOSECONDS.toMillis(t1 - t0);
System.out.println(String.format("并行流排序耗时: %d ms", millis));

// 并行流排序耗时: 472 ms
复制代码
```

正如你所见，同样的逻辑处理，通过并行流，我们的性能提升了近 **50%**。完成这一切，我们需要做的仅仅是将 `stream` 改成了 `parallelStream`。

<a name="8321d67b"></a>
## Map 集合

前面已经提到过 `Map` 是不支持 `Stream` 流的，因为 `Map` 接口并没有像 `Collection` 接口那样，定义了 `stream()` 方法。但是，我们可以对其 `key`, `values`, `entry` 使用 流操作，如 `map.keySet().stream()`, `map.values().stream()` 和 `map.entrySet().stream()`.

另外, JDK 8 中对 `map` 提供了一些其他新特性:

```
Map<Integer, String> map = new HashMap<>();

for (int i = 0; i < 10; i++) {
    // 与老版不同的是，putIfAbent() 方法在 put 之前，
    // 会判断 key 是否已经存在，存在则直接返回 value, 否则 put, 再返回 value
    map.putIfAbsent(i, "val" + i);
}

// forEach 可以很方便地对 map 进行遍历操作
map.forEach((key, value) -> System.out.println(value));
复制代码
```

除了上面的 `putIfAbsent()` 和 `forEach()` 外，我们还可以很方便地对某个 `key` 的值做相关操作：

```
// computeIfPresent(), 当 key 存在时，才会做相关处理
// 如下：对 key 为 3 的值，内部会先判断值是否存在，存在，则做 value + key 的拼接操作
map.computeIfPresent(3, (num, val) -> val + num);
map.get(3);             // val33

// 先判断 key 为 9 的元素是否存在，存在，则做删除操作
map.computeIfPresent(9, (num, val) -> null);
map.containsKey(9);     // false

// computeIfAbsent(), 当 key 不存在时，才会做相关处理
// 如下：先判断 key 为 23 的元素是否存在，不存在，则添加
map.computeIfAbsent(23, num -> "val" + num);
map.containsKey(23);    // true

// 先判断 key 为 3 的元素是否存在，存在，则不做任何处理
map.computeIfAbsent(3, num -> "bam");
map.get(3);             // val33
复制代码
```

关于删除操作，JDK 8 中提供了能够新的 `remove()` API:

```
map.remove(3, "val3");
map.get(3);             // val33

map.remove(3, "val33");
map.get(3);             // null
复制代码
```

如上代码，只有当给定的 `key` 和 `value` 完全匹配时，才会执行删除操作。

关于添加方法，JDK 8 中提供了带有默认值的 `getOrDefault()` 方法：

```
// 若 key 42 不存在，则返回 not found
map.getOrDefault(42, "not found");  // not found
复制代码
```

对于 `value` 的合并操作也变得更加简单：

```
// merge 方法，会先判断进行合并的 key 是否存在，不存在，则会添加元素
map.merge(9, "val9", (value, newValue) -> value.concat(newValue));
map.get(9);             // val9

// 若 key 的元素存在，则对 value 执行拼接操作
map.merge(9, "concat", (value, newValue) -> value.concat(newValue));
map.get(9);             // val9concat
复制代码
```

<a name="df70aa39"></a>
## 新的日期 API

Java 8 中在包 `java.time` 下添加了新的日期 API. 它和 [Joda-Time](https://link.juejin.im/?target=http%3A%2F%2Fwww.joda.org%2Fjoda-time%2F) 库相似，但又不完全相同。接下来，我会通过一些示例代码介绍一下新 API 中 最关键的特性：

<a name="Clock"></a>
### Clock

`Clock` 提供对当前日期和时间的访问。我们可以利用它来替代 `System.currentTimeMillis()` 方法。另外，通过 `clock.instant()` 能够获取一个 `instant` 实例， 此实例能够方便地转换成老版本中的 `java.util.Date` 对象。

```
Clock clock = Clock.systemDefaultZone();
long millis = clock.millis();

Instant instant = clock.instant();
Date legacyDate = Date.from(instant);   // 老版本 java.util.Date
复制代码
```

<a name="4791e622"></a>
### Timezones 时区

`ZoneId` 代表时区类。通过静态工厂方法方便地获取它，入参我们可以传入某个时区编码。另外，时区类还定义了一个偏移量，用来在当前时刻或某时间 与目标时区时间之间进行转换。

```
System.out.println(ZoneId.getAvailableZoneIds());
// prints all available timezone ids

ZoneId zone1 = ZoneId.of("Europe/Berlin");
ZoneId zone2 = ZoneId.of("Brazil/East");
System.out.println(zone1.getRules());
System.out.println(zone2.getRules());

// ZoneRules[currentStandardOffset=+01:00]
// ZoneRules[currentStandardOffset=-03:00]
复制代码
```

<a name="LocalTime"></a>
### LocalTime

`LocalTime` 表示一个没有指定时区的时间类，例如，`10 p.m`. 或者 `17：30:15`，下面示例代码中，将会使用上面创建的 时区对象创建两个 `LocalTime`。然后我们会比较两个时间，并计算它们之间的小时和分钟的不同。

```
LocalTime now1 = LocalTime.now(zone1);
LocalTime now2 = LocalTime.now(zone2);

System.out.println(now1.isBefore(now2));  // false

long hoursBetween = ChronoUnit.HOURS.between(now1, now2);
long minutesBetween = ChronoUnit.MINUTES.between(now1, now2);

System.out.println(hoursBetween);       // -3
System.out.println(minutesBetween);     // -239
复制代码
```

`LocalTime` 提供多个静态工厂方法，目的是为了简化对时间对象实例的创建和操作，包括对时间字符串进行解析的操作等。

```
LocalTime late = LocalTime.of(23, 59, 59);
System.out.println(late);       // 23:59:59

DateTimeFormatter germanFormatter =
    DateTimeFormatter
        .ofLocalizedTime(FormatStyle.SHORT)
        .withLocale(Locale.GERMAN);

LocalTime leetTime = LocalTime.parse("13:37", germanFormatter);
System.out.println(leetTime);   // 13:37
复制代码
```

<a name="LocalDate"></a>
## LocalDate

`LocalDate` 是一个日期对象，例如：`2014-03-11`。它和 `LocalTime` 一样是个 `final` 类型对象。下面的例子演示了如何通过加减日，月，年等来计算一个新的日期。

> `LocalDate`, `LocalTime`, 因为是 `final` 类型的对象，每一次操作都会返回一个新的时间对象。


```
LocalDate today = LocalDate.now();
// 今天加一天
LocalDate tomorrow = today.plus(1, ChronoUnit.DAYS);
// 明天减两天
LocalDate yesterday = tomorrow.minusDays(2);

// 2014 年七月的第四天
LocalDate independenceDay = LocalDate.of(2014, Month.JULY, 4);
DayOfWeek dayOfWeek = independenceDay.getDayOfWeek();
System.out.println(dayOfWeek);    // 星期五
复制代码
```

也可以直接解析日期字符串，生成 `LocalDate` 实例。（和 `LocalTime` 操作一样简单）

```
DateTimeFormatter germanFormatter =
    DateTimeFormatter
        .ofLocalizedDate(FormatStyle.MEDIUM)
        .withLocale(Locale.GERMAN);

LocalDate xmas = LocalDate.parse("24.12.2014", germanFormatter);
System.out.println(xmas);   // 2014-12-24

复制代码
```

<a name="LocalDateTime"></a>
### LocalDateTime

`LocalDateTime` 是一个**日期 - 时间**对象。你也可以将其看成是 `LocalDate` 和 `LocalTime` 的结合体。操作上，也大致相同。

> `LocalDateTime` 同样是一个 `final` 类型对象。


```
LocalDateTime sylvester = LocalDateTime.of(2014, Month.DECEMBER, 31, 23, 59, 59);

DayOfWeek dayOfWeek = sylvester.getDayOfWeek();
System.out.println(dayOfWeek);      // 星期三

Month month = sylvester.getMonth();
System.out.println(month);          // 十二月

// 获取改时间是该天中的第几分钟
long minuteOfDay = sylvester.getLong(ChronoField.MINUTE_OF_DAY);
System.out.println(minuteOfDay);    // 1439
复制代码
```

如果再加上的时区信息，`LocalDateTime` 还能够被转换成 `Instance` 实例。`Instance` 能够被转换成老版本中 `java.util.Date` 对象。

```
Instant instant = sylvester
        .atZone(ZoneId.systemDefault())
        .toInstant();

Date legacyDate = Date.from(instant);
System.out.println(legacyDate);     // Wed Dec 31 23:59:59 CET 2014
复制代码
```

格式化 `LocalDateTime` 对象就和格式化 LocalDate 或者 LocalTime 一样。除了使用预定义的格式以外，也可以自定义格式化输出。

```
DateTimeFormatter formatter =
    DateTimeFormatter
        .ofPattern("MMM dd, yyyy - HH:mm");

LocalDateTime parsed = LocalDateTime.parse("Nov 03, 2014 - 07:13", formatter);
String string = formatter.format(parsed);
System.out.println(string);     // Nov 03, 2014 - 07:13
复制代码
```

> 注意：和 `java.text.NumberFormat` 不同，新的 `DateTimeFormatter` 类是 `final` 类型的，同时也是线程安全的。更多细节请查看[这里](https://link.juejin.im/?target=http%3A%2F%2Fdownload.java.net%2Fjdk8%2Fdocs%2Fapi%2Fjava%2Ftime%2Fformat%2FDateTimeFormatter.html)


<a name="44fc421c"></a>
## Annotations 注解

在 Java 8 中，注解是可以重复的。让我通过下面的示例代码，来看看到底是咋回事。

首先，我们定义一个包装注解，里面包含了一个有着实际注解的数组：

```
@interface Hints {
    Hint[] value();
}

@Repeatable(Hints.class)
@interface Hint {
    String value();
}
复制代码
```

Java 8 中，通过 `@Repeatable`，允许我们对同一个类使用多重注解：

第一种形态：使用注解容器（老方法）

```
@Hints({@Hint("hint1"), @Hint("hint2")})
class Person {}
复制代码
```

第二种形态：使用可重复注解（新方法）

```
@Hint("hint1")
@Hint("hint2")
class Person {}
复制代码
```

使用第二种形态，Java 编译器能够在内部自动对 `@Hint` 进行设置。这对于需要通过反射来读取注解信息时，是非常重要的。

```
Hint hint = Person.class.getAnnotation(Hint.class);
System.out.println(hint);                   // null

Hints hints1 = Person.class.getAnnotation(Hints.class);
System.out.println(hints1.value().length);  // 2

Hint[] hints2 = Person.class.getAnnotationsByType(Hint.class);
System.out.println(hints2.length);          // 2
复制代码
```

尽管我们绝对不会在 `Person` 类上声明 `@Hints` 注解，但是它的信息仍然是可以通过 `getAnnotation(Hints.class)` 来读取的。 并且，`getAnnotationsByType` 方法会更方便，因为它赋予了所有 `@Hints` 注解标注的方法直接的访问权限。

```
@Target({ElementType.TYPE_PARAMETER, ElementType.TYPE_USE})
@interface MyAnnotation {}
复制代码
```

<a name="433531fd"></a>
## 结语

Java 8 新特性的编程指南到此就告一段落了。当然，还有很多内容需要进一步研究和说明。这就需要靠读者您来对 JDK 8 进一步探究了， 例如：`Arrays.parallelSort`, `StampedLock` 和 `CompletableFuture` 等等，我这里也仅是起到抛砖引玉的作用而已。

最后，我希望这个教程能够对您有所帮助，也希望您阅读愉快。

<a name="b8c89356"></a>
## 小哈的微信公众号



