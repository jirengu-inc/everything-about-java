
原文：https://www.programcreek.com/2014/05/top-10-mistakes-java-developers-make/

## 1 将 Array 转换成 ArrayList 时出错

一些开发者经常用这样的代码将 Array 转换成 ArrayList

~~~
List<String> list = Arrays.asList(arr);
~~~

Arrays.asList() 的返回值是一个 ArrayList 类的对象，这个 ArrayList 类是 Arrays 类里的一个私有静态类（java.util.Arrays.ArrayList），并不是 java.util.ArrayList 类。

java.util.Arrays.ArrayList 有 set() / get() / contains() 方法，但是并不提供任何添加元素的方法，因此它的长度是固定的。如果你希望得到一个 java.util.ArrayList 类的实例，你应该这么做：

~~~
ArrayList<String> arrayList = new ArrayList<String>(Arrays.asList(arr));
~~~

ArrayList 的构造函数可以接受一个 Collection 实例，而 Collection 是 java.util.Arrays.ArrayList 的超类。

## 2 检查 array 里是否含有某个值时出错

一些开发者会这么写：

~~~
Set<String> set = new HashSet<String>(Arrays.asList(arr));
return set.contains(targetValue);
~~~

上面的代码可以工作，但是其实没必要把 list 转为 set，这有些浪费时间，简单的写法是这样的：

~~~
Arrays.asList(arr).contains(targetValue);
~~~

或者这样的

~~~
for(String s: arr){
	if(s.equals(targetValue))
		return true;
}
return false;
~~~

这两种写法中，前者可读性更好。

## 3 遍历 list 移除元素时出错

下面的代码在迭代时移除了元素：

~~~
ArrayList<String> list = new ArrayList<String>(Arrays.asList("a", "b", "c", "d"));
for (int i = 0; i < list.size(); i++) {
	list.remove(i);
}
System.out.println(list);
~~~

得到的结果是

~~~
[b, d]
~~~

这种代码的问题在于，当元素被移除时，list 的长度也随之变小了，index 也同时发生了变化。所以，如果你想要在循环中使用 index 移除多个元素，它可能不能正常工作。

你可能认为正确的方法是使用迭代器来删除元素，比如 foreach 循环看起来就是一个迭代器，其实这样还是有问题。

考虑以下代码（代码 1）：

~~~
ArrayList<String> list = new ArrayList<String>(Arrays.asList("a", "b", "c", "d"));
 
for (String s : list) {
	if (s.equals("a"))
		list.remove(s);
}
~~~

会抛出 ConcurrentModificationException 异常。

要正确地在遍历时删除元素，应该这么写（代码 2）：

~~~
ArrayList<String> list = new ArrayList<String>(Arrays.asList("a", "b", "c", "d"));
Iterator<String> iter = list.iterator();
while (iter.hasNext()) {
	String s = iter.next();
 
	if (s.equals("a")) {
		iter.remove();
	}
}
~~~

你必须在每次循环里先调用 .next() 再调用 .remove()。

在代码 1 中的 foreach 循环中，编译器会在元素的删除操作之后调用 .next()，导致 ConcurrentModificationException 异常，如果你想深入了解，可以[看看 ArrayList.iterator() 的源码](https://www.programcreek.com/2014/01/deep-understanding-of-arraylist-iterator/)。

## 4 用 Hashtable 还是用 HashMap

一般来说，算法中的 Hashtable 是一种常见的数据结构的名字。但是在 Java 中，这种数据结构的名字却是 HashMap，不是 Hashtable。Java 中 Hashtable 和 HashMap 的最重要的区别之一是 Hashtable 是同步的（synchronized）。因此大部分时候你不需要用 Hashtable，应该用 HashMap。

## 5 直接使用 Collection 的原始类型时出错

在 Java 中，「原始类型」和「无限制通配符类型」很容易被搞混。举例来说，Set 是一个原始类型，而 Set<?> 是一个无限制通配符类型。

下面的代码中的 add 接受原始类型 List 作为参数：

~~~
public static void add(List list, Object o){
	list.add(o);
}
public static void main(String[] args){
	List<String> list = new ArrayList<String>();
	add(list, 10);
	String s = list.get(0);
}
~~~

这个代码会在运行时才抛出异常：

~~~
Exception in thread "main" java.lang.ClassCastException: java.lang.Integer cannot be cast to java.lang.String
	at ...
~~~

使用原始类型的 collection 是很危险的，因为原始类型没有泛型检查。`Set / Set<?> / Set<Object>` 之间有非常大的差异，详情可以看看《[Set vs. Set<?>](https://www.programcreek.com/2013/12/raw-type-set-vs-unbounded-wildcard-set/)》和《[Java Type Erasure Mechanism](https://www.programcreek.com/2011/12/java-type-erasure-mechanism-example/)》。

## 6 访问级别设置过高

很多开发者为了省事，把类字段标记为 public，这不是个好习惯。好习惯应该是将访问级别设置得越低越好。

详见《[public, default, protected, and private](https://www.programcreek.com/2011/11/java-access-level-public-protected-private/)》。


## 7 ArrayList 和 LinkedList 选用错误

如果不了解 ArrayList 和 LinkedList 的区别，你很容易会倾向于使用 ArrayList，因为它看起来更常见。

但是，ArrayList 和 LinkedList 有巨大的性能差异。简单来说，如果 add/remove 操作较多，则应该使用 LinkedList；如果随机访问操作较多，则应该使用 ArrayList。

如果你想深入了解这些性能差异，可以看看《[ArrayList vs. LinkedList vs. Vector](https://www.programcreek.com/2013/03/arraylist-vs-linkedlist-vs-vector/)》。

## 8 可变还是不可变？

不可变对象有很多好处，比如简单、安全等。但是不可变对了要求每次改动都生成新的对象，对象一多就容易对垃圾回收造成压力。我们应该在可变对象和不可变对象上找到一个平衡点。

一般来说，可变对象可以避免产生太多中间对象。一个经典的例子就是连接大量字符串。如果你使用不可变字符串，你就会造出许多用完即弃的中间对象。这既浪费时间又消耗 CPU，所以这种情况下你应该使用可变对象，如 StringBuilder:

~~~
String result="";
for(String s: arr){
	result = result + s;
}
~~~

还有一些情况值得使用可变对象。比如你可以通过将可变对象传入方法来收集多个结果，从而绕开语法的限制。再比如排序和过滤操作，虽然你可以返回新的被排序之后的对象，但是如果元素数量众多，这就会[浪费不少内存](https://stackoverflow.com/a/23616293/1262580)。

扩展阅读《[为什么字符串是不可变的](https://www.programcreek.com/2013/04/why-string-is-immutable-in-java/)》。

## 9 超类和子类的构造函数

~~~
class Super {
  String s;
  public Super(String s){
    this.s = s;
  }
}

public class Sub extend Super{
  int x = 200;
  public Sub(String s){ // 编译错误
  }
  public Sub(){  // 编译错误
    System.out.println("Sub");
  }
  public static void main(String[] args){
    Sub s = new Sub();
  }
}
~~~

上述代码会有编译错误，因为没有实现 Super() 构造函数。Java 中，如果一个类没有定义构造函数，编译器将会插入一个默认的没有参数的构造函数。但是如果 Super 类已经有了一个构造函数 Super(String s)，那么编译器就不会插入这个默认的无参数的构造函数。这就是上述代码的遇到的情况。

Sub 类的两个构造函数，一个有参数一个没有参数，都会调用 Super 类的无参数构造函数。因为编译器会尝试在 Sub 类的两个构造函数里插入 `super()`，由于 Super 类没有无参数构造函数，所以编译器就报错了。

解决这个问题，有三种方法：

1. 给 Super 类添加一个 Super() 无参数构造函数
2. 删掉 Super 类里的有参数的构造函数
3. 在 Sub 类的构造函数里添加 super(value) 

想了解更多详情，可以看《[Constructors of Sub and Super Classes in Java?](https://www.programcreek.com/2013/04/what-are-the-frequently-asked-questions-about-constructors-in-java/)》。

## 10 用 "" 还是用构造函数

字符串可以通过两种途径来构造：

~~~
// 1. 使用双引号
String x = "abd";
// 2. 使用构造函数
String y = new String("abc");
~~~

有什么区别呢？

下面的代码可以很快地告诉你区别：

~~~
String a = "abcd";
String b = "abcd";
System.out.println(a == b);  // True
System.out.println(a.equals(b)); // True
 
String c = new String("abcd");
String d = new String("abcd");
System.out.println(c == d);  // False
System.out.println(c.equals(d)); // True
~~~

想了解这两种方式生成的字符串在内存中是如何存在的，可以看看《[Create Java String Using ” ” or Constructor?](https://www.programcreek.com/2014/03/create-java-string-by-double-quotes-vs-by-constructor/)》

## 总结

这 10 个错误是我综合 GitHub 上的项目、StackOverflow 上的问答和 Google 搜索关键词的趋势而分析得出的。它们可能并不是真正的 10 个最多的错误，但还是挺普遍的。如果你有异议，可以给我留言。如果你能告诉我其他常见的错误，我会非常感谢你。
