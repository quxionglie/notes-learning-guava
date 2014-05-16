[Google Guava] 集合扩展工具类
===========
原文链接 译文链接 译者：沈义扬，校对：丁一

#简介
有时候你需要实现自己的集合扩展。也许你想要在元素被添加到列表时增加特定的行为，或者你想实现一个Iterable，其底层实际上是遍历数据库查询的结果集。Guava为你，也为我们自己提供了若干工具方法，以便让类似的工作变得更简单。（毕竟，我们自己也要用这些工具扩展集合框架。）

#Forwarding装饰器
针对所有类型的集合接口，Guava都提供了Forwarding抽象类以简化装饰者模式的使用。

Forwarding抽象类定义了一个抽象方法：delegate()，你可以覆盖这个方法来返回被装饰对象。所有其他方法都会直接委托给delegate()。例如说：ForwardingList.get(int)实际上执行了delegate().get(int)。

通过创建ForwardingXXX的子类并实现delegate()方法，可以选择性地覆盖子类的方法来增加装饰功能，而不需要自己委托每个方法——译者注：因为所有方法都默认委托给delegate()返回的对象，你可以只覆盖需要装饰的方法。

此外，很多集合方法都对应一个”标准方法[standardxxx]“实现，可以用来恢复被装饰对象的默认行为，以提供相同的优点。比如在扩展AbstractList或JDK中的其他骨架类时，可以使用类似standardAddAll这样的方法。

让我们看看这个例子。假定你想装饰一个List，让其记录所有添加进来的元素。当然，无论元素是用什么方法——add(int, E), add(E), 或addAll(Collection)——添加进来的，我们都希望进行记录，因此我们需要覆盖所有这些方法。

```java
class AddLoggingList<E> extends ForwardingList<E> {
  final List<E> delegate; // backing list
  @Override protected List<E> delegate() {
    return delegate;
  }
  @Override public void add(int index, E elem) {
    log(index, elem);
    super.add(index, elem);
  }
  @Override public boolean add(E elem) {
    return standardAdd(elem); // implements in terms of add(int, E)
  }
  @Override public boolean addAll(Collection<? extends E> c) {
    return standardAddAll(c); // implements in terms of add
  }
}
```

记住，默认情况下，所有方法都直接转发到被代理对象，因此覆盖ForwardingMap.put并不会改变ForwardingMap.putAll的行为。小心覆盖所有需要改变行为的方法，并且确保装饰后的集合满足接口契约。

通常来说，类似于AbstractList的抽象集合骨架类，其大多数方法在Forwarding装饰器中都有对应的”标准方法”实现。

对提供特定视图的接口，Forwarding装饰器也为这些视图提供了相应的”标准方法”实现。例如，ForwardingMap提供StandardKeySet、StandardValues和StandardEntrySet类，它们在可以的情况下都会把自己的方法委托给被装饰的Map，把不能委托的声明为抽象方法。

Interface       | Forwarding Decorator
---             | ---
Collection	    | [ForwardingCollection](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/ForwardingCollection.html)
List	        | [ForwardingList](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/collect/ForwardingList.html)
Set	            | ForwardingSet
SortedSet	    | ForwardingSortedSet
Map	            | ForwardingMap
SortedMap	    | ForwardingSortedMap
ConcurrentMap	| ForwardingConcurrentMap
Map.Entry	    | ForwardingMapEntry
Queue	        | ForwardingQueue
Iterator	    | ForwardingIterator
ListIterator	| ForwardingListIterator
Multiset	    | ForwardingMultiset
Multimap	    | ForwardingMultimap
ListMultimap	| ForwardingListMultimap
SetMultimap	    | ForwardingSetMultimap

#PeekingIterator
有时候，普通的Iterator接口还不够。

Iterators提供一个Iterators.peekingIterator(Iterator)方法，来把Iterator包装为PeekingIterator，这是Iterator的子类，它能让你事先窥视[peek()]到下一次调用next()返回的元素。

注意：Iterators.peekingIterator返回的PeekingIterator不支持在peek()操作之后调用remove()方法。

举个例子：复制一个List，并去除连续的重复元素。

```
List<E> result = Lists.newArrayList();
PeekingIterator<E> iter = Iterators.peekingIterator(source.iterator());
while (iter.hasNext()) {
  E current = iter.next();
  while (iter.hasNext() && iter.peek().equals(current)) {
    // skip this duplicate element
    iter.next();
  }
  result.add(current);
}
```

传统的实现方式需要记录上一个元素，并在特定情况下后退，但这很难处理且容易出错。相较而言，PeekingIterator在理解和使用上就比较直接了。

#AbstractIterator
实现你自己的Iterator？[AbstractIterator](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/AbstractIterator.html)让生活更轻松。

用一个例子来解释AbstractIterator最简单。比方说，我们要包装一个iterator以跳过空值。

```java
public static Iterator<String> skipNulls(final Iterator<String> in) {
  return new AbstractIterator<String>() {
    protected String computeNext() {
      while (in.hasNext()) {
        String s = in.next();
        if (s != null) {
          return s;
        }
      }
      return endOfData();
    }
  };
}
```

你实现了computeNext()方法，来计算下一个值。如果循环结束了也没有找到下一个值，请返回endOfData()表明已经到达迭代的末尾。

注意：AbstractIterator继承了UnmodifiableIterator，所以禁止实现remove()方法。如果你需要支持remove()的迭代器，就不应该继承AbstractIterator。

#AbstractSequentialIterator

有一些迭代器用其他方式表示会更简单。[AbstractSequentialIterator](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/AbstractSequentialIterator.html) 就提供了表示迭代的另一种方式。

```java
 Iterator<Integer> powersOfTwo = new AbstractSequentialIterator<Integer>(1) { // note the initial value!
     protected Integer computeNext(Integer previous) {
       return (previous == 1 << 30) ? null : previous * 2;
     }
   };
```

我们在这儿实现了[computeNext(T)]("http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/collect/AbstractSequentialIterator.html")方法，它能接受前一个值作为参数。

注意，你必须额外传入一个初始值，或者传入null让迭代立即结束。因为computeNext(T)假定null值意味着迭代的末尾——AbstractSequentialIterator不能用来实现可能返回null的迭代器。

（全文完）
