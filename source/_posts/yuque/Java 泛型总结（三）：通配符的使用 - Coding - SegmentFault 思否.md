
---

title: Java 泛型总结（三）：通配符的使用 - Coding - SegmentFault 思否

date: 2019-01-22 12:57:08 +0800

tags: []

---
简介
--

前两篇文章介绍了泛型的基本用法、类型擦除以及泛型数组。在泛型的使用中，还有个重要的东西叫通配符，本文介绍通配符的使用。

这个系列的另外两篇文章：

*   [Java 泛型总结（一）：基本用法与类型擦除](https://segmentfault.com/a/1190000005179142)
*   [Java 泛型总结（二）：泛型与数组](https://segmentfault.com/a/1190000005179147)

数组的协变
-----

在了解通配符之前，先来了解一下数组。Java 中的数组是**协变**的，什么意思？看下面的例子：

    class Fruit {}
    class Apple extends Fruit {}
    class Jonathan extends Apple {}
    class Orange extends Fruit {}
    
    public class CovariantArrays {
        public static void main(String[] args) {       
            Fruit[] fruit = new Apple[10];
            fruit[0] = new Apple(); 
            fruit[1] = new Jonathan(); 
            
            try {
                
                fruit[0] = new Fruit(); 
            } catch(Exception e) { System.out.println(e); }
            try {
                
                fruit[0] = new Orange(); 
            } catch(Exception e) { System.out.println(e); }
            }
    } 

`main` 方法中的第一行，创建了一个 `Apple` 数组并把它赋给 `Fruit` 数组的引用。这是有意义的，`Apple` 是 `Fruit` 的子类，一个 `Apple` 对象也是一种 `Fruit` 对象，所以一个 `Apple` 数组也是一种 `Fruit` 的数组。这称作**数组的协变**，Java 把数组设计为协变的，对此是有争议的，有人认为这是一种缺陷。

尽管 `Apple[]` 可以 “向上转型” 为 `Fruit[]`，但数组元素的实际类型还是 `Apple`，我们只能向数组中放入 `Apple`或者 `Apple` 的子类。在上面的代码中，向数组中放入了 `Fruit` 对象和 `Orange` 对象。对于编译器来说，这是可以通过编译的，但是在运行时期，JVM 能够知道数组的实际类型是 `Apple[]`，所以当其它对象加入数组的时候就会抛出异常。

泛型设计的目的之一是要使这种运行时期的错误在编译期就能发现，看看用泛型容器类来代替数组会发生什么：

    
    ArrayList<Fruit> flist = new ArrayList<Apple>();
    

上面的代码根本就无法编译。当涉及到泛型时， 尽管 `Apple` 是 `Fruit` 的子类型，但是 `ArrayList<Apple>` 不是 `ArrayList<Fruit>` 的子类型，泛型不支持协变。

使用通配符
-----

从上面我们知道，`List<Number> list = ArrayList<Integer>` 这样的语句是无法通过编译的，尽管 `Integer` 是 `Number` 的子类型。那么如果我们确实需要建立这种 “向上转型” 的关系怎么办呢？这就需要通配符来发挥作用了。

### 上边界限定通配符

利用 `<? extends Fruit>` 形式的通配符，可以实现泛型的向上转型：

    public class GenericsAndCovariance {
        public static void main(String[] args) {
            
            List<? extends Fruit> flist = new ArrayList<Apple>();
            
            
            
            
            flist.add(null); 
            
            Fruit f = flist.get(0);
        }
    }

上面的例子中， `flist` 的类型是 `List<? extends Fruit>`，我们可以把它读作：一个类型的 List， 这个类型可以是继承了 `Fruit` 的某种类型。注意，**这并不是说这个 List 可以持有** `Fruit` **的任意类型**。通配符代表了一种特定的类型，它表示 “某种特定的类型，但是 `flist` 没有指定”。这样不太好理解，具体针对这个例子解释就是，`flist` 引用可以指向某个类型的 List，只要这个类型继承自 `Fruit`，可以是 `Fruit` 或者 `Apple`，比如例子中的 `new ArrayList<Apple>`，但是为了向上转型给 `flist`，`flist` 并不关心这个具体类型是什么。

如上所述，通配符 `List<? extends Fruit>` 表示某种特定类型 ( `Fruit` 或者其子类 ) 的 List，但是并不关心这个实际的类型到底是什么，反正是 `Fruit` 的子类型，`Fruit` 是它的上边界。那么对这样的一个 List 我们能做什么呢？其实如果我们不知道这个 List 到底持有什么类型，怎么可能安全的添加一个对象呢？在上面的代码中，向 `flist` 中添加任何对象，无论是 `Apple` 还是 `Orange` 甚至是 `Fruit` 对象，编译器都不允许，唯一可以添加的是 `null`。所以如果做了泛型的向上转型 (`List<? extends Fruit> flist = new ArrayList<Apple>()`)，那么我们也就失去了向这个 List 添加任何对象的能力，即使是 `Object` 也不行。

另一方面，如果调用某个返回 `Fruit` 的方法，这是安全的。因为我们知道，在这个 List 中，不管它实际的类型到底是什么，但肯定能转型为 `Fruit`，所以编译器允许返回 `Fruit`。

了解了通配符的作用和限制后，好像任何接受参数的方法我们都不能调用了。其实倒也不是，看下面的例子：

    public class CompilerIntelligence {
        public static void main(String[] args) {
            List<? extends Fruit> flist =
            Arrays.asList(new Apple());
            Apple a = (Apple)flist.get(0); 
            flist.contains(new Apple()); 
            flist.indexOf(new Apple()); 
            
            
    
        }
    }

在上面的例子中，`flist` 的类型是 `List<? extends Fruit>`，泛型参数使用了受限制的通配符，所以我们失去了向其中加入任何类型对象的例子，最后一行代码无法编译。

但是 `flist` 却可以调用 `contains` 和 `indexOf` 方法，它们都接受了一个 `Apple` 对象做参数。如果查看 `ArrayList` 的源代码，可以发现 `add()` 接受一个泛型类型作为参数，但是 `contains` 和 `indexOf` 接受一个 `Object` 类型的参数，下面是它们的方法签名：

    public boolean add(E e)
    public boolean contains(Object o)
    public int indexOf(Object o)
    

所以如果我们指定泛型参数为 `<? extends Fruit>` 时，`add()` 方法的参数变为 `? extends Fruit`，编译器无法判断这个参数接受的到底是 `Fruit` 的哪种类型，所以它不会接受任何类型。

然而，`contains` 和 `indexOf` 的类型是 `Object`，并没有涉及到通配符，所以编译器允许调用这两个方法。这意味着一切取决于泛型类的编写者来决定那些调用是 “安全” 的，并且用 `Object` 作为这些安全方法的参数。如果某些方法不允许类型参数是通配符时的调用，这些方法的参数应该用类型参数，比如 `add(E e)`。

当我们自己编写泛型类时，上面介绍的就有用了。下面编写一个 `Holder` 类：

    public class Holder<T> {
        private T value;
        public Holder() {}
        public Holder(T val) { value = val; }
        public void set(T val) { value = val; }
        public T get() { return value; }
        public boolean equals(Object obj) {
        return value.equals(obj);
        }
        public static void main(String[] args) {
            Holder<Apple> Apple = new Holder<Apple>(new Apple());
            Apple d = Apple.get();
            Apple.set(d);
            
            Holder<? extends Fruit> fruit = Apple; 
            Fruit p = fruit.get();
            d = (Apple)fruit.get(); 
            try {
                Orange c = (Orange)fruit.get(); 
            } catch(Exception e) { System.out.println(e); }
            
            
            System.out.println(fruit.equals(d)); 
        }
    } 
    

在 `Holer` 类中，`set()` 方法接受类型参数 `T` 的对象作为参数，`get()` 返回一个 `T` 类型，而 `equals()` 接受一个 `Object` 作为参数。`fruit` 的类型是 `Holder<? extends Fruit>`，所以`set()`方法不会接受任何对象的添加，但是 `equals()` 可以正常工作。

### 下边界限定通配符

通配符的另一个方向是　“超类型的通配符“: `? super T`，`T` 是类型参数的下界。使用这种形式的通配符，我们就可以 ”传递对象” 了。还是用例子解释：

    public class SuperTypeWildcards {
        static void writeTo(List<? super Apple> apples) {
            apples.add(new Apple());
            apples.add(new Jonathan());
            
        }
    }

`writeTo` 方法的参数 `apples` 的类型是 `List<? super Apple>`，它表示某种类型的 List，这个类型是 `Apple` 的基类型。也就是说，我们不知道实际类型是什么，但是这个类型肯定是 `Apple` 的父类型。因此，我们可以知道向这个 List 添加一个 `Apple` 或者其子类型的对象是安全的，这些对象都可以向上转型为 `Apple`。但是我们不知道加入 `Fruit` 对象是否安全，因为那样会使得这个 List 添加跟 `Apple` 无关的类型。

在了解了子类型边界和超类型边界之后，我们就可以知道如何向泛型类型中 “写入” ( 传递对象给方法参数) 以及如何从泛型类型中 “读取” ( 从方法中返回对象 )。下面是一个例子：

    public class Collections { 
      public static <T> void copy(List<? super T> dest, List<? extends T> src) 
      {
          for (int i=0; i<src.size(); i++) 
            dest.set(i,src.get(i)); 
      } 
    }

`src` 是原始数据的 List，因为要从这里面读取数据，所以用了上边界限定通配符：`<? extends T>`，取出的元素转型为 `T`。`dest` 是要写入的目标 List，所以用了下边界限定通配符：`<? super T>`，可以写入的元素类型是 `T` 及其子类型。

### 无边界通配符

还有一种通配符是无边界通配符，它的使用形式是一个单独的问号：`List<?>`，也就是没有任何限定。不做任何限制，跟不用类型参数的 `List` 有什么区别呢？

`List<?> list` 表示 `list` 是持有某种特定类型的 List，但是不知道具体是哪种类型。那么我们可以向其中添加对象吗？当然不可以，因为并不知道实际是哪种类型，所以不能添加任何类型，这是不安全的。而单独的 `List list` ，也就是没有传入泛型参数，表示这个 list 持有的元素的类型是 `Object`，因此可以添加任何类型的对象，只不过编译器会有警告信息。

总结
--

通配符的使用可以对泛型参数做出某些限制，使代码更安全，对于上边界和下边界限定的通配符总结如下：

*   使用 `List<? extends C> list` 这种形式，表示 list 可以引用一个 `ArrayList` ( 或者其它 List 的 子类 ) 的对象，这个对象包含的元素类型是 `C` 的子类型 ( 包含 `C` 本身）的一种。
*   使用 `List<? super C> list` 这种形式，表示 list 可以引用一个 `ArrayList` ( 或者其它 List 的 子类 ) 的对象，这个对象包含的元素就类型是 `C` 的超类型 ( 包含 `C` 本身 ) 的一种。

大多数情况下泛型的使用比较简单，但是如果自己编写支持泛型的代码需要对泛型有深入的了解。这几篇文章介绍了泛型的基本用法、类型擦除、泛型数组以及通配符的使用，涵盖了最常用的要点，泛型的总结就写到这里。

**参考**

*   Java 编程思想

**_如果我的文章对您有帮助，不妨点个赞支持一下(^\_^)_**
