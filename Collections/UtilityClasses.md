强大的集合工具类：java.util.Collections中未包含的集合工具
====

任何对JDK集合框架有经验的程序员都熟悉和喜欢java.util.Collections包含的工具方法。Guava沿着这些路线提供了更多的工具方法：适用于所有集合的静态方法。这是Guava最流行和成熟的部分之一。

我们用相对直观的方式把工具类与特定集合接口的对应关系归纳如下：

集合接口             | 属于JDK还是Guava     | 对应的Guava工具类
----                | ---                | ---
Collection          | JDK	              | Collections2：不要和java.util.Collections混淆
List	            | JDK	                 | Lists
Set	                | JDK	                | Sets
SortedSet	        | JDK	                | Sets
Map	                | JDK	| Maps
SortedMap	        | JDK	| Maps
Queue	            | JDK	| Queues
Multiset	        | Guava	| Multisets
Multimap	        | Guava	| Multimaps
BiMap	            | Guava	| Maps
Table	            | Guava	| Tables

在找类似转化、过滤的方法？请看第四章，函数式风格。

#静态工厂方法
在JDK 7之前，构造新的范型集合时要讨厌地重复声明范型：

    List<TypeThatsTooLongForItsOwnGood> list = new ArrayList<TypeThatsTooLongForItsOwnGood>();

我想我们都认为这很讨厌。因此Guava提供了能够推断范型的静态工厂方法：

    List<TypeThatsTooLongForItsOwnGood> list = Lists.newArrayList();
    Map<KeyType, LongishValueType> map = Maps.newLinkedHashMap();

可以肯定的是，JDK7版本的钻石操作符(<>)没有这样的麻烦：


    List<TypeThatsTooLongForItsOwnGood> list = new ArrayList<>();

但Guava的静态工厂方法远不止这么简单。用工厂方法模式，我们可以方便地在初始化时就指定起始元素。

    Set<Type> copySet = Sets.newHashSet(elements);
    List<String> theseElements = Lists.newArrayList("alpha", "beta", "gamma");

此外，通过为工厂方法命名（Effective Java第一条），我们可以提高集合初始化大小的可读性：

```java
List<Type> exactly100 = Lists.newArrayListWithCapacity(100);
List<Type> approx100 = Lists.newArrayListWithExpectedSize(100);
Set<Type> approx100Set = Sets.newHashSetWithExpectedSize(100);
```

确切的静态工厂方法和相应的工具类一起罗列在下面的章节。

注意：Guava引入的新集合类型没有暴露原始构造器，也没有在工具类中提供初始化方法。而是直接在集合类中提供了静态工厂方法，例如：

    Multiset<String> multiset = HashMultiset.create();

#Iterables
在可能的情况下，Guava提供的工具方法更偏向于接受Iterable而不是Collection类型。在Google，对于不存放在主存的集合——比如从数据库或其他数据中心收集的结果集，因为实际上还没有攫取全部数据，这类结果集都不能支持类似size()的操作 ——通常都不会用Collection类型来表示。

因此，很多你期望的支持所有集合的操作都在Iterables类中。大多数Iterables方法有一个在Iterators类中的对应版本，用来处理Iterator。

截至Guava 12版本，Iterables由FluentIterable进行了补充，它包装了一个Iterable实例，并对许多操作提供了”fluent”（链式调用）句法。

下面列出了一些最常用的工具方法，但更多Iterables的函数式方法将在第四章讨论。

常规方法
--                              | ---                            | ---
concat(Iterable<Iterable>)	    | 串联多个iterables的懒视图        | concat(Iterable...)
frequency(Iterable, Object)	    | 返回对象在iterable中出现的次数	 | 与Collections.frequency (Collection,   Object)比较；Multiset
partition(Iterable, int)	    | 把iterable按指定大小分割，得到的子集都不能进行修改操作	| Lists.partition(List, int)；paddedPartition(Iterable, int)
getFirst(Iterable, T default)	| 返回iterable的第一个元素，若iterable为空则返回默认值	| 与Iterable.iterator(). next()比较;FluentIterable.first()
getLast(Iterable)	            | 返回iterable的最后一个元素，若iterable为空则抛出NoSuchElementException	| getLast(Iterable, T default)；
FluentIterable.last()
elementsEqual(Iterable, Iterable)	| 如果两个iterable中的所有元素相等且顺序一致，返回true	| 与List.equals(Object)比较
unmodifiableIterable(Iterable)	    | 返回iterable的不可变视图	                            | 与Collections. unmodifiableCollection(Collection)比较
limit(Iterable, int)	            | 限制iterable的元素个数限制给定值	                    | FluentIterable.limit(int)
getOnlyElement(Iterable)	        | 获取iterable中唯一的元素，如果iterable为空或有多个元素，则快速失败	| getOnlyElement(Iterable, T default)

*译者注：懒视图意味着如果还没访问到某个iterable中的元素，则不会对它进行串联操作。

```java
Iterable<Integer> concatenated = Iterables.concat(
  Ints.asList(1, 2, 3),
  Ints.asList(4, 5, 6));
// concatenated has elements 1, 2, 3, 4, 5, 6

String lastAdded = Iterables.getLast(myLinkedHashSet);

String theElement = Iterables.getOnlyElement(thisSetIsDefinitelyASingleton);
  // if this set isn't a singleton, something is wrong!
  //如果set不是单元素集，就会出错了！
```  

#与Collection方法相似的工具方法

通常来说，Collection的实现天然支持操作其他Collection，但却不能操作Iterable。

下面的方法中，如果传入的Iterable是一个Collection实例，则实际操作将会委托给相应的Collection接口方法。例如，往Iterables.size方法传入是一个Collection实例，它不会真的遍历iterator获取大小，而是直接调用Collection.size。

方法	类似的Collection方法	等价的FluentIterable方法
addAll(Collection addTo,   Iterable toAdd)	Collection.addAll(Collection)	
contains(Iterable, Object)	Collection.contains(Object)	FluentIterable.contains(Object)
removeAll(Iterable   removeFrom, Collection toRemove)	Collection.removeAll(Collection)	
retainAll(Iterable   removeFrom, Collection toRetain)	Collection.retainAll(Collection)	
size(Iterable)	Collection.size()	FluentIterable.size()
toArray(Iterable, Class)	Collection.toArray(T[])	FluentIterable.toArray(Class)
isEmpty(Iterable)	Collection.isEmpty()	FluentIterable.isEmpty()
get(Iterable, int)	List.get(int)	FluentIterable.get(int)
toString(Iterable)	Collection.toString()	FluentIterable.toString()
FluentIterable

除了上面和第四章提到的方法，FluentIterable还有一些便利方法用来把自己拷贝到不可变集合

ImmutableList	
ImmutableSet	toImmutableSet()
ImmutableSortedSet	toImmutableSortedSet(Comparator)
Lists
除了静态工厂方法和函数式编程方法，Lists为List类型的对象提供了若干工具方法。

方法	描述
partition(List, int)	把List按指定大小分割
reverse(List)	返回给定List的反转视图。注: 如果List是不可变的，考虑改用ImmutableList.reverse()。
1
List countUp = Ints.asList(1, 2, 3, 4, 5);
2
List countDown = Lists.reverse(theList); // {5, 4, 3, 2, 1}
3
List<List> parts = Lists.partition(countUp, 2);//{{1,2}, {3,4}, {5}}
静态工厂方法

Lists提供如下静态工厂方法：

具体实现类型	工厂方法
ArrayList	basic, with elements, from Iterable, with exact capacity, with expected size, from Iterator
LinkedList	basic, from Iterable
Sets
Sets工具类包含了若干好用的方法。

集合理论方法

我们提供了很多标准的集合运算（Set-Theoretic）方法，这些方法接受Set参数并返回SetView，可用于：

直接当作Set使用，因为SetView也实现了Set接口；
用copyInto(Set)拷贝进另一个可变集合；
用immutableCopy()对自己做不可变拷贝。
方法
union(Set, Set)
intersection(Set, Set)
difference(Set, Set)
symmetricDifference(Set,   Set)
使用范例：

1
Set<String> wordsWithPrimeLength = ImmutableSet.of("one", "two", "three", "six", "seven", "eight");
2
Set<String> primes = ImmutableSet.of("two", "three", "five", "seven");
3
SetView<String> intersection = Sets.intersection(primes,wordsWithPrimeLength);
4
// intersection包含"two", "three", "seven"
5
return intersection.immutableCopy();//可以使用交集，但不可变拷贝的读取效率更高
其他Set工具方法

方法	描述	另请参见
cartesianProduct(List<Set>)	返回所有集合的笛卡儿积	cartesianProduct(Set...)
powerSet(Set)	返回给定集合的所有子集	
1
Set<String> animals = ImmutableSet.of("gerbil", "hamster");
2
Set<String> fruits = ImmutableSet.of("apple", "orange", "banana");
3
 
4
Set<List<String>> product = Sets.cartesianProduct(animals, fruits);
5
// {{"gerbil", "apple"}, {"gerbil", "orange"}, {"gerbil", "banana"},
6
//  {"hamster", "apple"}, {"hamster", "orange"}, {"hamster", "banana"}}
7
 
8
Set<Set<String>> animalSets = Sets.powerSet(animals);
9
// {{}, {"gerbil"}, {"hamster"}, {"gerbil", "hamster"}}
静态工厂方法

Sets提供如下静态工厂方法：

具体实现类型	工厂方法
HashSet	basic, with elements, from Iterable, with expected size, from Iterator
LinkedHashSet	basic, from Iterable, with expected size
TreeSet	basic, with Comparator, from Iterable
Maps
Maps类有若干值得单独说明的、很酷的方法。

uniqueIndex

Maps.uniqueIndex(Iterable,Function)通常针对的场景是：有一组对象，它们在某个属性上分别有独一无二的值，而我们希望能够按照这个属性值查找对象——译者注：这个方法返回一个Map，键为Function返回的属性值，值为Iterable中相应的元素，因此我们可以反复用这个Map进行查找操作。

比方说，我们有一堆字符串，这些字符串的长度都是独一无二的，而我们希望能够按照特定长度查找字符串：

1
ImmutableMap<Integer, String> stringsByIndex = Maps.uniqueIndex(strings,
2
    new Function<String, Integer> () {
3
        public Integer apply(String string) {
4
            return string.length();
5
        }
6
    });
如果索引值不是独一无二的，请参见下面的Multimaps.index方法。

difference

Maps.difference(Map, Map)用来比较两个Map以获取所有不同点。该方法返回MapDifference对象，把不同点的维恩图分解为：

entriesInCommon()	两个Map中都有的映射项，包括匹配的键与值
entriesDiffering()	键相同但是值不同值映射项。返回的Map的值类型为MapDifference.ValueDifference，以表示左右两个不同的值
entriesOnlyOnLeft()	键只存在于左边Map的映射项
entriesOnlyOnRight()	键只存在于右边Map的映射项
1
Map<String, Integer> left = ImmutableMap.of("a", 1, "b", 2, "c", 3);
2
Map<String, Integer> left = ImmutableMap.of("a", 1, "b", 2, "c", 3);
3
MapDifference<String, Integer> diff = Maps.difference(left, right);
4
 
5
diff.entriesInCommon(); // {"b" => 2}
6
diff.entriesInCommon(); // {"b" => 2}
7
diff.entriesOnlyOnLeft(); // {"a" => 1}
8
diff.entriesOnlyOnRight(); // {"d" => 5}
处理BiMap的工具方法

Guava中处理BiMap的工具方法在Maps类中，因为BiMap也是一种Map实现。

BiMap工具方法	相应的Map工具方法
synchronizedBiMap(BiMap)	Collections.synchronizedMap(Map)
unmodifiableBiMap(BiMap)	Collections.unmodifiableMap(Map)
静态工厂方法

Maps提供如下静态工厂方法：

具体实现类型	工厂方法
HashMap	basic, from Map, with expected size
LinkedHashMap	basic, from Map
TreeMap	basic, from Comparator, from SortedMap
EnumMap	from Class, from Map
ConcurrentMap：支持所有操作	basic
IdentityHashMap	basic
Multisets
标准的Collection操作会忽略Multiset重复元素的个数，而只关心元素是否存在于Multiset中，如containsAll方法。为此，Multisets提供了若干方法，以顾及Multiset元素的重复性：

方法	说明	和Collection方法的区别
containsOccurrences(Multiset   sup, Multiset sub)	对任意o，如果sub.count(o)<=super.count(o)，返回true	Collection.containsAll忽略个数，而只关心sub的元素是否都在super中
removeOccurrences(Multiset   removeFrom, Multiset toRemove)	对toRemove中的重复元素，仅在removeFrom中删除相同个数。	Collection.removeAll移除所有出现在toRemove的元素
retainOccurrences(Multiset   removeFrom, Multiset toRetain)	修改removeFrom，以保证任意o都符合removeFrom.count(o)<=toRetain.count(o)	Collection.retainAll保留所有出现在toRetain的元素
intersection(Multiset,   Multiset)	返回两个multiset的交集;	没有类似方法
01
Multiset<String> multiset1 = HashMultiset.create();
02
multiset1.add("a", 2);
03
 
04
Multiset<String> multiset2 = HashMultiset.create();
05
multiset2.add("a", 5);
06
 
07
multiset1.containsAll(multiset2); //返回true；因为包含了所有不重复元素，
08
//虽然multiset1实际上包含2个"a"，而multiset2包含5个"a"
09
Multisets.containsOccurrences(multiset1, multiset2); // returns false
10
 
11
multiset2.removeOccurrences(multiset1); // multiset2 现在包含3个"a"
12
multiset2.removeAll(multiset1);//multiset2移除所有"a"，虽然multiset1只有2个"a"
13
multiset2.isEmpty(); // returns true
Multisets中的其他工具方法还包括：

copyHighestCountFirst(Multiset)	返回Multiset的不可变拷贝，并将元素按重复出现的次数做降序排列
unmodifiableMultiset(Multiset)	返回Multiset的只读视图
unmodifiableSortedMultiset(SortedMultiset)	返回SortedMultiset的只读视图
1
Multiset<String> multiset = HashMultiset.create();
2
multiset.add("a", 3);
3
multiset.add("b", 5);
4
multiset.add("c", 1);
5
 
6
ImmutableMultiset highestCountFirst = Multisets.copyHighestCountFirst(multiset);
7
//highestCountFirst，包括它的entrySet和elementSet，按{"b", "a", "c"}排列元素
Multimaps
Multimaps提供了若干值得单独说明的通用工具方法

index

作为Maps.uniqueIndex的兄弟方法，Multimaps.index(Iterable, Function)通常针对的场景是：有一组对象，它们有共同的特定属性，我们希望按照这个属性的值查询对象，但属性值不一定是独一无二的。

比方说，我们想把字符串按长度分组。

01
ImmutableSet digits = ImmutableSet.of("zero", "one", "two", "three", "four", "five", "six", "seven", "eight", "nine");
02
Function<String, Integer> lengthFunction = new Function<String, Integer>() {
03
    public Integer apply(String string) {
04
        return string.length();
05
    }
06
};
07
 
08
ImmutableListMultimap<Integer, String> digitsByLength= Multimaps.index(digits, lengthFunction);
09
/*
10
*  digitsByLength maps:
11
*  3 => {"one", "two", "six"}
12
*  4 => {"zero", "four", "five", "nine"}
13
*  5 => {"three", "seven", "eight"}
14
*/
invertFrom

鉴于Multimap可以把多个键映射到同一个值（译者注：实际上这是任何map都有的特性），也可以把一个键映射到多个值，反转Multimap也会很有用。Guava 提供了invertFrom(Multimap toInvert,
Multimap dest)做这个操作，并且你可以自由选择反转后的Multimap实现。

注：如果你使用的是ImmutableMultimap，考虑改用ImmutableMultimap.inverse()做反转。

01
ArrayListMultimap<String, Integer> multimap = ArrayListMultimap.create();
02
multimap.putAll("b", Ints.asList(2, 4, 6));
03
multimap.putAll("a", Ints.asList(4, 2, 1));
04
multimap.putAll("c", Ints.asList(2, 5, 3));
05
 
06
TreeMultimap<Integer, String> inverse = Multimaps.invertFrom(multimap, TreeMultimap<String, Integer>.create());
07
//注意我们选择的实现，因为选了TreeMultimap，得到的反转结果是有序的
08
/*
09
* inverse maps:
10
*  1 => {"a"}
11
*  2 => {"a", "b", "c"}
12
*  3 => {"c"}
13
*  4 => {"a", "b"}
14
*  5 => {"c"}
15
*  6 => {"b"}
16
*/
forMap

想在Map对象上使用Multimap的方法吗？forMap(Map)把Map包装成SetMultimap。这个方法特别有用，例如，与Multimaps.invertFrom结合使用，可以把多对一的Map反转为一对多的Multimap。

1
Map<String, Integer> map = ImmutableMap.of("a", 1, "b", 1, "c", 2);
2
SetMultimap<String, Integer> multimap = Multimaps.forMap(map);
3
// multimap：["a" => {1}, "b" => {1}, "c" => {2}]
4
Multimap<Integer, String> inverse = Multimaps.invertFrom(multimap, HashMultimap<Integer, String>.create());
5
// inverse：[1 => {"a","b"}, 2 => {"c"}]
包装器

Multimaps提供了传统的包装方法，以及让你选择Map和Collection类型以自定义Multimap实现的工具方法。

只读包装	Multimap	ListMultimap	SetMultimap	SortedSetMultimap
同步包装	Multimap	ListMultimap	SetMultimap	SortedSetMultimap
自定义实现	Multimap	ListMultimap	SetMultimap	SortedSetMultimap
自定义Multimap的方法允许你指定Multimap中的特定实现。但要注意的是：

Multimap假设对Map和Supplier产生的集合对象有完全所有权。这些自定义对象应避免手动更新，并且在提供给Multimap时应该是空的，此外还不应该使用软引用、弱引用或虚引用。
无法保证修改了Multimap以后，底层Map的内容是什么样的。
即使Map和Supplier产生的集合都是线程安全的，它们组成的Multimap也不能保证并发操作的线程安全性。并发读操作是工作正常的，但需要保证并发读写的话，请考虑用同步包装器解决。
只有当Map、Supplier、Supplier产生的集合对象、以及Multimap存放的键值类型都是可序列化的，Multimap才是可序列化的。
Multimap.get(key)返回的集合对象和Supplier返回的集合对象并不是同一类型。但如果Supplier返回的是随机访问集合，那么Multimap.get(key)返回的集合也是可随机访问的。
请注意，用来自定义Multimap的方法需要一个Supplier参数，以创建崭新的集合。下面有个实现ListMultimap的例子——用TreeMap做映射，而每个键对应的多个值用LinkedList存储。

1
ListMultimap<String, Integer> myMultimap = Multimaps.newListMultimap(
2
    Maps.<String, Collection>newTreeMap(),
3
    new Supplier<LinkedList>() {
4
        public LinkedList get() {
5
            return Lists.newLinkedList();
6
        }
7
    });
Tables
Tables类提供了若干称手的工具方法。

自定义Table

堪比Multimaps.newXXXMultimap(Map, Supplier)工具方法，Tables.newCustomTable(Map, Supplier<Map>)允许你指定Table用什么样的map实现行和列。

1
// 使用LinkedHashMaps替代HashMaps
2
Table<String, Character, Integer> table = Tables.newCustomTable(
3
Maps.<String, Map<Character, Integer>>newLinkedHashMap(),
4
new Supplier<Map<Character, Integer>> () {
5
public Map<Character, Integer> get() {
6
return Maps.newLinkedHashMap();
7
}
8
});
transpose

transpose(Table<R, C, V>)方法允许你把Table<C, R, V>转置成Table<R, C, V>。例如，如果你在用Table构建加权有向图，这个方法就可以把有向图反转。

包装器

还有很多你熟悉和喜欢的Table包装类。然而，在大多数情况下还请使用ImmutableTable

Unmodifiable	Table	RowSortedTable
（全文完）