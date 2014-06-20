Guava学习笔记
======

[Guava学习笔记](http://xionglie.github.io/notes-learning-guava)

笔记的大部分中文翻译内容来源于以下

[Google Guava官方教程（中文版）](http://ifeve.com/google-guava/)

[英文原文链接](http://code.google.com/p/guava-libraries/wiki/GuavaExplained)

译者: 沈义扬，罗立树，何一昕，武祖  校对：方腾飞

感谢[并发编程网](http://ifeve.com/)给我们带来的这一系列好文章。

引言

Guava工程包含了若干被Google的 Java项目广泛依赖 的核心库，例如：集合 [collections] 、缓存 [caching] 、原生类型支持 [primitives support] 、并发库 [concurrency libraries] 、通用注解 [common annotations] 、字符串处理 [string processing] 、I/O 等等。 所有这些工具每天都在被Google的工程师应用在产品服务中。

查阅Javadoc并不一定是学习这些库最有效的方式。在此，我们希望通过此文档为Guava中最流行和最强大的功能，提供更具可读性和解释性的说明。

目录

#1. 基本工具 [Basic utilities](BasicUtilities/README.md)
让使用Java语言变得更舒适

1.1 使用和避免null：null是模棱两可的，会引起令人困惑的错误，有些时候它让人很不舒服。很多Guava工具类用快速失败拒绝null值，而不是盲目地接受

1.2 前置条件: 让方法中的条件检查更简单

1.3 常见Object方法: 简化Object方法实现，如hashCode()和toString()

1.4 排序: Guava强大的”流畅风格比较器”

1.5 Throwables：简化了异常和错误的传播与检查

#2. 集合[Collections](Collections/README.md)
Guava对JDK集合的扩展，这是Guava最成熟和为人所知的部分

2.1 不可变集合: 用不变的集合进行防御性编程和性能提升。

2.2 新集合类型: multisets, multimaps, tables, bidirectional maps等

2.3 强大的集合工具类: 提供java.util.Collections中没有的集合工具

2.4 扩展工具类：让实现和扩展集合类变得更容易，比如创建Collection的装饰器，或实现迭代器

#3. 缓存[Caches](Caches/README.md)
Guava Cache：本地缓存实现，支持多种缓存过期策略

#4. 函数式风格[Functional idioms](FunctionalIdioms/README.md)
Guava的函数式支持可以显著简化代码，但请谨慎使用它

#5. 并发[Concurrency](Concurrency/README.md)
强大而简单的抽象，让编写正确的并发代码更简单

5.1 ListenableFuture：完成后触发回调的Future

5.2 Service框架：抽象可开启和关闭的服务，帮助你维护服务的状态逻辑

#6. 字符串处理[Strings](Strings/README.md)
非常有用的字符串工具，包括分割、连接、填充等操作

#7. 原生类型[Primitives](Primitives/README.md)
扩展 JDK 未提供的原生类型（如int、char）操作， 包括某些类型的无符号形式

#8. 区间[Ranges](Ranges/README.md)
可比较类型的区间API，包括连续和离散类型

#9. [I/O](IO/README.md)
简化I/O尤其是I/O流和文件的操作，针对Java5和6版本

#10. 散列[Hash](Hash/README.md)
提供比Object.hashCode()更复杂的散列实现，并提供布鲁姆过滤器的实现

#11. 事件总线[EventBus](EventBus/README.md)
发布-订阅模式的组件通信，但组件不需要显式地注册到其他组件中

#12. 数学运算[Math](Math/README.md)
优化的、充分测试的数学工具类

#13. 反射[Reflection](Reflection/README.md)
Guava 的 Java 反射机制工具类
