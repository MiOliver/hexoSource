---
title: Java 泛型
date: 2017-03-22 11:21:20
tags:
- java
- generic
- 泛型
categories:
- Java
---

### 为什么要使用泛型程序设计
泛型程序设计意味着编写的代码可以被很多不同类型的对象所重用.
在Java中增加泛型类之前, 泛型程序设计是用继承实现的. 例如ArrayList类只维护一个Object引用的数组:
```
public class ArrayList {
  private Object[] elementData;
  public Object get(int i) {...}
  public void add(Object o) {...}
}
```
这样实现有两个问题.

1. 当获取一个值时必须进行强制类型转换.
```
ArrayList first = new ArrayList();
...;
String filename = (String) files.get(0);
```

2. 没有进行错误检查, 可以向数组列表中添加任何类的对象.
```
files.add(new file("..."));
```
而泛型提供了一个更好的解决方案: 类型参数:
```
ArrayList<String> files = new ArrayList<String>();
```
备注:

问题1: 为什么针对files.get(0), 强制转换为String可以成功?
答: 虽然ArrayList存储的是Object, 但它也会存储额外的信息, 用来确定所存储的元素类型, 所以才能保证强制转换的正确性.

问题2: 为什么是以Object为引用?
答: 因为Object是Java中的最原始的类型, 除了基本数据类型外, 所有的对象均派生于Object, 即所有的对象都可向上转型为Object.



### 定义简单泛型类

```
public class Pair<T> {
  private T first;
  private T second;

  public Pair() { first = null; second = null;}
  public Pair(T first, T second) {this.first = first; this.second = second;}

  public T getFirst() {return first;}
  public T getSecond() {return second;}

  public void setFirst(T newValue) {first = newValue;}
  public void setSecond(T newValue) {second = newValue;}
}
```
 在Java中使用:
 变量E表示集合的元素类型;
  K和V分别表示表的关键字与值的类型;
  T, U和S表示任意类型;



### 定义泛型方法
```
class ArrayAlg {
  public static <T> T getMiddle(T... a) {
    return a[a.length / 2];
  }
}
```
然后, 我们可以这样进行调用:
```
String middle = ArrayAlg.<String>getMiddle("hello", "world", "Java");
```
备注:

1. 泛型方法既可以定义在普通类中, 也可以定义在泛型类中.
2. 如果泛型方法的参数类型可以推导出来, 则可省略, 如
```
String middle = ArrayAlg.getMiddle("John", "Q.", "Public");
```
但是在无法推导出来时候, 例如double和int, 则还是需要类型参数.
```
// ERROR
double middle = ArrayAlg.getMiddle(3.14, 1729, 0);
```
类型变量的限定

有时, 类或方法需要对类型变量加以约束. 例如我们计算数组中的最小元素:
```
class ArrayAlg {
  public static <T> T min(T[] a) {
    if (a == null || a.length == 0) return null;
    T smallest = a[0];
    for (int i = 1; i < a.length; i++) {
      if (smallest.compareTo(a[i]) > 0) smallest = a[i];
    }
    return smallest;
  }
}
```
这里存在一个问题在于: 调用compareTo方法的对象必须实现了Comparable接口才行, 而T类型并不确定是否实现了Comparable接口.

我们需要扩展T类型:
```
public static <T extends Comparable> T min(T[] a) {...}
```
而一个类型变量或通配符可以有多个限定:
```
T extends Comparable & Serializable
```
一个实际的例子:
```
public class PairTest2 {
  public static void main(String[] args) {
    String[] strs = new String[]{"hello", "world", "i", "love", "coding"};
    Pair<String> mm = ArrayAlg.minmax(strs);
    System.out.println("min=" + mm.getFirst());
    System.out.println("max=" + mm.getSecond());
  }
}

class ArrayAlg {
  public static <T extends Comparable> Pair<T> minmax(T[] a) {
    if (a == null || a.length == 0) return null;
    T min = a[0];
    T max = a[0];
    for (int i = 1; i < a.length; i++) {
      if (min.compareTo(a[i]) > 0) min = a[i];
      if (max.compareTo(a[i]) < 0) max = a[i];
    }
    return new Pair<>(min, max);
  }
}
```

### 泛型代码和虚拟机

虚拟机没有泛型类型对象--所有对象都属于普通类.

无论何时定义一个泛型类型, 都自动提供了一个相应的原始类型. 原始类型的名字就是删去类型参数后的泛型类型名. 擦除类型变量, 并替换为限定类型.

如Pair<T>的原始类型如下:
```
public class Pair {
  private Object first;
  private Object second;
  ......
}
```
而假定对T进行了扩展, 则为扩展的类型:
```
public class Interval<T extends Comparable & Serializable> implements Serializable {}
```
其中原始类型为: Comparable

但如果某些变量的类型为Serializable, 则编译器在必要时候进行强制转换(Comparable --> Serializable)

翻译泛型表达式
```
Pair<Employee> buddies = ...;
Employee buddy = buddies.getFirst();
```
擦除getFirst的返回类型后将返回Object类型. 编译器自动插入Employee的强制类型转换. 即编译器把这个方法调用翻译为两条虚拟机指令:

1. 对原始方法Pair.getFirst的调用.

2. 将返回的Object类型强制转换为Employee类型.

由于对象buddies会存储实际类型的信息(Employee), 所以可以保证强制类型转换成功.

翻译泛型方法

类型擦除也会出现在泛型方法中:
public static <T extends Comparable> T min(T[] a);
经过类型擦除后变成:
public static Comparable min(Comparable[] a);
但方法的擦除带来两个复杂的问题, 例如:
class DateInterval extends Pair<Date> {
  public void setSecond(Date second) {...}
}
这里由于Pair也有setSecond(Date d)方法, 所以它们为同样的方法, 动态运行时候可以绑定变量的类型, 决定调用哪个方法(多态).

但是由于类型擦除后:

class DateInterval extends Pair {
  public void setSecond(Date second) {...}
}
Pair中的setSecond为: public void setSecond(Object second) {...}, 所以无法进行动态绑定(Date和Object为不同的类型, 即此时两个setSecond为不同的方法).

由于类型擦除导致多态失效. 所以我们需要用桥方法将两个setSecond方法"多态"起来:

class DateInterval extends Pair {
  public void setSecond(Date second) {...}
  public void setSecond(Object second) {
    setSecond((Date) second);
  }
}
编译器生成了第二个setSecond方法, 从而解决了擦除导致多态失效的问题.

总结如下:

1. 虚拟机中没有泛型, 只有普通的类和方法.

2. 所有的类型参数都用它们的限定类型替换.

3. 桥方法被合成保持多态.

4. 为保持类型安全性, 必要时插入强制类型转换.



### 约束与局限性

1. 不能用基本类型实例化类型参数

例如没有Pair<double>, 只有Pair<Double>, 因为擦除后只有Object, 而Object不能存储double类型.

运行时类型查询只适用于原始类型

由于存在类型擦除, 所以泛型类型实际上存储的是原始类型. 所以:

if (a instanceof Pair<String>)
是语法错误的.

if (a instanceof Pair<Object>)
也是语法错误的. 因为a被当做Pair类型, 而元素类型被擦除为Object而已, 它本身为一个普通的类, 不存在任何的泛型信息.

所以

if (a instanceof Pair)
是正确的.

同理, 任何Pair的getClass肯定都等于Pair.class:

Pair<String> strPair = ...;
Pair<Employee> empPair = ...;
strPair.getClass() == empPair.getClass();

2. 不能创建参数化类型的数组

之所以不能实例化参数类型的数组, 是因为数组会记住它元素的类型, 例如字符串的数组是不能存储浮点数的.

而如果对泛型数组进行实例化, 由于擦除的存在, 导致数组的类型为Object, 则可以存储任何的类型, 这跟数组的语法相冲突, 所以在语法层面上, 参数化类型的数组本身是不允许的. 如:

Pair<String>[] table = new Pair<String>[10]; //ERROR
Varargs警告

假设我们编写如下的代码:

public static <T> void addAll(Collection<T> coll, T... ts) {
  for (t: ts) coll.add(t);
}
Collection<Pair<String>> table = ...;
Pair<String> pair1 = ...;
Pair<String> pair2 = ...;
addAll(table, pair1, pair2);
这在语法层面是没有问题的, 运行起来是存在警告的, 是因为虚拟机会建立一个Pair<String>数组, 而这违反了"不能创建参数化类型的数组".

这里之所以正确是因为: 1. 数组的存储空间在编译时期确定的, 所以需要确定数组元素的类型. 2. 而针对集合Collection来说, 它的存储空间是动态递增的, 所以无需考虑元素的类型.

这可以增加@SafeVarargs来抑制这个警告.

@SafeVarargs
public static <T> void addAll(Collection<T> coll, T... ts){}
不能实例化类型变量

不能使用像new T(...), new T[...]或T.class这样的表达式中的类型变量, 例如下例的Pair<T>构造器是非法的:

public Pair() {first = new T(); second = new T();}
因为类型擦除将T改变为Object, 而new Object()肯定不是代码的本意.

同理, 我们也不能使用:

first = T.class.newInstance();
因为类型擦除的存在, T.class无法明确其Class类型. 所以我们需要显式的指明其Class类型:

public static <T> Pair<T> makePair(Class<T> c1) {
  try {return new Pair<>(c1.newInstance(), c1.newInstance());}
  catch (Exception ex) {return null;}
}
我们可以这样调用:

Pair<String> p = Pair.makePair(String.class);
而new T[...]着实让人头疼, 因为类型擦除的原因导致无法确切知道数组的原始类型(语法层面上数组必须知道其元素类型, 才能判断出String[]存储double时候会报错), 则我们需要反射的机制(在运行时获取其class的信息, 从而获取其具体的类型, 则可进行new的操作)动态获取其数据类型, 来进行new T[...].

public static <T extends Comparable> T[] minmax(T... a) {
  T[] mm = (T[])Array.newInstance(a.getClass().getComponentType(), 2);
}

3. 泛型类的静态上下文中类型变量无效

静态的方法或变量是跟具体的类实例无关的, 而泛型的存在本身就跟具体的类实例有关, 两者冲突导致静态域或方法中引用类型变量是无效的.

public class Singleton<T> {
  private static T singleInstance; //ERROR
  public static T getSingleInstance() {} //ERROR
}

4. 不能抛出或捕获泛型类的实例

因为一旦类型擦除, 根本就不确定其具体的异常类型.
```
public static <T extends Throwable> void doWork(Class<T> t) {
  try {

  } catch (Throwable e) { //OK

  } catch (T e) { //ERROR

  }
}
```
由于不能抛出或捕获泛型类, 所以也不能对泛型类进行扩展Exception:

public class Problem<T> extends Exception {} // ERROR
备注: 对"可以消除已检查异常的检查", 不太理解(书章节12.6.7, p543)

擦除后的冲突

例如我们编写如下的代码:

public class Pair<T> {
  public boolean equals(T value) {return first.equals(value) && second.equals(value);}
}
由于擦除的存在, 导致Pair<String>实际上有两个equals:

boolean equals(String) //defined in Pair<T>
boolean equals(Object) //inherited from Objects
要么使用"桥方法", 要么重命名函数进行修复.

备注: 泛型规范的原则之一: 要想支持擦除的转换, 就需要强行限制一个类或类型变量不能同时成为两个接口类型的子类, 而这两个接口是同一个接口的不同参数化.



### 泛型类型的继承规则

无论S与T有什么联系(例如子类和父类的关系), Pair<S>和Pair<T>均没有任何关系. 因为Pair<S>和Pair<T>的本质类型都是Pair.

```

class A {
  private String s;
  A(String s) {
    this.s = s;
  }
  public String show() {
    return s;
  }
}
class B extends A {
  B(String s) {
    super(s);
  }
}
public class PairTest1 {
  public static void main(String[] args) {
    Pair<B> b = new Pair<>(new B("hello"), new B("world"));
//    Pair<A> a = b; // ERROR, Pair<B>无法转换为Pair<A>
    Pair c = b;
    c.setFirst(new B("java"));
    System.out.println(((B)c.getFirst()).show());
  }
}

```
### 通配符类型

Pair<? extends Employee>表示任何泛型Pair类型, 它的类型参数是Employee的子类, 如Pair<Manager>, 但不是Pair<String>.

所以, 如果我们要打印出所有雇员的信息, 不能定义:

public static void printBuddies(Pair<Employee> p);
而应该定义:

public static void printBuddies(Pair<? extends Employee> p);
备注:

1. 针对语法糖extends, 它往往表示扩展某个接口,类型或者继承了某个类. 例如interface A extends B, 则说明接口A扩展了接口B, class A extends B, 代表A继承B.

所以** A extends B, 则类型为B.

2. 针对A extends B来说, 只适合get的操作, 因为明确知道其基本类型为B, 但不能执行set操作, 因为不知道具体类型是什么.

通配符的超类型限定

与"? extends Employee"相反, "? super Manager"限制为Manager的所有超类型.

void setFirst(? super Manager);
? extends Employee getFirst();
备注: 针对? super Manager, 只适合set的操作, 因为知道具体类型为Manager, 但不能执行get操作, 因为不知道其基本类型.

无限定通配符

对于Pair<?>的方法:

? getFirst()
void setFirst(?)
getFirst的返回值只能赋给一个Object. setFirst方法不能被调用, 甚至不能用Object调用. Pair<?>和Pair本质的不同在于: 可以用任意Object对象调用原始的Pair类的setObject方法.

备注: 这里setObject泛指一切set的方法.

所以如果我们要测试一个Pair是否包含一个null引用, 则可以这样定义:

public static boolean hasNulls(Pair<?> p) {
  return p.getFirst() == null || p.getSecond() == null;
}
而无需定义成:

public static <T> boolean hasNulls(Pair<T> p){}
