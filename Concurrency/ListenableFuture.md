ListenableFuture解析
===========

[原文地址](http://code.google.com/p/guava-libraries/wiki/ListenableFutureExplained)  译者：罗立树  校对：方腾飞

并发编程是一个难题，但是一个强大而简单的抽象可以显著的简化并发的编写。出于这样的考虑，Guava 定义了 ListenableFuture接口并继承了JDK concurrent包下的Future 接口。

我们强烈地建议你在代码中多使用ListenableFuture来代替JDK的 Future, 因为：
* 大多数Futures 方法中需要它。
* 转到ListenableFuture 编程比较容易。
* Guava提供的通用公共类封装了公共的操作方方法，不需要提供Future和ListenableFuture的扩展方法。


#接口

传统JDK中的Future通过异步的方式计算返回结果:在多线程运算中可能或者可能在没有结束返回结果，Future是运行中的多线程的一个引用句柄，确保在服务执行返回一个Result。

ListenableFuture可以允许你注册回调方法(callbacks)，在运算（多线程执行）完成的时候进行调用,  或者在运算（多线程执行）完成后立即执行。这样简单的改进，使得可以明显的支持更多的操作，这样的功能在JDK concurrent中的Future是不支持的。

ListenableFuture 中的基础方法是addListener(Runnable, Executor), 该方法会在多线程运算完的时候，指定的Runnable参数传入的对象会被指定的Executor执行。

#添加回调（Callbacks）

多数用户喜欢使用 Futures.addCallback(ListenableFuture<V>, FutureCallback<V>, Executor)的方式, 或者 另外一个版本version（译者注：addCallback(ListenableFuture<V> future,FutureCallback<? super V> callback)），默认是采用 MoreExecutors.sameThreadExecutor()线程池, 为了简化使用，Callback采用轻量级的设计.  FutureCallback<V> 中实现了两个方法:
* onSuccess(V),在Future成功的时候执行，根据Future结果来判断。
* onFailure(Throwable), 在Future失败的时候执行，根据Future结果来判断。

#ListenableFuture的创建

对应JDK中的 ExecutorService.submit(Callable) 提交多线程异步运算的方式，Guava 提供了ListeningExecutorService 接口, 该接口返回 ListenableFuture 而相应的 ExecutorService 返回普通的 Future。将 ExecutorService 转为 ListeningExecutorService，可以使用MoreExecutors.listeningDecorator(ExecutorService)进行装饰。

```java
ListeningExecutorService service = MoreExecutors.listeningDecorator(Executors.newFixedThreadPool(10));
ListenableFuture explosion = service.submit(new Callable() {
  public Explosion call() {
    return pushBigRedButton();
  }
});
Futures.addCallback(explosion, new FutureCallback() {
  // we want this handler to run immediately after we push the big red button!
  public void onSuccess(Explosion explosion) {
    walkAwayFrom(explosion);
  }
  public void onFailure(Throwable thrown) {
    battleArchNemesis(); // escaped the explosion!
  }
});
```

另外, 假如你是从 FutureTask转换而来的, Guava 提供ListenableFutureTask.create(Callable<V>) 和ListenableFutureTask.create(Runnable, V). 和 JDK不同的是, ListenableFutureTask 不能随意被继承（译者注：ListenableFutureTask中的done方法实现了调用listener的操作）。

假如你喜欢抽象的方式来设置future的值，而不是想实现接口中的方法，可以考虑继承抽象类AbstractFuture<V> 或者直接使用 SettableFuture 。

假如你必须将其他API提供的Future转换成 ListenableFuture，你没有别的方法只能采用硬编码的方式JdkFutureAdapters.listenInPoolThread(Future) 来将 Future 转换成 ListenableFuture。尽可能地采用修改原生的代码返回 ListenableFuture会更好一些。

#Application

使用ListenableFuture 最重要的理由是它可以进行一系列的复杂链式的异步操作。

```
ListenableFuture<RowKey> rowKeyFuture = indexService.lookUp(query);
AsyncFunction<RowKey, QueryResult> queryFunction =
  new AsyncFunction<RowKey, QueryResult>() {
    public ListenableFuture<QueryResult> apply(RowKey rowKey) {
      return dataService.read(rowKey);
    }
  };
ListenableFuture<QueryResult> queryFuture = Futures.transform(rowKeyFuture, queryFunction, queryExecutor);
```

其他更多的操作可以更加有效的支持而JDK中的Future是没法支持的.

不同的操作可以在不同的Executors中执行，单独的ListenableFuture 可以有多个操作等待。

当一个操作开始的时候其他的一些操作也会尽快开始执行–”fan-out”–ListenableFuture 能够满足这样的场景：促发所有的回调（callbacks）。反之更简单的工作是，同样可以满足“fan-in”场景，促发ListenableFuture 获取（get）计算结果，同时其它的Futures也会尽快执行：可以参考 the implementation of Futures.allAsList 。（译者注：fan-in和fan-out是软件设计的一个术语，可以参考这里：http://baike.baidu.com/view/388892.htm#1或者看这里的解析Design Principles: Fan-In vs Fan-Out，这里fan-out的实现就是封装的ListenableFuture通过回调，调用其它代码片段。fan-in的意义是可以调用其它Future）

方法	| 描述	| 参考
--- | --- | ---
transform(ListenableFuture<A>, AsyncFunction<A, B>, Executor)*	 | 返回一个新的ListenableFuture ，该ListenableFuture 返回的result是由传入的AsyncFunction 参数指派到传入的 ListenableFuture中.	       | transform(ListenableFuture<A>, AsyncFunction<A, B>)
transform(ListenableFuture<A>, Function<A, B>, Executor)	       | 返回一个新的ListenableFuture ，该ListenableFuture 返回的result是由传入的Function 参数指派到传入的 ListenableFuture中.	           | transform(ListenableFuture<A>, Function<A, B>)
allAsList(Iterable<ListenableFuture<V>>)	                       | 返回一个ListenableFuture ，该ListenableFuture 返回的result是一个List，List中的值是每个ListenableFuture的返回值，假如传入的其中之一fails或者cancel，这个Future fails 或者canceled	 | allAsList(ListenableFuture<V>...)
successfulAsList(Iterable<ListenableFuture<V>>)	                 | 返回一个ListenableFuture ，该Future的结果包含所有成功的Future，按照原来的顺序，当其中之一Failed或者cancel，则用null替代	          | successfulAsList(ListenableFuture<V>...)

AsyncFunction<A, B> 中提供一个方法ListenableFuture<B> apply(A input)，它可以被用于异步变换值。

```java
List<ListenableFuture> queries;
// The queries go to all different data centers, but we want to wait until they're all done or failed.

ListenableFuture<List> successfulQueries = Futures.successfulAsList(queries);

Futures.addCallback(successfulQueries, callbackOnSuccessfulQueries);
```

#CheckedFuture

Guava也提供了 [CheckedFuture<V, X extends Exception>](http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/util/concurrent/CheckedFuture.html) 接口。CheckedFuture 是一个ListenableFuture ，其中包含了多个版本的get 方法，方法声明抛出检查异常.这样使得创建一个在执行逻辑中可以抛出异常的Future更加容易 。将 ListenableFuture 转换成CheckedFuture，可以使用 Futures.makeChecked(ListenableFuture<V>, Function<Exception, X>)。

（全文完）