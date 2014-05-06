函数式风格[Functional idioms]
========
Guava的函数式支持可以显著简化代码，但请谨慎使用它

注意事项
截至JDK7，Java中也只能通过笨拙冗长的匿名类来达到近似函数式编程的效果。预计JDK8中会有所改变，但Guava现在就想给JDK5以上用户提供这类支持。

过度使用Guava函数式编程会导致冗长、混乱、可读性差而且低效的代码。这是迄今为止最容易（也是最经常）被滥用的部分，如果你想通过函数式风格达成一行代码，致使这行代码长到荒唐，Guava团队会泪流满面。

比较如下代码：

```java
Function<String, Integer> lengthFunction = new Function<String, Integer>() {
  public Integer apply(String string) {
    return string.length();
  }
};
Predicate<String> allCaps = new Predicate<String>() {
  public boolean apply(String string) {
    return CharMatcher.JAVA_UPPER_CASE.matchesAllOf(string);
  }
};
Multiset<Integer> lengths = HashMultiset.create(
  Iterables.transform(Iterables.filter(strings, allCaps), lengthFunction));
```
或FluentIterable的版本

```java
Multiset<Integer> lengths = HashMultiset.create(
  FluentIterable.from(strings)
    .filter(new Predicate<String>() {
       public boolean apply(String string) {
         return CharMatcher.JAVA_UPPER_CASE.matchesAllOf(string);
       }
     })
    .transform(new Function<String, Integer>() {
       public Integer apply(String string) {
         return string.length();
       }
     }));
```
还有

```
Multiset<Integer> lengths = HashMultiset.create();
for (String string : strings) {
  if (CharMatcher.JAVA_UPPER_CASE.matchesAllOf(string)) {
    lengths.add(string.length());
  }
}
```

即使用了静态导入，甚至把Function和Predicate的声明放到别的文件，第一种代码实现仍然不简洁，可读性差并且效率较低。

截至JDK7，命令式代码仍应是默认和第一选择。不应该随便使用函数式风格，除非你绝对确定以下两点之一：

使用函数式风格以后，整个工程的代码行会净减少。在上面的例子中，函数式版本用了11行， 命令式代码用了6行，把函数的定义放到另一个文件或常量中，并不能帮助减少总代码行。

为了提高效率，转换集合的结果需要懒视图，而不是明确计算过的集合。此外，确保你已经阅读和重读了Effective Java的第55条，并且除了阅读本章后面的说明，你还真正做了性能测试并且有测试数据来证明函数式版本更快。

请务必确保，当使用Guava函数式的时候，用传统的命令式做同样的事情不会更具可读性。尝试把代码写下来，看看它是不是真的那么糟糕？会不会比你想尝试的极其笨拙的函数式 更具可读性。

#Functions[函数]和Predicates[断言]

本节只讨论直接与Function和Predicate打交道的Guava功能。一些其他工具类也和”函数式风格”相关，例如Iterables.concat(Iterable<Iterable>)，和其他用常量时间返回视图的方法。尝试看看2.3节的集合工具类。

Guava提供两个基本的函数式接口：

* Function<A, B>，它声明了单个方法B apply(A input)。Function对象通常被预期为引用透明的——没有副作用——并且引用透明性中的”相等”语义与equals一致，如a.equals(b)意味着function.apply(a).equals(function.apply(b))。
* Predicate<T>，它声明了单个方法boolean apply(T input)。Predicate对象通常也被预期为无副作用函数，并且”相等”语义与equals一致。
特殊的断言

字符类型有自己特定版本的Predicate——CharMatcher，它通常更高效，并且在某些需求方面更有用。CharMatcher实现了Predicate<Character>，可以当作Predicate一样使用，要把Predicate转成CharMatcher，可以使用CharMatcher.forPredicate。更多细节请参考第6章-字符串处理。

此外，对可比较类型和基于比较逻辑的Predicate，Range类可以满足大多数需求——它表示一个不可变区间。Range类实现了Predicate，用以判断值是否在区间内。例如，Range.atMost(2)就是个完全合法的Predicate<Integer>。更多使用Range的细节请参照第8章。

操作Functions和Predicates

Functions提供简便的Function构造和操作方法，包括：
```
forMap(Map<A, B>)
compose(Function<B, C>, Function<A, B>)
constant(T)
identity()
toStringFunction()
```java
细节请参考Javadoc。

相应地，Predicates提供了更多构造和处理Predicate的方法，下面是一些例子：

instanceOf(Class)	assignableFrom(Class)	contains(Pattern)
in(Collection)	isNull()	alwaysFalse()
alwaysTrue()	equalTo(Object)	compose(Predicate, Function)
and(Predicate...)	or(Predicate...)	not(Predicate)
细节请参考Javadoc。

使用函数式编程
Guava提供了很多工具方法，以便用Function或Predicate操作集合。这些方法通常可以在集合工具类找到，如Iterables，Lists，Sets，Maps，Multimaps等。

断言

断言的最基本应用就是过滤集合。所有Guava过滤方法都返回”视图”——译者注：即并非用一个新的集合表示过滤，而只是基于原集合的视图。

集合类型	过滤方法
Iterable	Iterables.filter(Iterable, Predicate)FluentIterable.filter(Predicate)
Iterator	Iterators.filter(Iterator, Predicate)
Collection	Collections2.filter(Collection, Predicate)
Set	Sets.filter(Set, Predicate)
SortedSet	Sets.filter(SortedSet, Predicate)
Map	Maps.filterKeys(Map, Predicate)Maps.filterValues(Map, Predicate)Maps.filterEntries(Map, Predicate)
SortedMap	Maps.filterKeys(SortedMap, Predicate)Maps.filterValues(SortedMap, Predicate)Maps.filterEntries(SortedMap, Predicate)
Multimap	Multimaps.filterKeys(Multimap, Predicate)Multimaps.filterValues(Multimap, Predicate)Multimaps.filterEntries(Multimap, Predicate)
*List的过滤视图被省略了，因为不能有效地支持类似get(int)的操作。请改用Lists.newArrayList(Collections2.filter(list, predicate))做拷贝过滤。

除了简单过滤，Guava另外提供了若干用Predicate处理Iterable的工具——通常在Iterables工具类中，或者是FluentIterable的”fluent”（链式调用）方法。

Iterables方法签名	说明	另请参见
boolean all(Iterable, Predicate)	是否所有元素满足断言？懒实现：如果发现有元素不满足，不会继续迭代	Iterators.all(Iterator, Predicate)FluentIterable.allMatch(Predicate)
boolean any(Iterable, Predicate)	是否有任意元素满足元素满足断言？懒实现：只会迭代到发现满足的元素	Iterators.any(Iterator, Predicate)FluentIterable.anyMatch(Predicate)
T find(Iterable, Predicate)	循环并返回一个满足元素满足断言的元素，如果没有则抛出NoSuchElementException	Iterators.find(Iterator, Predicate)
Iterables.find(Iterable, Predicate, T default)
Iterators.find(Iterator, Predicate, T default)
Optional<T> tryFind(Iterable, Predicate)	返回一个满足元素满足断言的元素，若没有则返回Optional.absent()	Iterators.find(Iterator, Predicate)
Iterables.find(Iterable, Predicate, T default)
Iterators.find(Iterator, Predicate, T default)
indexOf(Iterable, Predicate)	返回第一个满足元素满足断言的元素索引值，若没有返回-1	Iterators.indexOf(Iterator, Predicate)
removeIf(Iterable, Predicate)	移除所有满足元素满足断言的元素，实际调用Iterator.remove()方法	Iterators.removeIf(Iterator, Predicate)
函数

到目前为止，函数最常见的用途为转换集合。同样，所有的Guava转换方法也返回原集合的视图。

集合类型	转换方法
Iterable	Iterables.transform(Iterable, Function)FluentIterable.transform(Function)
Iterator	Iterators.transform(Iterator, Function)
Collection	Collections2.transform(Collection, Function)
List	Lists.transform(List, Function)
Map*	Maps.transformValues(Map, Function)Maps.transformEntries(Map, EntryTransformer)
SortedMap*	Maps.transformValues(SortedMap, Function)Maps.transformEntries(SortedMap, EntryTransformer)
Multimap*	Multimaps.transformValues(Multimap, Function)Multimaps.transformEntries(Multimap, EntryTransformer)
ListMultimap*	Multimaps.transformValues(ListMultimap, Function)Multimaps.transformEntries(ListMultimap, EntryTransformer)
Table	Tables.transformValues(Table, Function)
*Map和Multimap有特殊的方法，其中有个EntryTransformer<K, V1, V2>参数，它可以使用旧的键值来计算，并且用计算结果替换旧值。

*对Set的转换操作被省略了，因为不能有效支持contains(Object)操作——译者注：懒视图实际上不会全部计算转换后的Set元素，因此不能高效地支持contains(Object)。请改用Sets.newHashSet(Collections2.transform(set, function))进行拷贝转换。

01
List<String> names;
02
Map<String, Person> personWithName;
03
List<Person> people = Lists.transform(names, Functions.forMap(personWithName));
04

05
ListMultimap<String, String> firstNameToLastNames;
06
// maps first names to all last names of people with that first name
07

08
ListMultimap<String, String> firstNameToName = Multimaps.transformEntries(firstNameToLastNames,
09
    new EntryTransformer<String, String, String> () {
10
        public String transformEntry(String firstName, String lastName) {
11
            return firstName + " " + lastName;
12
        }
13
    });
可以组合Function使用的类包括：

Ordering	Ordering.onResultOf(Function)
Predicate	Predicates.compose(Predicate, Function)
Equivalence	Equivalence.onResultOf(Function)
Supplier	Suppliers.compose(Function, Supplier)
Function	Functions.compose(Function, Function)
此外，ListenableFuture API支持转换ListenableFuture。Futures也提供了接受AsyncFunction参数的方法。AsyncFunction是Function的变种，它允许异步计算值。

Futures.transform(ListenableFuture, Function)
Futures.transform(ListenableFuture, Function, Executor)
Futures.transform(ListenableFuture, AsyncFunction)
Futures.transform(ListenableFuture, AsyncFunction, Executor)


