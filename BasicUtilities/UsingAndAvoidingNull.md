使用和避免null
======

null是模棱两可的，会引起令人困惑的错误，有些时候它让人很不舒服。很多Guava工具类用快速失败拒绝null值，而不是盲目地接受

#1. Optional

大多数情况下，开发人员使用null表明的是某种缺失情形：可能是已经有一个默认值，或没有值，或找不到值。例如，Map.get返回null就表示找不到给定键对应的值。

Guava用Optional<T>表示可能为null的T类型引用。一个Optional实例可能包含非null的引用（我们称之为引用存在），也可能什么也不包括（称之为引用缺失）。它从不说包含的是null值，而是用存在或缺失来表示。但Optional从不会包含null值引用。

```java
Optional<Integer> possible = Optional.of(5);
possible.isPresent(); // returns true
possible.get(); // returns 5
```

Optional无意直接模拟其他编程环境中的”可选” or “可能”语义，但它们的确有相似之处。

Optional最常用的一些操作被罗列如下：

创建Optional实例（以下都是静态方法）：

方法							| 说明 	
----						| ------											
Optional.of(T)				| 创建指定引用的Optional实例，若引用为null则快速失败 	
Optional.absent()			| 创建引用缺失的Optional实例 						
Optional.fromNullable(T)	| 创建指定引用的Optional实例，若引用为null则表示缺失 	

用Optional实例查询引用（以下都是非静态方法）：

方法							| 说明 	
----						| ------	
boolean isPresent()			| 如果Optional包含非null的引用（引用存在），返回true
T get()						| 返回Optional所包含的引用，若引用缺失，则抛出java.lang.IllegalStateException
T or(T)						| 返回Optional所包含的引用，若引用缺失，返回指定的值
T orNull()					| 返回Optional所包含的引用，若引用缺失，返回null
Set<T> asSet()				| 返回Optional所包含引用的单例不可变集，如果引用存在，返回一个只有单一元素的集合，如果引用缺失，返回一个空集合。

使用Optional的意义在哪儿？

使用Optional除了赋予null语义，增加了可读性，最大的优点在于它是一种傻瓜式的防护。Optional迫使你积极思考引用缺失的情况，因为你必须显式地从Optional获取引用。直接使用null很容易让人忘掉某些情形，尽管FindBugs可以帮助查找null相关的问题，但是我们还是认为它并不能准确地定位问题根源。

如同输入参数，方法的返回值也可能是null。和其他人一样，你绝对很可能会忘记别人写的方法method(a,b)会返回一个null，就好像当你实现method(a,b)时，也很可能忘记输入参数a可以为null。将方法的返回类型指定为Optional，也可以迫使调用者思考返回的引用缺失的情形。

其他处理null的便利方法

当你需要用一个默认值来替换可能的null，请使用Objects.firstNonNull(T, T) 方法。如果两个值都是null，该方法会抛出NullPointerException。Optional也是一个比较好的替代方案，例如：Optional.of(first).or(second).

还有其它一些方法专门处理null或空字符串：emptyToNull(String)，nullToEmpty(String)，isNullOrEmpty(String)。我们想要强调的是，这些方法主要用来与混淆null/空的API进行交互。当每次你写下混淆null/空的代码时，Guava团队都泪流满面。（好的做法是积极地把null和空区分开，以表示不同的含义，在代码中把null和空同等对待是一种令人不安的坏味道。

#2. 如何使用
##你应该更新的Java知识之Optional
[你应该更新的Java知识之Optional](http://www.blogbus.com/logs/235329092.html)

java.lang.NullPointerException，只要敢自称Java程序员，那对这个异常就再熟悉不过了。为了防止抛出这个异常，我们经常会写出这样的代码：

```java
Person person = people.find("John Smith");
if (person != null) {
 person.doSomething();
}
```

使用Optional

```java
Optional person = people.find("John Smith");
if (person.isPresent()) {
 person.get().doSomething();
}
```

这里如果isPresent()返回false，说明这是个空对象，否则，我们就可以把其中的内容取出来做自己想做的操作了。

如果你期待的是代码量的减少，恐怕这里要让你失望了。单从代码量上来说，Optional甚至比原来的代码还多。但好处在于，你绝对不会忘记判空，因为这里我们得到的不是Person类的对象，而是Optional。

有时候，当一个对象为null的时候，我们并不是简单的忽略，而是给出一个缺省值，比如找不到这个人，任务就交给经理来做。使用Optional可以很容易地做到这一点，以上面的代码为例：

```java
  Optional person = people.find("John Smith");
  person.or(manager).doSomething()
```

说白了，Optinal是给了我们一个更有意义的“空”。

##你应该更新的Java知识之Optional高级用法
http://dreamhead.blogbus.com/logs/235334714.html


#3. 如何正确的使用 Optional

###Guava Optional. How to use the correct

http://stackoverflow.com/questions/11561789/guava-optional-how-to-use-the-correct

####Answer
Guava contributor here...

Any or all of these things are fine, but some of them may be overkill.

Generally, as discussed in [this StackOverflow answer](http://stackoverflow.com/a/9561334/869736), Optional is primarily used for two things: to make it clearer what you would've meant by null, and in method return values to make sure the caller takes care of the "absent" case (which it's easier to forget with null). We certainly don't advocate replacing every nullable value with an Optional everywhere in your code -- we certainly don't do that within Guava itself!

A lot of this will have to be your decision -- there's no universal rule, it's a relatively subjective judgement, and I don't have enough context to determine what I'd do in your place -- but based on what context you've provided, I'd consider making the methods return Optional, but probably wouldn't change any of the other fields or anything.

-------------

###What's the point of Guava's Optional class

http://stackoverflow.com/questions/9561295/whats-the-point-of-guavas-optional-class

####Question
I've recently read about this and seen people using this class, but in pretty much all cases, using null would've worked as well - if not more intuitively. Can someone provide a concrete example where Optional would achieve something that null couldn't or in a much cleaner way? The only thing I can think of is to use it with Maps that don't accept null keys, but even that could be done with a side "mapping" of null's value. Can anyone provide me with a more convincing argument? Thank you.

####Answers
Guava team member here.

Probably the single biggest disadvantage of null is that it's not obvious what it should mean in any given context: it doesn't have an illustrative name. It's not always obvious that null means "no value for this parameter" -- heck, as a return value, sometimes it means "error", or even "success" (!!), or simply "the correct answer is nothing". Optional is frequently the concept you actually mean when you make a variable nullable, but not always. When it isn't, we recommend that you write your own class, similar to Optional but with a different naming scheme, to make clear what you actually mean.

But I would say the biggest advantage of Optional isn't in readability: the advantage is its idiot-proof-ness. It forces you to actively think about the absent case if you want your program to compile at all, since you have to actively unwrap the Optional and address that case. Null makes it disturbingly easy to simply forget things, and though FindBugs helps, I don't think it addresses the issue nearly as well. This is especially relevant when you're returning values that may or may not be "present." You (and others) are far more likely to forget that other.method(a, b) could return a null value than you're likely to forget that a could be null when you're implementing other.method. Returning Optional makes it impossible for callers to forget that case, since they have to unwrap the object themselves.

For these reasons, we recommend that you use Optional as a return type for your methods, but not necessarily in your method arguments.

(This is totally cribbed, by the way, from the discussion [here](https://plus.google.com/103410944693236881656/posts/ayfR8F56PGy).)

#4. 总结

* Optinal是给了一个更有意义的“空”；
* Optional并不能使代码量的减少。但好处在于，你绝对不会忘记判空；
* 不要在代码中每一个可能为空的值都使用Optinal替换；

扩展: @Nullable @ParametersAreNonnullByDefault






