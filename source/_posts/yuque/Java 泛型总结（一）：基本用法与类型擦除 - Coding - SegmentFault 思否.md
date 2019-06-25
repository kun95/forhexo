
---

title: Java 泛型总结（一）：基本用法与类型擦除 - Coding - SegmentFault 思否

date: 2019-01-22 12:56:27 +0800

tags: []

---
<a name="y1fsyh"></a>
## [](#y1fsyh)简介
Java 在 1.5 引入了泛型机制，泛型本质是参数化类型，也就是说变量的类型是一个参数，在使用时再指定为具体类型。泛型可以用于类、接口、方法，通过使用泛型可以使代码更简单、安全。然而 Java 中的泛型使用了类型擦除，所以只是伪泛型。这篇文章对泛型的使用以及存在的问题做个总结，主要参考自 《Java 编程思想》。

这个系列的另外两篇文章：

- [Java 泛型总结（二）：泛型与数组](https://segmentfault.com/a/1190000005179147)

- [Java 泛型总结（三）：通配符的使用](https://segmentfault.com/a/1190000005337789)


<a name="rp66wu"></a>
## [](#rp66wu)基本用法
<a name="3kpkow"></a>
### [](#3kpkow)泛型类

如果有一个类 `Holder` 用于包装一个变量，这个变量的类型可能是任意的，怎么编写 `Holder` 呢？在没有泛型之前可以这样：
```java
public class Holder1 {
    private Object a;

    public Holder1(Object a) {
        this.a = a;
    }

    public void set(Object a) {
        this.a = a;
    }
    public Object get(){
        return a;
    }

    public static void main(String[] args) {
        Holder1 holder1 = new Holder1("not Generic");
        String s = (String) holder1.get();
        holder1.set(1);
        Integer x = (Integer) holder1.get();
    }
}j
```

在 `Holder1` 中，有一个用 `Object` 引用的变量。因为任何类型都可以向上转型为 `Object`，所以这个 `Holder` 可以接受任何类型。在取出的时候 `Holder` 只知道它保存的是一个 `Object` 对象，所以要强制转换为对应的类型。在 `main` 方法中， `holder1` 先是保存了一个字符串，也就是 `String` 对象，接着又变为保存一个 `Integer` 对象(参数 `1` 会自动装箱)。从 `Holder` 中取出变量时强制转换已经比较麻烦，这里还要记住不同的类型，要是转错了就会出现运行时异常。

下面看看 `Holder` 的泛型版本：
```java
public class Holder2<T> {

    private T a;
    public Holder2(T a) {
        this.a = a;
    }

    public T get() {
        return a;
    }

    public void set(T a) {
        this.a = a;
    }

    public static void main(String[] args) {
        Holder2<String> holder2 = new Holder2<>("Generic");
        String s = holder2.get();

        holder2.set("test");
        holder2.set(1); // 编译不通过

    }
}
```

在 `Holder2` 中， 变量 `a` 是一个参数化类型 `T`，T`` 只是一个标识，用其它字母也是可以的。创建 `Holder2` 对象的时候，在尖括号中传入了参数 `T` 的类型，那么在这个对象中，所有出现 `T` 的地方相当于都用 `String` 替换了。现在的 `get` 的取出来的不是 `Object` ，而是 `String` 对象，因此不需要类型转换。另外，当调用 `set` 时，只能传入 `String` 类型，否则编译无法通过。这就保证了 `holder2` 中的类型安全，避免由于不小心传入错误的类型。

通过上面的例子可以看出泛使得代码更简便、安全。引入泛型之后，Java 库的一些类，比如常用的容器类也被改写为支持泛型，我们使用的时候都会传入参数类型，如：`ArrayList<Integer> list = ArrayList<>();`。

<a name="1fk2uo"></a>
### [](#1fk2uo)泛型方法
泛型不仅可以针对类，还可以单独使某个方法是泛型的，举个例子：
```java
public class GenericMethod {
    // 如果在参数中使用了泛型，那么在函数的签名上要用尖括号表现出泛型
    public <K,V> void f(K k,V v) {
        System.out.println(k.getClass().getSimpleName());
        System.out.println(v.getClass().getSimpleName());
    }

    public static void main(String[] args) {
        GenericMethod gm = new GenericMethod();
        gm.f(new Integer(0),new String("generic"));
    }
}

代码输出：
    Integer
    String
```

`GenericMethod` 类本身不是泛型的，创建它的对象的时候不需要传入泛型参数，但是它的方法 `f` 是泛型方法。在返回类型之前是它的参数标识 `<K,V>`，注意这里有两个泛型参数，所以泛型参数可以有多个。

调用泛型方法时可以不显式传入泛型参数，上面的调用就没有。这是因为编译器会使用参数类型推断，根据传入的实参的类型 (这里是 `integer` 和 `String`) 推断出 `K` 和 `V` 的类型。

<a name="ffqotm"></a>
## [](#ffqotm)类型擦除
<a name="e4r1ku"></a>
### [](#e4r1ku)什么是类型擦除
Java 的泛型使用了类型擦除机制，这个引来了很大的争议，以至于 Java 的泛型功能受到限制，只能说是”伪泛型“。什么叫类型擦除呢？简单的说就是，类型参数只存在于编译期，在运行时，Java 的虚拟机 ( JVM ) 并不知道泛型的存在。先看个例子：
```java
public class ErasedTypeEquivalence {
    public static void main(String[] args) {
        Class c1 = new ArrayList<String>().getClass();
        Class c2 = new ArrayList<Integer>().getClass();
        System.out.println(c1 == c2);
    }
}
```

上面的代码有两个不同的 `ArrayList`：`ArrayList<Integer>` 和 `ArrayList<String>`。在我们看来它们的参数化类型不同，一个保存整性，一个保存字符串。但是通过比较它们的 `Class` 对象，上面的代码输出是 `true`。这说明在 JVM 看来它们是同一个类。而在 C++、C# 这些支持真泛型的语言中，它们就是不同的类。

泛型参数会擦除到它的第一个边界，比如说上面的 `Holder2` 类，参数类型是一个单独的 `T`，那么就擦除到 `Object`,相当于所有出现 `T` 的地方都用 `Object` 替换。所以在 JVM 看来，保存的变量 `a` 还是 `Object` 类型。之所以取出来自动就是我们传入的参数类型，这是因为编译器在编译生成的字节码文件中插入了类型转换的代码，不需要我们手动转型了。如果参数类型有边界那么就擦除到它的第一个边界，这个下一节再说。<br />简而言之：在代码层面，虽有泛型的存在，但是在运行时候的jvm层面却感受不到泛型的存在

<a name="gkovvz"></a>
### [](#gkovvz)擦除带来的问题
擦除会出现一些问题，下面是一个例子：
```java
class HasF {
    public void f() {
        System.out.println("HasF.f()");
    }
}
public class Manipulator<T> {
    private T obj;

    public Manipulator(T obj) {
        this.obj = obj;
    }

    public void manipulate() {
        obj.f(); 
    }

    public static void main(String[] args) {
        HasF hasF  = new HasF();
        Manipulator<HasF> manipulator = new Manipulator<>(hasF);
        manipulator.manipulate();
    }
}
```

上面的 `Manipulator` 是一个泛型类，内部用一个泛型化的变量 `obj`，在 `manipulate` 方法中，调用了 `obj` 的方法 `f()`，但是这行代码无法编译。因为类型擦除，编译器不确定 `obj` 是否有 `f()` 方法。解决这个问题的方法是给 `T` 一个边界:

```java
class Manipulator2<T extends HasF> {
    private T obj;
    public Manipulator2(T x) { obj = x; }
    public void manipulate() { obj.f(); }
}
```

现在 `T` 的类型是 `<T extends HasF>`，这表示 T`` 必须是 HasF`` 或者 HasF`` 的导出类型。这样，调用 `f()` 方法才安全。`HasF` 就是 `T` 的边界，因此通过类型擦除后，所有出现 `T` 的<br />地方都用 `HasF` 替换。这样编译器就知道 `obj` 是有方法 `f()` 的。

但是这样就抵消了泛型带来的好处，上面的类完全可以改成这样：
```java
class Manipulator3 {
    private HasF obj;
    public Manipulator3(HasF x) { obj = x; }
    public void manipulate() { obj.f(); }
}
```

所以泛型只有在比较复杂的类中才体现出作用。但是像 `<T extends HasF>` 这种形式的东西不是完全没有意义的。如果类中有一个返回 `T` 类型的方法，泛型就有用了，因为这样会返回准确类型。比如下面的例子：

```java
class ReturnGenericType<T extends HasF> {
    private T obj;
    public ReturnGenericType(T x) { obj = x; }
    public T get() { return obj; }
}
```

这里的 `get()` 方法返回的是泛型参数的准确类型，而不是 `HasF`。

<a name="i1rmga"></a>
### [](#i1rmga)类型擦除的补偿
类型擦除导致泛型丧失了一些功能，任何在运行期需要知道确切类型的代码都无法工作。比如下面的例子：
```java
public class Erased<T> {
    private final int SIZE = 100;
    public static void f(Object arg) {
        if(arg instanceof T) {} 
        T var = new T(); 
        T[] array = new T[SIZE]; 
        T[] array = (T)new Object[SIZE]; 
    }
}
```

通过 `new T()` 创建对象是不行的，一是由于类型擦除，二是由于编译器不知道 `T` 是否有默认的构造器。一种解决的办法是传递一个工厂对象并且通过它创建新的实例。
```java
interface FactoryI<T> {
    T create();
}
class Foo2<T> {
    private T x;
    public <F extends FactoryI<T>> Foo2(F factory) {
        x = factory.create();
    }
}
class IntegerFactory implements FactoryI<Integer> {
    public Integer create() {
        return new Integer(0);
    }
}
class Widget {
    public static class Factory implements FactoryI<Widget> {
        public Widget create() {
            return new Widget();
        }
    }
}
public class FactoryConstraint {
    public static void main(String[] args) {
        new Foo2<Integer>(new IntegerFactory());
        new Foo2<Widget>(new Widget.Factory());
    }
}
```

另一种解决的方法是利用模板设计模式：
```java
abstract class GenericWithCreate<T> {
    final T element;
    GenericWithCreate() { element = create(); }
    abstract T create();
}
class X {}
class Creator extends GenericWithCreate<X> {
    X create() { return new X(); }
    void f() {
        System.out.println(element.getClass().getSimpleName());
    }
}
public class CreatorGeneric {
    public static void main(String[] args) {
        Creator c = new Creator();
        c.f();
    }
}
```

具体类型的创建放到了子类继承父类时，在 `create` 方法中创建实际的类型并返回。

<a name="elwdex"></a>
## [](#elwdex)总结

本文介绍了 Java 泛型的使用，以及类型擦除相关的问题。一般情况下泛型的使用比较简单，但是某些情况下，尤其是自己编写使用泛型的类或者方法时要注意类型擦除的问题。接下来会介绍数组与泛型的关系以及通配符的使用，有兴趣的读者可进入下一篇：[Java 泛型总结（二）：泛型与数组](https://segmentfault.com/a/1190000005179147)。

**参考**

- Java 编程思想


**_如果我的文章对您有帮助，不妨点个赞支持一下(____)_**

