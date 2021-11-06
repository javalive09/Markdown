# Java2(多线程、IO流、容器、内部类、泛型)

## 多线程

### 线程和进程的关系

线程寄生在进程中。一个进程中可以有多个线程。 进程本质上是一个执行的程序。 线程是多任务处理的最小单位。 多线程比多进程消耗更少的资源。

### 定义

在单个程序中同时执行多个不同的任务。多线程的目的是为了最大限度的利用cpu资源。 当程序运行时，会自动产生一个主线程，main方法就在这个主线程中运行。

### 生命周期

![](.gitbook/assets/threadlife.jpg)

#### 状态初始：新创建了一个线程对象，并没有调用start\(\)方法之前

#### 可运行状态：调用了start\(\) 等待cpu调度

#### 运行状态：获取cpu时间片，run\(\)执行

#### 阻塞状态：暂时让出cpu时间片

* 等待阻塞：o.wait\(\)  线程进入等待队列
* 同步阻塞：锁对象被占用 线程进入锁池
* 其他阻塞：sleep\(\) 、 join\(\)、I/O请求 

### 运行流程

![](.gitbook/assets/thread_lifecycle.jpg)

### 控制

#### sleep\(\)

睡眠

#### join\(\)

加入启动它的线程，让异步变同步 调用join时会阻塞当前线程

```text
        Thread t = new Thread(() -> SystemClock.sleep(50 * 1000));
        t.start();
        t.join(20 * 1000); // 当前线程阻塞20秒
```

#### yield\(\)

让出，让给相同优先级的其他线程执行

#### notify\(\)

notify（），通知其他线程使用cpu资源，一般写在线程执行完后。 （必须在synchronized块或方法中） notify（），wait（），synchronized 是一个组合。他们一定是配合使用的，来进行线程间交互。

#### sleep和wait的区别

sleep不释放对象锁（占着cup资源的茅坑不拉屎，睡觉去了） wait 释放对象锁（交出cup资源, monitor锁资源，进入等待池） wait 使用时线程必须有对象锁。（必须在synchronized块或方法中）

#### synchronized 的几种使用场景

对象中的同步方法: 锁是对象本身。 静态的同步方法：锁是方法所在的类class对象。

#### 如何做并发加法

**atomic**

AtomicLong  
AtomicInteger incrementAndGet\(\) decrementAndGet\(\)

**java8 并发加法器**

LongAdder DoubleAdder Android N之后支持java 8

### 守护线程

Java将线程分为User线程和Daemon线程两种 1\) 所谓守护线程就是运行在程序后台的线程，程序的主线程Main（比方java程序一开始启动时创建的那个线程）不会是守护线程。

2\) Daemon thread在Java里面的定义是，如果虚拟机中只有Daemon thread 在运行，则虚拟机退出。通常Daemon线程用来为User线程提供某些服务。程序的main\(\)方法线程是一个User进程，User进程创建的进程为User进程。当所有的User线程结束后，JVM才会结束。

3\) 通过在一个线程对象上调用setDaemon\(true\)，可以将user线程创建的线程明确地设置成Daemon线程。例如，时钟处理线程、idle线程、垃圾回收线程、屏幕更新线程等，都是Daemon线程。通常新创建的线程会从创建它的进程哪里继承daemon状态，除非明确地在线程对象上调用setDaemon方法来改变daemon状态。需要注意的是，setDaemon\(\)方法必须在调用线程的start\(\)方法之前调用。一旦一个线程开始执行（如，调用了start\(\)方法），它的daemon状态不能再修改。通过方法isDaemon\(\)可以知道一个线程是否Daemon线程。

4\) 必须等所有的Non-daemon线程都运行结束了，只剩下daemon的时候，JVM才会停下来，注意Main主程序是Non-daemon线程，默认产生的线程全部是Non-daemon线程。

5\) daemon的作用 当设置线程t为Daemon线程时，只要User线程（main线程）一结束，程序立即退出，Daemon线程没有时间从10数到1。但是，如果将线程t设成非daemon，即User线程，则该线程可以完成自己的工作（从10数到1）。如下：

```text
import static java.util.concurrent.TimeUnit.*;     
public class DaemonTest {     
    public static void main(String[] args) throws InterruptedException {     
        Runnable r = new Runnable() {     
            public void run() {     
                for (int time = 10; time > 0; --time) {     
                    System.out.println("Time #" + time);     
                    try {     
                        SECONDS.sleep(2);     
                    } catch (InterruptedException e) {     
                        e.printStackTrace();     
                    }     
                }     
            }     
        };     

        Thread t = new Thread(r);     
        t.setDaemon(true);  // try to set this to "false" and see what happens     
        t.start();     

        System.out.println("Main thread waiting...");     
        SECONDS.sleep(6);     
        System.out.println("Main thread exited.");     
    }     
}
```

### Callable

有返回值的线程 调用get方法时会阻塞调用线程

应用场景： 更优雅的 用多线程方式 获取多个数据

例如 一个接口 要做从3个数据库获取3种数据并返回 可以用3个线程来优化速度（runnable也可以，不优雅） 原来需要10秒 现在只需要5秒 并且非常优雅的直接返回了数据

```text
        ExecutorService executorService = Executors.newCachedThreadPool();
        Future<String> future1 = executorService.submit(() -> {
            Log.i("ThreadTest", "future1 start");
            SystemClock.sleep(5 * 1000);
            Log.i("ThreadTest", "future1 end");
            return "result1";
        });
        Future<String> future2 = executorService.submit(() -> {
            Log.i("ThreadTest", "future2 start");
            SystemClock.sleep(2 * 1000);
            Log.i("ThreadTest", "future2 end");
            return "result2";
        });
        Future<String> future3 = executorService.submit(() -> {
            Log.i("ThreadTest", "future3 start");
            SystemClock.sleep(3 * 1000);
            Log.i("ThreadTest", "future3 end");
            return "result3";
        });

        future1.get();
        future2.get();
        future3.get();
```

```text
        ExecutorService executorService = Executors.newCachedThreadPool();
        Future<String> future = executorService.submit(() -> {
            SystemClock.sleep(20 * 1000);
            return "result";
        });
        String result = "time out";
        try {
             result = future.get(5, TimeUnit.SECONDS); // 当前线程阻塞5秒
        }catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        } catch (TimeoutException e) {
            e.printStackTrace();
        }
```

### 线程本地变量ThreadLocal

将多线程公共变量，变成线程私有变量 如: Looper 中prepare方法 new了一个looper，并将Looper变成了线程本地变量 set\(\) get\(\) 

![](.gitbook/assets/threadlocal.jpg)

### 多线程完成触发事件如何设计

#### 使用计数法：变量计数、容器计数

```text
public void startScan(ArrayList<String> paths) {  
    try {  
        for (String path : paths) {  
            scanDir(path);//多线程执行  
        }  
        while (mTaskCounter.get() > 0) {  
            Thread.sleep(100);  
        }  

        //do finish action  
    } catch (Exception ex) {  
        ex.printStackTrace();  
    }  
}
```

#### [多线程 扫描 文件 demo](https://github.com/javalive09/ScanSdCardDemo/tree/master/app/src/main/java/peter/util/scandemo)

### 线程池

#### 作用

通过复用线程控制线程数量，来节省内存开销，和提高运行速度。

#### 参数含义

```text
public ThreadPoolExecutor(int corePoolSize,  
                              int maximumPoolSize,  
                              long keepAliveTime,  
                              TimeUnit unit,  
                              BlockingQueue<Runnable> workQueue,  
                              ThreadFactory threadFactory,  
                              RejectedExecutionHandler handler)
```

* corePoolSize：核心线程数量。
* maximumPoolSize：最大线程数。
* keepAliveTime：闲置线程存活时间。
* unit：keepAliveTime的单位。
* workQueue：阻塞队列。
* threadFactory：线程工厂。
* handler：任务超过最大线程数后的策略。

#### ThreadPoolExecutor 里面定义的 4 种 handler 策略

1. AbortPolicy： 拒绝所提交的任务，并抛出RejectedExecutionException异常；
2. CallerRunsPolicy： 重试添加当前的任务，自动重复调用execute\(\) 方法，直到成功；
3. DiscardPolicy： 拒绝任务直接无声抛弃，没有异常信息；
4. DiscardOldestPolicy： 抛弃队列里面等待最久的一个线程，然后把拒绝任务加到队列；

#### 线程池执行逻辑  ThreadPoolExecutor.java  -  execute\(\)

```text
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    /*
     * Proceed in 3 steps:
     *
     * 1. If fewer than corePoolSize threads are running, try to
     * start a new thread with the given command as its first
     * task.  The call to addWorker atomically checks runState and
     * workerCount, and so prevents false alarms that would add
     * threads when it shouldn't, by returning false.
     *
     * 2. If a task can be successfully queued, then we still need
     * to double-check whether we should have added a thread
     * (because existing ones died since last checking) or that
     * the pool shut down since entry into this method. So we
     * recheck state and if necessary roll back the enqueuing if
     * stopped, or start a new thread if there are none.
     *
     * 3. If we cannot queue task, then we try to add a new
     * thread.  If it fails, we know we are shut down or saturated
     * and so reject the task.
     */
    int c = ctl.get();
	//如果线程池的线程个数少于corePoolSize则创建新线程执行当前任务
    if (workerCountOf(c) < corePoolSize) {
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
	//如果线程个数大于corePoolSize或者创建线程失败，则将任务存放在阻塞队列workQueue中
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
	//如果当前任务无法放进阻塞队列中，则创建新的线程来执行任务
    else if (!addWorker(command, false))
        reject(command);
}

```

![](.gitbook/assets/threadpool.jpg)

1. 如果当前运行的线程少于 corePoolSize，则会创建新的线程来执行新的任务；
2. 如果运行的线程个数等于或者大于 corePoolSize，则会将提交的任务存放到阻塞队列 workQueue 中；
3. 如果当前 workQueue 队列已满的话，则会创建新的线程来执行任务；
4. 如果线程个数已经超过了 maximumPoolSize，则会使用饱和策略 RejectedExecutionHandler 来进行处理。

{% hint style="info" %}
corePool不是固定的哪几个线程，核心线程是动态变化的
{% endhint %}

#### 关闭线程池

shutdown\(\)；不会立即终止线程池，而是要等所有任务缓存队列中的任务都执行完后才终止，但再也不会接受新的任务

shutdownNow\(\)；立即终止线程池，并尝试打断正在执行的任务，并且清空任务缓存队列，返回尚未执行的任务

#### 线程池容量的动态调整

setCorePoolSize\(\); 设置核心池大小 

setMaximumPoolSize\(\); 设置线程池最大能创建的线程数目大小

#### [合理配置线程池参数](https://juejin.im/post/5aeec0106fb9a07ab379574f#heading-4)

## IO流

缓冲流速度快的原因： 不用每次都频繁的和硬件设备（扇区）进行读写。和硬件设备读写最影响速度

### InputStream read方法

提供的两种都是将流读入byte\[\]中。无参Read（），byte\[0\] & 0xff 由于我们需要的是2进制内容，而不是补码，所以要进行高位清零，&0xff可以实现清零功能。返回int类型的内容。其内部也是调用有参的read方法，只不过是一个byte，一个byte的读，然后转成int 返回。

### OutputStream write方法

都是将byte\[\]、int\(里面装byte型内容\)写入流中。

> 注：byte\[\]相当于转移流的小容器，小桶 ByteArrayOutputstream（）含有可扩展的byte\[\]属性。用于装内容。流写入内存中的操作。把流变成数组。Bytearray out ByteArrayInputstream（）把数组变成流。Bytearray in 把内存作为输出方式

## 容器

容器中只能存放对象

### Collection

对应的工具类Collections，包含排序、洗牌等操作

![](.gitbook/assets/collection_pic.png)

#### List

**ArrayList**

本质是Object数组 private transient Object\[\] elementData 默认数组的长度为10 add时超过10则数组长度增加原来长度的一半 list变数组：（T\[\]）list.toArray\(newT\[list.size\(\)\]\);

**LinkedList**

本质是双向的链表

```text
private static  class  Node<E>{

        E item;

        Node<E> next;

       Node<E> prev;

}
```

add是new Node 节点，然后关联起来。

#### set

和数学上的集合的概念相同，元素不重复

**HashSet**

本质是HashMap，无序的set private transient HashMapmap ; add时用key来存值，value为一个object对象 可以用来做随机抽奖的容器。

**TreeSet**

本质是 TreeMap，有序的set 构造时可以加入comparator 参数，自定义定义排序规则 add时用key来存值，value为一个object对象

### Map

#### HashMap

key无序的map， jdk1.7: 本质是Entry数组 + 单向链表。 jdk1.8: 本质是Entry数组 + 红黑树。

```text
transient Entry[] table
static Entry <K,V> implementsMap.Entry<K,V>{

       final K key;

       V value;

       Entry<K,V> next;

       Final int hash;

}
```

默认数组长度为16 默认负载因子0.75 \(可在初始化时显式指定\) 当size &gt; 12\(16 \_0.75\) 扩容，扩容到容量的2倍（16\_2） put元素时，会先对比hash值和equals内容。判断键值是否相等。 可以用来做统计文字出现次数的容器 

![](.gitbook/assets/hashmap.jpg)

#### concurrentHashmap

segments put时候，由于锁分段技术 多个锁保证 put的效率 get时候，由于value用volatile修饰，是透明的，可以完全无锁 保证效率

![](.gitbook/assets/currenthashmap.png)

#### TreeMap

key 有序的map

### Queue 中的几个方法的区别

```text
add：当添加元素个数超过队列的限制 抛出异常
offer：当添加元素个数超过队列的限制 不抛出异常 只返回false

remove：删除队列第一个元素，当队列为空时 抛出异常
poll：删除队列第一个元素，当队列为空时 不抛出异常 只返回null

element: 查询队列第一个元素，当队列为空时 抛出异常
peek：查询队列第一个元素，当队列为空时 不抛出异常 只返回null
```

### 为什么重写类中的equals方法必须重写hashcode

将对象放到集合（set、map）时会涉及对比hashcode的情况。如果不重写调用的hashcode方法则会调用object类的hashcode方法（对象的地址），这样会造成内容相同的对象，在内存地址不同的情况下，会被存入set、map中。但是集合中要求对象中的内容不能重复的。这样就矛盾了

## 内部类

### 静态内部类（Static Inner Class）

static 修饰的内部类 只能访问外部类的静态成员变量与静态方法

```text
Outerclass.InnerClass inner = newOuterClass.InnerClass();
```

静态内部类和非静态内部类的唯一区别是： 静态内部类没有了指向外部类的引用 如果内部类不会引用到外部类的东西的话，强烈建议使用静态内部类，因为这样更节省资源，减少内部类指向外部类的引用——Effective Java

### 成员内部类（Member Inner Class）

和成员同一级别的内部类

可以访问外部类的所有成员（成员变量、方法）

```text
OuterClass.InnerClass inner = new OuterClass().newInnerClass();
```

可以加括号理解\(new OutClass\(\)\).newInnerClass\(\);

### 局部内部类（Local Inner Class）

方法中的内部类 访问外部类成员变量的方式：OuterClass.this.a; 只能访问所在方法中的final变量

### 匿名内部类（Anonymous Inner Class）

隐式的继承一个父类或实现一个借口 自动向上转型

### 引入内部类要解决的问题是

一个类可以访问另一个类的私有成员（封装性） 让Java可以实现多继承

## 泛型

类型的参数化

### 泛型方法

用返回泛型标记

```text
public static <E> void printArray(E[] inputArray) // 返回泛型（一个类型）

public static <T> T printArray(E[] inputArray) //返回泛型 （多个类型）
```

### 泛型类

用类标记

```text
public class Box<T> {

  private T t;

  public void add(T t) {
    this.t = t;
  }

  public T get() {
    return t;
  }
```

### 类型通配符

使用?代替具体的类型参数。 List&lt;?&gt; 在逻辑上是List,List 等所有List&lt;具体类型实参&gt;的父类。

```text
List<? extends Number> //类型通配符上限，上边界

List<? super Number> // 类型通配符下限，下边界
```

