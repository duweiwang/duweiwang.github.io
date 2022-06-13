---
title: 并发编程设计模式
tags: Java
----------

+ Active Object
    方法的调用和执行分离，调用是同步的，执行是异步的。
    核心：对象自己维护一个线程，通过线程安全的队列实现同步调用。
+ Async Method Invocation
    调用方法的线程不堵塞等待结果，直接返回。用于并行处理多个无依赖关系的任务。可以通过回调回去结果，也可以调用堵塞方法获取结果
+ Balking
    阻行模式，只有处于正确的状态才走对应的逻辑。
+ Commander

+ Event Queue
    访问有限资源时，对请求进行排队依次响应。
+ Event-based Asynchronous
+ Guarded Suspension
    保护性挂起模式：操作满足条件才执行，否则挂起等待条件的发生。
+ half-Sync/Half-Async
    半同步半异步模式：将任务分成两个阶段，同步阶段和异步阶段。同步阶段做一些校验逻辑，在主线程判断是否真的需要执行此任务。需要执行才切线程，否则拦截，避免线程切换。
    核心方法是：onPreCall、onCall、onPostCall
+ Leader/Followers

+ Lockable Object

+ Master-Worker
    集中化并行处理。Map-reduce。将大任务分割成多个小任务，再将小任务的结果合起来。
+ Monitor
    Monitor其实就是Java的Synchronized关键字，也称为管程。安全的访问对象，避免多线程冲突
+ Producer Consumer
    减少生产者和消费者之间的耦合
+ Promise
    像CompletableFuture一样，通过一个类来代理异步执行的结果。通过这个类判断异步是否成功，成功才返回结果。
+ Queue based load leveling
    基于队列的负载均衡：在任务和服务之间构建队列作为缓冲，来平缓间断的高峰负载压力，避免服务或任务的失败。
+ Reactor

+ Reader Writer Lock
    对共享内存的访问，每次都加锁会降低吞吐量，分离读写锁，只对写操作加锁，提高吞吐量。适合读多写少的情况。
+ Saga

+ Thread Pool
    线程池：*大量* *短生命周期*的任务，创建线程的成本高于实际任务的执行。通过复用线程解决此问题。
+ Version Number
    版本号：解决多个线程同时更新实体对象时的冲突问题
    问题：两个线程都从数据库读到一个A对象，在内存中A的属性都一致。当其中一个线程修改对象后更新回数据库时。导致另一个线程持有的数据变的不新鲜了。


#### 一、半同步/半异步（Half-Sync/Half-Async）
##### 意图
从异步I/O中解耦同步I/O，不降低执行效率的情况下简化并发程序

##### 类图
![Half-Sync/Half-Async class diagram](./etc/half-sync-half-async.png)

##### 应用
以下情况考虑使用：

* 系统有以下特征:
  * the system must perform tasks in response to external events that occur asynchronously, like hardware interrupts in OS
  * it is inefficient to dedicate separate thread of control to perform synchronous I/O for each external source of event
  * the higher level tasks in the system can be simplified significantly if I/O is performed synchronously.
* one or more tasks in a system must run in a single thread of control, while other tasks may benefit from multi-threading.

##### 现实中的例子
* [BSD Unix networking subsystem](https://www.dre.vanderbilt.edu/~schmidt/PDF/PLoP-95.pdf)
* [Real Time CORBA](http://www.omg.org/news/meetings/workshops/presentations/realtime2001/4-3_Pyarali_thread-pool.pdf)
* [Android AsyncTask framework](http://developer.android.com/reference/android/os/AsyncTask.html)

##### Credits
* [Douglas C. Schmidt and Charles D. Cranor - Half Sync/Half Async](https://www.dre.vanderbilt.edu/~schmidt/PDF/PLoP-95.pdf)
* [Pattern Oriented Software Architecture Volume 2: Patterns for Concurrent and Networked Objects](https://www.amazon.com/gp/product/0471606952/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=0471606952&linkCode=as2&tag=javadesignpat-20&linkId=889e4af72dca8261129bf14935e0f8dc)


#### 二、异步方法调用（Async Method Invocation）
##### 意图
异步方法调用是一种子任务执行时不阻塞调用者线程的设计模式。它能并行处理多个不相关的子任务，通过回调或者等待任务执行结束来获取结果。

##### 类图
![alt text](./etc/async-method-invocation.png "Async Method Invocation")

##### 应用
一下情况考虑使用异步方法调用：

* 有多个不相关的子任务可以并行执行
* 想提升一组sequential tasks的性能
* you have limited amount of processing capacity or long running tasks and the
  caller should not wait the tasks to be ready

##### 现实中的例子

* [FutureTask](http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/FutureTask.html), [CompletableFuture](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html) and [ExecutorService](http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorService.html) (Java)
* [Task-based Asynchronous Pattern](https://msdn.microsoft.com/en-us/library/hh873175.aspx) (.NET)



#### 三、止步模式（Balking）

##### 意图
避免对象在执行某行为时处于不确定、不合适的状态。

##### 类图
![alt text](./etc/balking.png "Balking")

##### 应用

* 只有在对象处于特定状态时才执行它的某一逻辑
* objects are generally only in a state that is prone to balking temporarilybut for an unknown amount of time

##### 相关模式
* 保护性挂起-Guarded Suspension Pattern
* 双重校验锁-Double Checked Locking Pattern


#### 四、Event-based Asynchronous

##### 意图
The Event-based Asynchronous Pattern makes available the advantages of multithreaded applications while hiding many
of the complex issues inherent in multithreaded design. Using a class that supports this pattern can allow you to:

1. Perform time-consuming tasks, such as downloads and database operations, "in the background," without interrupting your application.
2. Execute multiple operations simultaneously, receiving notifications when each completes.
3. Wait for resources to become available without stopping ("hanging") your application.
4. Communicate with pending asynchronous operations using the familiar events-and-delegates model.

##### 类图
![alt text](./etc/event-asynchronous.png "Event-based Asynchronous")

##### 应用
以下情况考虑使用：

* Time-consuming tasks are needed to run in the background without disrupting the current application.

##### Credits

* [Event-based Asynchronous Pattern Overview](https://msdn.microsoft.com/en-us/library/wewwczdw%28v=vs.110%29.aspx?f=255&MSPPError=-2147217396)



#### 五、Commander

##### 意图

> 用于处理在分布式事务中遇到的所有问题

##### 类图
![alt text](./etc/commander.urm.png "Commander class diagram")

##### 应用
当我们向2个以上的数据库提交事务时，可能无法保证原子性，从而导致问题。

##### Explanation
Handling distributed transactions can be tricky, but if we choose to not handle it carefully, there could be unwanted consequences. Say, we have an e-commerce website which has a Payment microservice and a Shipping microservice. If the shipping is available currently but payment service is not up, or vice versa, how would we deal with it after having already received the order from the user?
We need a mechanism in place which can handle these kinds of situations. We have to direct the order to either one of the services (in this example, shipping) and then add the order into the database of the other service (in this example, payment), since two databses cannot be updated atomically. If currently unable to do it, there should be a queue where this request can be queued, and there has to be a mechanism which allows for a failure in the queueing as well. All this needs to be done by constant retries while ensuring idempotence (even if the request is made several times, the change should only be applied once) by a commander class, to reach a state of eventual consistency.

## Credits

* [https://www.grahamlea.com/2016/08/distributed-transactions-microservices-icebergs/]


#### 六、事件队列（Event Queue）

##### 意图
对访问相机、音频等有限资源的请求，排队异步处理
当访问有限制的资源时（音频、数据库等），要处理所有的这些请求，它把所有的请求放入队列，并异步处理
Gives the resource for the event when it is the next in the queue and in same time
removes it from the queue.

##### 类图
![alt text](./etc/model.png "Event Queue")

##### 应用

* 有限的访问资源，异步访问可接受时考虑使用

## Credits

* [Mihaly Kuprivecz - Event Queue] (http://gameprogrammingpatterns.com/event-queue.html)



#### 七、保护性暂停（Guarded Suspension）

##### 意图
当你想执行对象的方法，但是该对象的状态不对时，考虑使用此模式

##### 类图
![Guarded Suspension diagram](./etc/guarded-suspension.png)

##### 应用
Use Guarded Suspension pattern when the developer knows that the method execution will be blocked for a finite period of time

##### 相关模式

* 阻行模式（Balking） 

#### 八、Promise

## Also known as

CompletableFuture

##### 意图

A Promise represents a proxy for a value not necessarily known when the promise is created. It 
allows you to associate dependent promises to an asynchronous action's eventual success value or 
failure reason. Promises are a way to write async code that still appears as though it is executing 
in a synchronous way.

## Explanation

The Promise object is used for asynchronous computations. A Promise represents an operation that 
hasn't completed yet, but is expected in the future.

Promises provide a few advantages over callback objects:
 * Functional composition and error handling.
 * Prevents callback hell and provides callback aggregation.

Real world example

> We are developing a software solution that downloads files and calculates the number of lines and 
> character frequencies in those files. Promise is an ideal solution to make the code concise and 
> easy to understand.

In plain words

> Promise is a placeholder for an asynchronous operation that is ongoing.

Wikipedia says

> In computer science, future, promise, delay, and deferred refer to constructs used for 
> synchronizing program execution in some concurrent programming languages. They describe an object 
> that acts as a proxy for a result that is initially unknown, usually because the computation of 
> its value is not yet complete.

**Programmatic Example**

In the example a file is downloaded and its line count is calculated. The calculated line count is 
then consumed and printed on console.

Let's first introduce a support class we need for implementation. Here's `PromiseSupport`.

```java
class PromiseSupport<T> implements Future<T> {

  private static final Logger LOGGER = LoggerFactory.getLogger(PromiseSupport.class);

  private static final int RUNNING = 1;
  private static final int FAILED = 2;
  private static final int COMPLETED = 3;

  private final Object lock;

  private volatile int state = RUNNING;
  private T value;
  private Exception exception;

  PromiseSupport() {
    this.lock = new Object();
  }

  void fulfill(T value) {
    this.value = value;
    this.state = COMPLETED;
    synchronized (lock) {
      lock.notifyAll();
    }
  }

  void fulfillExceptionally(Exception exception) {
    this.exception = exception;
    this.state = FAILED;
    synchronized (lock) {
      lock.notifyAll();
    }
  }

  @Override
  public boolean cancel(boolean mayInterruptIfRunning) {
    return false;
  }

  @Override
  public boolean isCancelled() {
    return false;
  }

  @Override
  public boolean isDone() {
    return state > RUNNING;
  }

  @Override
  public T get() throws InterruptedException, ExecutionException {
    synchronized (lock) {
      while (state == RUNNING) {
        lock.wait();
      }
    }
    if (state == COMPLETED) {
      return value;
    }
    throw new ExecutionException(exception);
  }

  @Override
  public T get(long timeout, TimeUnit unit) throws ExecutionException {
    synchronized (lock) {
      while (state == RUNNING) {
        try {
          lock.wait(unit.toMillis(timeout));
        } catch (InterruptedException e) {
          LOGGER.warn("Interrupted!", e);
          Thread.currentThread().interrupt();
        }
      }
    }

    if (state == COMPLETED) {
      return value;
    }
    throw new ExecutionException(exception);
  }
}
```

With `PromiseSupport` in place we can implement the actual `Promise`.

```java
public class Promise<T> extends PromiseSupport<T> {

  private Runnable fulfillmentAction;
  private Consumer<? super Throwable> exceptionHandler;

  public Promise() {
  }

  @Override
  public void fulfill(T value) {
    super.fulfill(value);
    postFulfillment();
  }

  @Override
  public void fulfillExceptionally(Exception exception) {
    super.fulfillExceptionally(exception);
    handleException(exception);
    postFulfillment();
  }

  private void handleException(Exception exception) {
    if (exceptionHandler == null) {
      return;
    }
    exceptionHandler.accept(exception);
  }

  private void postFulfillment() {
    if (fulfillmentAction == null) {
      return;
    }
    fulfillmentAction.run();
  }

  public Promise<T> fulfillInAsync(final Callable<T> task, Executor executor) {
    executor.execute(() -> {
      try {
        fulfill(task.call());
      } catch (Exception ex) {
        fulfillExceptionally(ex);
      }
    });
    return this;
  }

  public Promise<Void> thenAccept(Consumer<? super T> action) {
    var dest = new Promise<Void>();
    fulfillmentAction = new ConsumeAction(this, dest, action);
    return dest;
  }

  public Promise<T> onError(Consumer<? super Throwable> exceptionHandler) {
    this.exceptionHandler = exceptionHandler;
    return this;
  }

  public <V> Promise<V> thenApply(Function<? super T, V> func) {
    Promise<V> dest = new Promise<>();
    fulfillmentAction = new TransformAction<>(this, dest, func);
    return dest;
  }

  private class ConsumeAction implements Runnable {

    private final Promise<T> src;
    private final Promise<Void> dest;
    private final Consumer<? super T> action;

    private ConsumeAction(Promise<T> src, Promise<Void> dest, Consumer<? super T> action) {
      this.src = src;
      this.dest = dest;
      this.action = action;
    }

    @Override
    public void run() {
      try {
        action.accept(src.get());
        dest.fulfill(null);
      } catch (Throwable throwable) {
        dest.fulfillExceptionally((Exception) throwable.getCause());
      }
    }
  }

  private class TransformAction<V> implements Runnable {

    private final Promise<T> src;
    private final Promise<V> dest;
    private final Function<? super T, V> func;

    private TransformAction(Promise<T> src, Promise<V> dest, Function<? super T, V> func) {
      this.src = src;
      this.dest = dest;
      this.func = func;
    }

    @Override
    public void run() {
      try {
        dest.fulfill(func.apply(src.get()));
      } catch (Throwable throwable) {
        dest.fulfillExceptionally((Exception) throwable.getCause());
      }
    }
  }
}
```

Now we can show the full example in action. Here's how to download and count the number of lines in 
a file using `Promise`.

```java
  countLines().thenAccept(
      count -> {
        LOGGER.info("Line count is: {}", count);
        taskCompleted();
      }
  );

  private Promise<Integer> countLines() {
    return download(DEFAULT_URL).thenApply(Utility::countLines);
  }

  private Promise<String> download(String urlString) {
    return new Promise<String>()
        .fulfillInAsync(
            () -> Utility.downloadFile(urlString), executor)
        .onError(
            throwable -> {
              throwable.printStackTrace();
              taskCompleted();
            }
        );
  }
```

##### 类图

![alt text](./etc/promise.png "Promise")

##### 应用

Promise pattern is applicable in concurrent programming when some work needs to be done 
asynchronously and:

* Code maintainability and readability suffers due to callback hell.
* You need to compose promises and need better error handling for asynchronous tasks.
* You want to use functional style of programming.


##### 现实中的例子

* [java.util.concurrent.CompletableFuture](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html)
* [Guava ListenableFuture](https://github.com/google/guava/wiki/ListenableFutureExplained)

## Related Patterns

 * [Async Method Invocation](https://java-design-patterns.com/patterns/async-method-invocation/)
 * [Callback](https://java-design-patterns.com/patterns/callback/)

## Tutorials

* [Guide To CompletableFuture](https://www.baeldung.com/java-completablefuture)

## Credits

* [You are missing the point to Promises](https://gist.github.com/domenic/3889970)
* [Functional style callbacks using CompletableFuture](https://www.infoq.com/articles/Functional-Style-Callbacks-Using-CompletableFuture)
* [Java 8 in Action: Lambdas, Streams, and functional-style programming](https://www.amazon.com/gp/product/1617291994/ref=as_li_qf_asin_il_tl?ie=UTF8&tag=javadesignpat-20&creative=9325&linkCode=as2&creativeASIN=1617291994&linkId=995af46887bb7b65e6c788a23eaf7146)
* [Modern Java in Action: Lambdas, streams, functional and reactive programming](https://www.amazon.com/gp/product/1617293563/ref=as_li_qf_asin_il_tl?ie=UTF8&tag=javadesignpat-20&creative=9325&linkCode=as2&creativeASIN=1617293563&linkId=f70fe0d3e1efaff89554a6479c53759c)


#### 九、Leader/Followers

##### 意图
The Leader/Followers pattern provides a concurrency model where multiple 
threads can efficiently de-multiplex events and dispatch event handlers 
that process I/O handles shared by the threads.

##### 类图
![Leader/Followers class diagram](./etc/leader-followers.png)

##### 应用
Use Leader-Followers pattern when

* multiple threads take turns sharing a set of event sources in order to detect, de-multiplex, dispatch and process service requests that occur on the event sources.

##### 现实中的例子

* [ACE Thread Pool Reactor framework](https://www.dre.vanderbilt.edu/~schmidt/PDF/HPL.pdf)
* [JAWS](http://www.dre.vanderbilt.edu/~schmidt/PDF/PDCP.pdf)
* [Real-time CORBA](http://www.dre.vanderbilt.edu/~schmidt/PDF/RTS.pdf)

## Credits

* [Douglas C. Schmidt and Carlos O’Ryan - Leader/Followers](http://www.kircher-schwanninger.de/michael/publications/lf.pdf)


#### 十、Master-Worker
## Also known as

> Master-slave or Map-reduce

##### 意图

> Used for centralised parallel processing.

##### 类图
![alt text](./etc/master-worker-pattern.urm.png "Master-Worker pattern class diagram")

##### 应用
This pattern can be used when data can be divided into multiple parts, all of which need to go through the same computation to give a result, which need to be aggregated to get the final result.

## Explanation
In this pattern, parallel processing is performed using a system consisting of a master and some number of workers, where a master divides the work among the workers, gets the result back from them and assimilates all the results to give final result. The only communication is between the master and the worker - none of the workers communicate among one another and the user only communicates with the master to get the required job done. The master has to maintain a record of how the divided data has been distributed, how many workers have finished their work and returned a result, and the results themselves to be able to aggregate the data correctly.

## Credits

* [https://docs.gigaspaces.com/sbp/master-worker-pattern.html]
* [http://www.cs.sjsu.edu/~pearce/oom/patterns/behavioral/masterslave.htm]


#### 十一、Reactor

##### 意图
The Reactor design pattern handles service requests that are delivered concurrently to an application by one or more clients. The application can register specific handlers for processing which are called by reactor on specific events. Dispatching of event handlers is performed by an initiation dispatcher, which manages the registered event handlers. Demultiplexing of service requests is performed by a synchronous event demultiplexer.

##### 类图
![Reactor](./etc/reactor.png "Reactor")

##### 应用
Use Reactor pattern when

* A server application needs to handle concurrent service requests from multiple clients.
* A server application needs to be available for receiving requests from new clients even when handling older client requests.
* A server must maximize throughput, minimize latency and use CPU efficiently without blocking.

##### 现实中的例子

* [Spring Reactor](http://projectreactor.io/)

## Credits

* [Douglas C. Schmidt - Reactor](https://www.dre.vanderbilt.edu/~schmidt/PDF/Reactor.pdf)
* [Pattern Oriented Software Architecture Volume 2: Patterns for Concurrent and Networked Objects](https://www.amazon.com/gp/product/0471606952/ref=as_li_tl?ie=UTF8&camp=1789&creative=9325&creativeASIN=0471606952&linkCode=as2&tag=javadesignpat-20&linkId=889e4af72dca8261129bf14935e0f8dc)
* [Doug Lea - Scalable IO in Java](http://gee.cs.oswego.edu/dl/cpjslides/nio.pdf)
* [Netty](http://netty.io/)



#### 十二、线程池（Thread Pool）

##### 意图
当我们有大量的任务，但是其生命周期都很短，为每个任务创建、销毁线程会很耗性能。线程池通过重用线程来解决此问题。

##### 解释

现实中的例子

> We have a large number of relatively short tasks at hand. We need to peel huge amounts of potatoes 
> and serve mighty amount of coffee cups. Creating a new thread for each task would be a waste so we 
> establish a thread pool.       

In plain words

> Thread Pool is a concurrency pattern where threads are allocated once and reused between tasks. 

Wikipedia says

> In computer programming, a thread pool is a software design pattern for achieving concurrency of 
> execution in a computer program. Often also called a replicated workers or worker-crew model, 
> a thread pool maintains multiple threads waiting for tasks to be allocated for concurrent 
> execution by the supervising program. By maintaining a pool of threads, the model increases 
> performance and avoids latency in execution due to frequent creation and destruction of threads 
> for short-lived tasks. The number of available threads is tuned to the computing resources 
> available to the program, such as a parallel task queue after completion of execution.

**Programmatic Example**

Let's first look at our task hierarchy. We have a base class and then concrete `CoffeeMakingTask` 
and `PotatoPeelingTask`.

```java
public abstract class Task {

  private static final AtomicInteger ID_GENERATOR = new AtomicInteger();

  private final int id;
  private final int timeMs;

  public Task(final int timeMs) {
    this.id = ID_GENERATOR.incrementAndGet();
    this.timeMs = timeMs;
  }

  public int getId() {
    return id;
  }

  public int getTimeMs() {
    return timeMs;
  }

  @Override
  public String toString() {
    return String.format("id=%d timeMs=%d", id, timeMs);
  }
}

public class CoffeeMakingTask extends Task {

  private static final int TIME_PER_CUP = 100;

  public CoffeeMakingTask(int numCups) {
    super(numCups * TIME_PER_CUP);
  }

  @Override
  public String toString() {
    return String.format("%s %s", this.getClass().getSimpleName(), super.toString());
  }
}

public class PotatoPeelingTask extends Task {

  private static final int TIME_PER_POTATO = 200;

  public PotatoPeelingTask(int numPotatoes) {
    super(numPotatoes * TIME_PER_POTATO);
  }

  @Override
  public String toString() {
    return String.format("%s %s", this.getClass().getSimpleName(), super.toString());
  }
}
```

Next we present a runnable `Worker` class that the thread pool will utilize to handle all the potato 
peeling and coffee making.

```java
public class Worker implements Runnable {

  private static final Logger LOGGER = LoggerFactory.getLogger(Worker.class);

  private final Task task;

  public Worker(final Task task) {
    this.task = task;
  }

  @Override
  public void run() {
    LOGGER.info("{} processing {}", Thread.currentThread().getName(), task.toString());
    try {
      Thread.sleep(task.getTimeMs());
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
}
```

Now we are ready to show the full example in action.

```java
    LOGGER.info("Program started");

    // Create a list of tasks to be executed
    var tasks = List.of(
        new PotatoPeelingTask(3),
        new PotatoPeelingTask(6),
        new CoffeeMakingTask(2),
        new CoffeeMakingTask(6),
        new PotatoPeelingTask(4),
        new CoffeeMakingTask(2),
        new PotatoPeelingTask(4),
        new CoffeeMakingTask(9),
        new PotatoPeelingTask(3),
        new CoffeeMakingTask(2),
        new PotatoPeelingTask(4),
        new CoffeeMakingTask(2),
        new CoffeeMakingTask(7),
        new PotatoPeelingTask(4),
        new PotatoPeelingTask(5));

    // Creates a thread pool that reuses a fixed number of threads operating off a shared
    // unbounded queue. At any point, at most nThreads threads will be active processing
    // tasks. If additional tasks are submitted when all threads are active, they will wait
    // in the queue until a thread is available.
    var executor = Executors.newFixedThreadPool(3);

    // Allocate new worker for each task
    // The worker is executed when a thread becomes
    // available in the thread pool
    tasks.stream().map(Worker::new).forEach(executor::execute);
    // All tasks were executed, now shutdown
    executor.shutdown();
    while (!executor.isTerminated()) {
      Thread.yield();
    }
    LOGGER.info("Program finished");
```

##### 类图

![alt text](./etc/thread_pool_urm.png "Thread Pool")

##### 应用

以下情况考虑使用线程池

* 执行大量短小生命周期的任务

## Credits

* [Effective Java](https://www.amazon.com/gp/product/0134685997/ref=as_li_qf_asin_il_tl?ie=UTF8&tag=javadesignpat-20&creative=9325&linkCode=as2&creativeASIN=0134685997&linkId=e1b9ddd5e669591642c4f30d40cd9f6b)
* [Java Concurrency in Practice](https://www.amazon.com/gp/product/0321349601/ref=as_li_qf_asin_il_tl?ie=UTF8&tag=javadesignpat-20&creative=9325&linkCode=as2&creativeASIN=0321349601&linkId=fbedb3bad3c6cbead5afa56eea39ed59)

#### 十三、信号量-Semaphore
## Also known as
Counting Semaphore

##### 意图
创建锁来协调对一片（a pool of）资源的访问。
Only a limited number of threads, specified at the creation 
of the semaphore, can access the resources at any given time.
A semaphore which only allows one concurrent access to a resource
is called a binary semaphore.

##### 类图
![alt text](./etc/semaphore.png "Semaphore")

##### 应用
Use a Semaphore when 

* You have a pool of resources to allocate to different threads
* Concurrent access to a resource could lead to a race condition 

## Credits

* [Semaphore(programming)] (http://en.wikipedia.org/wiki/Semaphore_(programming))
* [Semaphores] (http://tutorials.jenkov.com/java-concurrency/semaphores.html)

#### 十四、Saga

## Also known as
This pattern has a similar goal with two-phase commit (XA transaction)

##### 意图
This pattern is used in distributed services to perform a group of operations atomically.
This is an analog of transaction in a database but in terms of microservices architecture this is executed 
in a distributed environment

## Explanation
A saga is a sequence of local transactions in a certain context. If one transaction fails for some reason, 
the saga executes compensating transactions(rollbacks) to undo the impact of the preceding transactions.
There are two types of Saga:

- Choreography-Based Saga. 
In this approach, there is no central orchestrator. 
Each service participating in the Saga performs their transaction and publish events. 
The other services act upon those events and perform their transactions. 
Also, they may or not publish other events based on the situation.

- Orchestration-Based Saga
In this approach, there is a Saga orchestrator that manages all the transactions and directs 
the participant services to execute local transactions based on events. 
This orchestrator can also be though of as a Saga Manager.

##### 类图
![alt text](./etc/saga.urm.png "Saga pattern class diagram")

##### 应用
Use the Saga pattern, if:

- you need to perform a group of operations related to different microservices atomically
- you need to rollback changes in different places in case of failure one of the operation
- you need to take care of data consistency in different places including different databases
- you can not use 2PC(two phase commit)

## Credits

- [Pattern: Saga](https://microservices.io/patterns/data/saga.html)
- [Saga distributed transactions pattern](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/saga/saga)



#### 十五、读写锁-Reader Writer Lock

##### 意图  

Suppose we have a shared memory area with the basic constraints detailed above. It is possible to protect the shared data behind a mutual exclusion mutex, in which case no two threads can access the data at the same time. However, this solution is suboptimal, because it is possible that a reader R1 might have the lock, and then another reader R2 requests access. It would be foolish for R2 to wait until R1 was done before starting its own read operation; instead, R2 should start right away. This is the motivation for the Reader Writer Lock pattern.

##### 类图
![alt text](./etc/reader-writer-lock.png "Reader writer lock")

##### 应用  

Application need to  increase the performance of resource synchronize for multiple thread, in particularly there are mixed read/write operations.

##### 现实中的例子

* [Java Reader Writer Lock](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/locks/ReadWriteLock.html)

## Credits

* [Readers–writer lock](https://en.wikipedia.org/wiki/Readers%E2%80%93writer_lock)
* [Readers–writers_problem](https://en.wikipedia.org/wiki/Readers%E2%80%93writers_problem)


十六、Queue based load leveling

##### 意图
Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth 
intermittent heavy loads that may otherwise cause the service to fail or the task to time out. 
This pattern can help to minimize the impact of peaks in demand on availability and responsiveness 
for both the task and the service.

##### 类图
![alt text](./etc/queue-load-leveling.gif "queue-load-leveling")

##### 应用

* This pattern is ideally suited to any type of application that uses services that may be subject to overloading.
* This pattern might not be suitable if the application expects a response from the service with minimal latency.

## Tutorials
* [Queue-Based Load Leveling Pattern](http://java-design-patterns.com/blog/queue-load-leveling/)

## Real world example

* A Microsoft Azure web role stores data by using a separate storage service. If a large number of instances of the web role run concurrently, it is possible that the storage service could be overwhelmed and be unable to respond to requests quickly enough to prevent these requests from timing out or failing. 

## Credits

* [Queue-Based Load Leveling pattern](https://docs.microsoft.com/en-us/azure/architecture/patterns/queue-based-load-leveling)


#### 十七、互斥（Mutex）

##### Also known as

* Mutual Exclusion Lock
* Binary Semaphore

##### 意图
通过锁，一次只允许一个线程访问资源

##### 类图
![alt text](./etc/mutex.png "Mutex")

##### 应用
以下情况考虑使用互斥

* You need to prevent two threads accessing a critical section at the same time
* Concurrent access to a resource could lead to a race condition 

## Credits

* [Lock (computer science)](http://en.wikipedia.org/wiki/Lock_(computer_science))
* [Semaphores](http://tutorials.jenkov.com/java-concurrency/semaphores.html)

#### 十八、生产者消费者（Producer Consumer）

##### 意图
把一个任务分成生产和消费两部分来达到解耦的目的

##### 类图
![alt text](./etc/producer-consumer.png "Producer Consumer")

##### 应用
以下情况考虑使用生产者消费者模式：

* 通过把一个work分成生产和消费两个过程来解耦系统
* Addresses the issue of different timing require to produce work or consuming work
