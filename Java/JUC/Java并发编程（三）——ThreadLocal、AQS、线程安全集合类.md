# 一、ThreadLocal详解

好的博客：

[一针见血ThreadLocal](http://www.threadlocal.cn/)

[Java：关于 ThreadLocal 的知识来了](https://blog.csdn.net/weixin_50280576/article/details/113849450)

[由浅入深，全面解析ThreadLocal](https://blog.csdn.net/weixin_44050144/article/details/113061884)

## 1.1 ThreadLocal的使用

**ThreadLocal的作用：**

* ThreadLocal 能实现每一个线程都有自己专属的本地变量副本，不同线程之间不会相互干扰，主要解决了让每个线程绑定自己的值，通过使用get()和set()方法，获取默认值或将其值更改为当前线程所存的副本的值从而避免了线程安全问题。



**ThreadLocal的使用场景：**

* 在Java的多线程编程中，为保证多个线程对共享变量的安全访问，通常会使用synchronized来保证同一时刻只有一个线程对共享变量进行操作。**这种情况下可以将类变量放到ThreadLocal类型的对象中，使变量在每个线程中都有独立拷贝，不会出现一个线程读取变量时而被另一个线程修改的现象**。

1. 在进行对象跨层传递的时候，使用ThreadLocal可以避免多次传递，打破层次间的约束。
2. 线程间数据隔离
3. 进行事务操作，用于存储线程事务信息。
4. 数据库连接，Session会话管理。





**ThreadLocal 和Synchronized的区别：**

|  | synchronized                                      | ThreadLocal |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 原理         | 同步机制采用`以时间换空间`的方式,只提供了一份变量, 让不同的线程排队访问 | ThreadLocal采用`以空间换时间`的方式, 为每一个线程都提供了一份变量的副本, 从而实现同访问而相不干扰 |
| 侧重点 | 多个线程之间访问资源的同步 | 多线程中让每个线程之间的数据相互隔离 |



**ThreadLocal 常用的 API 介绍**

```java
/**
返回该线程局部变量在当前线程副本中的值。如果该变量对于当前线程没有值，它首先被初始化调用 initialValue 方法得到返回的值。
*/
public T get() {}
/**
返回当前线程的这个线程局部变量的“初始值”。该方法将在线程第一次使用 get 方法访问变量时被调用
除非线程之前调用了set 方法，在这种情况下，initialValue 方法将不会被线程调用。
通常，这个方法在每个线程中最多调用一次，但是在后续调用 remove 和 get 的情况下，它可能会被再次调用。
*/
protected T initialValue() {
	return null;
}
/**
删除当前线程局部变量的值。如果这个线程局部变量随后被当前线程调用了 get ，它的值将通过调用它的 initialValue 方法重新初始化，除非它的值在过渡期间被当前线程调用了 set 。这可能导致在当前线程中多次调用 initialValue 方法。
*/
public void remove() {}
/**
将当前线程的这个线程局部变量的副本设置为指定的值。大多数子类将不需要覆盖这个方法，仅仅依靠 initialValue 方法来设置线程局部变量的值
*/
public void set(T value) {}
/**
创建线程局部变量。变量的初始值是通过方法上 Supplier 的 get 方法来确定的。jdk1.8 才有的。
*/
public static <S> ThreadLocal<S> withInitial(Supplier<? extends S> supplier) {}
```





**ThreadLocal 简单使用**，下面代码为两个线程进行买票，买票之间互相不影响。

```java
class House {
    ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 0);

    public void saleHouse() {
        Integer value = threadLocal.get();
        value++;
        threadLocal.set(value);
    }
}

/**
 * 两个线程，每个线程操作自己的数据
 */
public class ThreadLocalDemo {

    public static void main(String[] args) {

        House house = new House();

        new Thread(() -> {
            try {
                for (int i = 1; i <=3; i++) {
                    house.saleHouse(); // 线程t1增加了三次
                }
                System.out.println(Thread.currentThread().getName()+"\t"+"---"+house.threadLocal.get()); //t1	---3
            }finally {
                house.threadLocal.remove();// 如果不清理自定义的 ThreadLocal 变量，可能会影响后续业务逻辑和造成内存泄露等问题
            }
        },"t1").start();

        new Thread(() -> {
            try {
                for (int i = 1; i <=2; i++) {
                    house.saleHouse(); // 线程t2增加了两次
                }
                System.out.println(Thread.currentThread().getName()+"\t"+"---"+house.threadLocal.get()); //t2	---2
            }finally {
                house.threadLocal.remove();
            }
        },"t2").start();

        System.out.println(Thread.currentThread().getName()+"\t"+"---"+house.threadLocal.get()); //main	---0
    }
}
```

打印结果：

![image-20210827003518704](https://gitee.com/jobim/blogimage/raw/master/img/20210827003518.png)

可以看到每个线程都有自己的threadLocal值



**SimpleDateFormat 线程安全的使用：**

![image-20210827110033957](https://gitee.com/jobim/blogimage/raw/master/img/20210827110034.png)

* 多线程下使用 SimpleDateFormat 有线程安全问题，如果非要使用，建议为每个线程创建独立的格式实例，如果多线程同时访问一个格式，则它必须保持外部同步。解决办法：1、将 SimpleDateFormat 定义为局部变量；2、使用 ThreadLocal ，也叫做线程本地变量或者线程本地存储；3、加锁

正确使用代码示例：（构建日期转换工具类）

```java
public class DateUtils {
    private static final ThreadLocal<SimpleDateFormat>  sdf_threadLocal =
            ThreadLocal.withInitial(()-> new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));

    /**
     * ThreadLocal可以确保每个线程都可以得到各自单独的一个SimpleDateFormat的对象，那么自然也就不存在竞争问题了。
     * @param stringDate
     * @return
     * @throws Exception
     */
    public static Date parseDateTL(String stringDate)throws Exception {
        return sdf_threadLocal.get().parse(stringDate);
    }

    public static void main(String[] args) throws Exception {
        for (int i = 1; i <=30; i++) {
            new Thread(() -> {
                try {
                    System.out.println(DateUtils.parseDateTL("2020-11-11 11:11:11"));
                } catch (Exception e) {
                    e.printStackTrace();
                }
            },String.valueOf(i)).start();
        }
    }
}
```



## 1.2 ThreadLocal实现原理





**Thread，ThreadLocal，ThreadLocalMap，Entry的关系：**

* 首先ThreadLocalMap是以数组形式存储一个个Entry键值对的，它是Thread的一个静态内部类，而Entry是ThreadLocalMap的静态内部类，Entry的key就是new出来的ThreadLocal，value就是set的值，所以一个Thread可以有多个ThreadLocal-value键值对。
* JVM内部维护了一个线程版的Map<Thread,T>(通过ThreadLocal对象的set方法，结果把ThreadLocal对象自己当做key，放进了ThreadLoalMap中),每个线程要用到这个T的时候，用当前的线程去Map里面获取，通过这样让每个线程都拥有了自己独立的变量，

![image-20210827111507166](https://gitee.com/jobim/blogimage/raw/master/img/20210827111507.png)

![threadlocal数据结构](https://gitee.com/jobim/blogimage/raw/master/img/20210827110446.png)





* **set方法分析**

  1. 首先获取当前线程，并根据当前线程获取一个Map
  2. 如果获取的Map不为空，则将参数设置到Map中（当前ThreadLocal的引用作为key）
  3. 如果Map为空，则给该线程创建 Map，并设置初始值

  ```java
  // 作用：将当前线程的这个线程局部变量的副本设置为指定的值
  public void set(T value) {
      // 拿到当前线程
      Thread t = Thread.currentThread();
      // 获取线程对应的ThreadLocalMap
      ThreadLocalMap map = getMap(t);
      // 如果map不为空，设置键值对，this代表调用这个方法的ThreadLocal对象
      if (map != null)
          map.set(this, value);
      else
          // 1）当前线程Thread 不存在ThreadLocalMap对象
          // 2）则调用createMap进行ThreadLocalMap对象的初始化
          // 3）并将 t(当前线程)和value(t对应的值)作为第一个entry存放至ThreadLocalMap中
          createMap(t, value);
  }
  
  /**
   * 获取当前线程Thread对应维护的ThreadLocalMap 
   *
   * @param  t the current thread 当前线程
   * @return the map 对应维护的ThreadLocalMap 
   */
  ThreadLocalMap getMap(Thread t) {
      return t.threadLocals;
  }
  
  /**
   *创建当前线程Thread对应维护的ThreadLocalMap 
   *
   * @param t 当前线程
   * @param firstValue 存放到map中第一个entry的值
   */
  void createMap(Thread t, T firstValue) {
      //这里的this是调用此方法的threadLocal
      t.threadLocals = new ThreadLocalMap(this, firstValue);
  }
  ```



* **get方法分析**

  1. 首先获取当前线程, 根据当前线程获取一个Map
  2. 如果获取的Map不为空，则在Map中以ThreadLocal的引用作为key来在Map中获取对应的Entry e，否则转到4
  3. 如果e不为null，则返回e.value，否则转到4
  4. Map为空或者e为空，则通过initialValue函数获取初始值value，然后用ThreadLocal的引用和value作为firstKey和firstValue创建一个新的Map

  ```java
  public T get() {
      // 获取当前线程对象
      Thread t = Thread.currentThread();
      // 获取此线程对象中维护的ThreadLocalMap对象
      ThreadLocalMap map = getMap(t);
      // 如果此map存在
      if (map != null) {
          // 以当前的ThreadLocal 为 key，调用getEntry获取对应的存储实体e
          ThreadLocalMap.Entry e = map.getEntry(this);
          // 对e进行判空 
          if (e != null) {
              @SuppressWarnings("unchecked")
              // 获取存储实体 e 对应的 value值
              // 即为我们想要的当前线程对应此ThreadLocal的值
              T result = (T)e.value;
              return result;
          }
      }
      /*
          初始化 : 有两种情况有执行当前代码
          第一种情况: map不存在，表示此线程没有维护的ThreadLocalMap对象
          第二种情况: map存在, 但是没有与当前ThreadLocal关联的entry
       */
      return setInitialValue();
  }
  
  // 初始化操作
  private T setInitialValue() {
      // 调用initialValue获取初始化的值
      // 此方法可以被子类重写, 如果不重写默认返回null
      T value = initialValue();
      // 获取当前线程对象
      Thread t = Thread.currentThread();
      // 获取此线程对象中维护的ThreadLocalMap对象
      ThreadLocalMap map = getMap(t);
      // 判断map是否存在
      if (map != null)
          // 存在则调用map.set设置此实体entry
          map.set(this, value);
      else
          // 1）当前线程Thread 不存在ThreadLocalMap对象
          // 2）则调用createMap进行ThreadLocalMap对象的初始化
          // 3）并将 t(当前线程)和value(t对应的值)作为第一个entry存放至ThreadLocalMap中
          createMap(t, value);
      // 返回设置的值value
      return value;
  }
  ```

  

* **remove方法分析**

  1. 首先获取当前线程，并根据当前线程获取一个Map
  2. 如果获取的Map不为空，则移除当前ThreadLocal对象对应的entry

  ```java
  /**
   * 删除当前线程中保存的ThreadLocal对应的实体entry
   */
  public void remove() {
      // 获取当前线程对象中维护的ThreadLocalMap对象
      ThreadLocalMap m = getMap(Thread.currentThread());
      // 如果此map存在
      if (m != null)
          // 存在则调用map.remove
          // 以当前ThreadLocal为key删除对应的实体entry
          m.remove(this);
  }
  ```

  



**ThreadLocal的原理：**

* **每一个 Thread 对象均含有一个 ThreadLocalMap 类型的成员变量 threadLocals** ，它存储本线程中所有ThreadLocal对象及其对应的值
* ThreadLocalMap 由一个个 **Entry 对象构成**，Entry 继承自 WeakReference<ThreadLocal<?>> ，一个 Entry 由 ThreadLocal 对象和 Object 构成。由此可见， Entry 的key是ThreadLocal对象，并且是一个**弱引用**。当没指向key的强引用后，该key就会被垃圾收集器回收
* 当执行**set方法**时，**ThreadLocal首先会获取当前线程对象**，然后**获取当前线程的ThreadLocalMap对象，再以当前ThreadLocal对象为key，将值存储进ThreadLocalMap对象中**。
* get方法执行过程类似。ThreadLocal首先会获取当前线程对象，然后获取当前线程的ThreadLocalMap对象。再以当前ThreadLocal对象为key，获取对应的value。
* ThreadLocalMap的键值为ThreadLocal对象，而且可以有多个threadLocal变量，因此保存在map中
* ThreadLocal本身并不存储值，它只是**作为一个key来让线程从ThreadLocalMap获取value**
* 由于**每一条线程均含有各自私有的ThreadLocalMap容器**，这些容器相互独立互不影响，因此不会存在线程安全性问题，从而也无需使用同步机制来保证多条线程访问容器的互斥性。



## 1.3 ThreadLocal内存泄漏

> 好的博客：[ThreadLocal 内存泄露问题](https://blog.csdn.net/JH39456194/article/details/107304997)



**什么是内存泄漏？**

* Memory Leak 程序中**已经动态分配的堆内存**由于某种原因, **程序未释放或者无法释放, 造成系统内部的浪费**, 导致程序运行速度减缓甚至系统崩溃等严重结果. **内存泄漏的堆积终将导致内存溢出** 
* 即：不再会被使用的对象或者变量占用的内存不能被回收，就是内存泄露。



### 1.3.1 强、软、弱、虚、四大引用

1. **强引用（StrongReference）**

   强引用是使用最普遍的引用。如果一个对象具有强引用，那垃圾回收器绝不会回收它。

   ```java
   Object o=new Object();   //  强引用
   o=null;     // 帮助垃圾收集器回收此对象
   ```

2. **软引用（SoftReference）**

   如果一个对象只具有软引用，则**内存空间足够，垃圾回收器就不会回收它**；如果**内存空间不足了，就会回收这些对象的内存**。只要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。

   ```java
   //当我们内存不够用的时候，soft会被回收的情况
   SoftReference<MyObject> softReference = new SoftReference<>(new Object());
   ```

3. **弱引用（WeakReference）**

   对于只有弱引用的对象来说，只要垃圾回收机制一运行，不管JVM的内存空间是否足够，都会回收该对象占用的内存。 

   ```java
   //垃圾回收机制一运行,会回收该对象占用的内存
   WeakReference<MyObject> weakReference = new WeakReference<>(new Object());
   ```

4. **虚引用（PhantomReference）**

   顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收，它不能单独使用也不能通过它访问对象。

   虚引用必须和引用队列 (ReferenceQueue)联合使用,当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。

   ```java
   ReferenceQueue<MyObject> referenceQueue = new ReferenceQueue();
   //和引用队列进行关联，当虚引用对象被回收后，会进入ReferenceQueue队列中
   PhantomReference<MyObject> phantomReference = new PhantomReference<>(new MyObject(),referenceQueue);
   ```

   

### 1.3.2 原因分析



 **ThreadLocal内存泄露的原因：**

 `ThreadLocalMap` 中使用的 key 为 `ThreadLocal` 的弱引用，而 value 是强引用。

![image-20210827151629361](https://gitee.com/jobim/blogimage/raw/master/img/20210827151629.png)

所以，如果 `ThreadLocal` 没有被外部强引用的情况下，在垃圾回收的时候，key 会被清理掉，而 value 不会被清理掉。这样一来，`ThreadLocalMap` 中就会出现 key 为 null 的 Entry。假如我们不做任何措施的话，value 永远无法被 GC 回收，这个时候就可能会产生内存泄露。

![ThreadLocal内存泄漏](https://gitee.com/jobim/blogimage/raw/master/img/20210827114535.png)



* 假设**ThreadLocalMap中的key使用了强引用**, 那么会出现内存泄漏吗?

  ![image-20210827152729885](https://gitee.com/jobim/blogimage/raw/master/img/20210827152730.png)

  ThreadLocalMap中的`key使用了强引用, 是无法完全避免内存泄漏的`



因此，**ThreadLocal内存泄漏的根源**是：

1. **没有手动删除这个 Entry**
2. **CurrentThread当前线程依然运行**,由于**ThreadLocalMap的生命周期跟Thread一样长**，如果没有手动删除对应key就会导致内存泄漏，而不是因为弱引用。



**那么为什么 key 要用弱引用呢？**

* 事实上，在 `ThreadLocalMap中的set/getEntry` 方法中，会`对 key 为 null`（也即是 ThreadLocal 为 null ）进行判断，**如果为 null 的话，那么会把 value 置为 null 的。**

* 这就意味着使用完 ThreadLocal , CurrentThread 依然运行的前提下．就算忘记调用 remove 方法，弱引用比强引用可以多一层保障：弱引用的 ThreadLocal 会被回收．对应value在下一次 ThreadLocaIMap 调用 set/get/remove 中的任一方法的时候会被清除，从而避免内存泄漏．





# 二、AQS详解

好的博客：

[Lock简介与初识AQS](https://blog.csdn.net/ThinkWon/article/details/102468837)

[AQS(AbstractQueuedSynchronizer)详解与源码分析](https://blog.csdn.net/ThinkWon/article/details/102469112)



## 2.1 AQS介绍

**什么是AQS？**

* AQS，全称AbstractQueuedSynchronizer（队列同步器），位于java.util.concurrent.locks包下。
* 是JDK1.5提供的一套用于实现阻塞锁和一系列依赖FIFO等待队列的同步器(First Input First Output先进先出)的框架实现。是除了java自带的synchronized 关键字之外的锁机制。 可以将AQS作为一个队列来理解。
* CAS+LockSupport的park()和unpark()+CLH队列（一种FIFO的双向队列）
* 我们常用的ReentrantLock、Semaphore、CountDownLatch、CyclicBarrier等并发类均是基于AQS来实现的。AQS的设计是基于模板方法模式，通过继承AQS，并实现其模板方法，来达到同步状态的管理。


AQS的功能在使用中可以分为两种:**独占锁和共享锁**

* 独占锁：每次只能有一个线程持有锁。eg: ReentrantLock就是独占锁
* 共享锁：允许多个线程同时获得锁，并发访问共享资源。eg: ReentrantReadWriteLock中的读锁、CountDownL.atch



**`AQS核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态`。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用CLH队列实现的，即将暂时获取不到锁的线程加入到队列中。**

![image-20210827161000128](https://gitee.com/jobim/blogimage/raw/master/img/20210827161000.png)



<hr/>



AQS设计是基于**模板方法模式**的，一般的使用方式是：

　　**1.使用者继承AbstractQueuedSynchronizer并重写指定的方法。（这些重写方法很简单，无非是对于共享资源state的获取和释放）**

　　**2.将AQS组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法。**



我们来看看AQS定义的这些可重写的方法：

* **protected boolean tryAcquire(int arg) **: 独占式获取同步状态，试着获取，成功返回true，反之为false
* **protected boolean tryRelease(int arg)** ：独占式释放同步状态，等待中的其他线程此时将有机会获取到同步状态；
* **protected int tryAcquireShared(int arg)** ：共享式获取同步状态，返回值大于等于0，代表获取成功；反之获取失败；
* **protected boolean tryReleaseShared(int arg)** ：共享式释放同步状态，成功为true，失败为false
* **protected boolean isHeldExclusively() **： 是否在独占模式下被线程占用。



AQS提供的模板方法：

* 独占锁

  ```java
  void acquire(int arg);// 独占式获取同步状态，如果获取失败则插入同步队列进行等待；
  void acquireInterruptibly(int arg);// 与acquire方法相同，但在同步队列中进行等待的时候可以检测中断；
  boolean tryAcquireNanos(int arg, long nanosTimeout);// 在acquireInterruptibly基础上增加了超时等待功能，在超时时间内没有获得同步状态返回false;
  boolean release(int arg);// 释放同步状态，该方法会唤醒在同步队列中的下一个节点
  ```

* 共享锁

  ```java
  void acquireShared(int arg);// 共享式获取同步状态，与独占式的区别在于同一时刻有多个线程获取同步状态；
  void acquireSharedInterruptibly(int arg);// 在acquireShared方法基础上增加了能响应中断的功能；
  boolean tryAcquireSharedNanos(int arg, long nanosTimeout);// 在acquireSharedInterruptibly基础上增加了超时等待的功能；
  boolean releaseShared(int arg);// 共享式释放同步状态
  ```

  

**使用总结：**

* 首先，我们需要去继承AbstractQueuedSynchronizer这个类，然后我们根据我们的需求去重写相应的方法，比如要实现一个独占锁，那就去重写tryAcquire，tryRelease方法，要实现共享锁，就去重写tryAcquireShared，tryReleaseShared；
* 然后，在我们的组件中调用AQS中的模板方法就可以了，而这些模板方法是会调用到我们之前重写的那些方法的。也就是说，我们只需要很小的工作量就可以实现自己的同步组件，重写的那些方法，仅仅是一些简单的对于共享资源state的获取和释放操作，至于像是获取资源失败，线程需要阻塞之类的操作，自然是AQS帮我们完成了。



## 2.2 AQS源码分析



**AQS的基本实现：**

* AQS维护一个共享资源state，通过内置的FIFO来完成获取资源线程的排队工作。（这个内置的同步队列称为"CLH"队列）。该队列由一个一个的Node结点组成，每个Node结点维护一个prev引用和next引用，分别指向自己的前驱和后继结点。AQS维护两个指针，分别指向队列头部head和尾部tail。

  ![image-20210827163123520](https://gitee.com/jobim/blogimage/raw/master/img/20210827163123.png)



* 其实就是个**双端双向链表**。

* 当线程获取资源失败（比如tryAcquire时试图设置state状态失败），会被构造成一个结点加入CLH队列中，同时当前线程会被阻塞在队列中（通过LockSupport.park实现，其实是等待态）。当持有同步状态的线程释放同步状态时，会唤醒后继结点，然后此结点线程继续加入到对同步状态的争夺中。





**AQS内部结构代码：**

```java
static final class Node {} //

private transient volatile Node head;

private transient volatile Node tail;

private volatile int state; // 同步状态
```





**Node节点结构：**

```java
static final class Node {
    // 表示线程已被取消（等待超时或者被中断）
    static final int CANCELLED =  1;
    // 表示线程已经准备好了，就等资源释放了
    static final int SIGNAL  = -1;
    // 表示节点在等待队列中，节点线程等待唤醒
    static final int CONDITION = -2;
    // 表示下一次共享式同步状态会被无条件地传播下去 
    static final int PROPAGATE = -3;
    // Node初始化的时候的默认值
    volatile int waitStatus;
    /**当前结点的前驱结点 */
    volatile Node prev;
    /** 当前结点的后继结点 */
    volatile Node next;
    /** 与当前结点关联的排队中的线程 */
    volatile Thread thread;
    /** ...... */
}
```





**`以ReentrantLock中非公平锁分析AQS的执行过程`**

* 分析代码

  ```java
  public class AQSDemo {
      public static void main(String[] args) {
          
          ReentrantLock lock = new ReentrantLock();
          
          new Thread(()-> {
              lock.lock(); //线程A获取锁
              try {
                  System.out.println(Thread.currentThread().getName()+"----lock");
                  Thread.sleep(3000); //占据了锁3秒钟
              } catch (InterruptedException e) {
                  e.printStackTrace();
              } finally {
                  System.out.println(Thread.currentThread().getName()+"----unlock");
                  lock.unlock();
              }
          },"ThreadA").start();
  
  
          new Thread(()-> {
              lock.lock(); //线程B争抢锁
              try {
                  System.out.println(Thread.currentThread().getName()+"----lock");
              } catch (Exception e) {
                  e.printStackTrace();
              } finally {
                  System.out.println(Thread.currentThread().getName()+"----unlock");
                  lock.unlock();
              }
          },"ThreadB").start();
  
          new Thread(()-> {
              lock.lock(); //线程C争抢锁
              try {
                  System.out.println(Thread.currentThread().getName()+"----lock");
              } catch (Exception e) {
                  e.printStackTrace();
              } finally {
                  System.out.println(Thread.currentThread().getName()+"----unlock");
                  lock.unlock();
              }
          },"ThreadC").start();
      }
  }
  ```

* 执行结果：

  ![image-20210827174124881](https://gitee.com/jobim/blogimage/raw/master/img/20210827174124.png)





<hr/>



**lock()函数**

```java
//ReentrantLock的lock方法
public void lock() {
    sync.lock();
}

//NonfairSync中的lock()函数
final void lock() {
    // 使用CAS设置将state的值设置为1，这也是获取锁的过程，只有state为0的时候才可以设置成功，设置成功，也就相当于当前线程获取锁成功。
    // 在ReentrantLock中，state来标识当前锁的状态。state = 0：锁没有被其他线程持有。state > 0，锁被其他线程持有，state的数量代表了重入的次数。
    if (compareAndSetState(0, 1))
        //当前线程获取锁成功后，将owner设置为当前线程
        setExclusiveOwnerThread(Thread.currentThread());
    else
        //如果获取锁失败，说明当前锁已经被其他线程占用
        acquire(1);
}
```



**调用AQS的acquire()方法，独占锁的获取：**

1. 再次调用tryAcquire去获取锁。
2. tryAcquire获取锁失败，则调用addWaiter将当前线程加入等待队列，并返回当前线程节点。
3. 调用acquireQueued将队列中的当前线程挂起。

![image-20210827195417452](https://gitee.com/jobim/blogimage/raw/master/img/20210827195417.png)

```java
public final void acquire(int arg) {
    //再次尝试同步状态是否获取成功，如果成功则方法结束返回
    //若失败则先调用addWaiter()方法再调用acquireQueued()方法
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

* **tryAcquire()方法尝试去获得锁**

  ```java
  //NonfairSync
  protected final boolean tryAcquire(int acquires) {
      return nonfairTryAcquire(acquires);
  }
  
  //执行Sync的nonfairTryAcquire方法
  final boolean nonfairTryAcquire(int acquires) {
  	//拿到当前线程
      final Thread current = Thread.currentThread();
      //拿到当前的state
      int c = getState();
      //只有当c == 0的情况下，才说明当前锁没有被占用，才进行CAS尝试替换。这里提前判断，为了提升性能，防止每次都进行CAS操作
      if (c == 0) {
          if (compareAndSetState(0, acquires)) {
              setExclusiveOwnerThread(current);
              return true;
          }
      }
      //执行到这儿，说明当前锁已经被占用了
      //则判断占用锁的线程是否是当前线程，如果是，则就是重入
      else if (current == getExclusiveOwnerThread()) {
          int nextc = c + acquires;
          if (nextc < 0) // overflow
              throw new Error("Maximum lock count exceeded");
          setState(nextc);
          return true;
      }
      return false;
  }
  ```

  

* **当线程获取独占式锁失败后就会将当前线程加入同步队列，并将该节点挂起**

  * **addWaiter()方法，将当前线程加入同步队列**

    ```java
    private Node addWaiter(Node mode) {
    	//用当前线程创建一个新的Node节点
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        Node pred = tail;
        //这里判断tail不等于null。说明队列中已经有等待的线程了，直接尝试将当前线程往队列末尾追加
        if (pred != null) {
        	//将当前节点尾插入的方式插入同步队列中
            node.prev = pred;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        //当前同步队列尾节点为null，说明当前线程是第一个加入同步队列进行等待的线程
       enq(node);
       return node;
    }
    ```

    **enq()方法，处理当前同步队列尾节点为null时进行入队操作，即第一个加入同步队列进行等待的线程**

    ```java
    private Node enq(final Node node) {
        //这里是一个死循环，之所以使用死循环，因为在将当前线程加入队列的时候，
        //可能会因为其他现成提前加入成功了导致CAS失败，此时会继续再次循环尝试加入，直到加入成功为止。
        for (;;) {
            Node t = tail;
            //这里首先判断如果队列中没有等待线程，就直接初始化一个Node
            //并且将head和tail都指向初始化的node。然后再执行二次循环将当前线程加入队列。
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                //否则就尝试通过CAS将当前现成加入队列，如果加入失败，则继续循环尝试加入，知道成功返回为止。
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }
    ```

  * **执行acquireQueued()方法将节点线程挂起**

    ```java
    //执行到这个方法，说明当前线程已经加入到了等待队列中。
    //这个方法要做的事情，就是将当前线程挂起。
    final boolean acquireQueued(final Node node, int arg) {
            boolean failed = true;
            try {
                boolean interrupted = false;
                //这里是一个死循环，只有获得了锁的线程，才能退出。
                //挂起的线程被唤醒后，会重新进行锁的抢占，抢占失败，则继续被挂起.再非公平锁中，新来的线程可以抢占锁资源。
                //被挂起的线程可能被中断唤醒。中断唤醒后，因为不能获取锁。所以会再次被挂起。
                for (;;) {
                	//获取当前线程节点的前一个节点，只有该节点为第二个节点才有机会获取锁
                    final Node p = node.predecessor();
                    //这里p == head，为了保证线程节点从队列上的唤醒顺序必须是从前到后按顺序依次释放
                    if (p == head && tryAcquire(arg)) {
                    	//执行到这里，有两种情况：
                    	//1、在挂起前尝试获取锁成功了。
                    	//2、锁被释放，从队列上将线程唤醒后，线程获取锁成功。
                    	//如果当前线程获取了锁，则将当前线程节点从队列中移除。
                        setHead(node);
                        p.next = null; // help GC
                        failed = false;
                        return interrupted;
                    }
                    //这里开始真正挂起当前线程，先通过shouldParkAfterFailedAcquire判断是否可以挂起线程，再通过parkAndCheckInterrupt挂起当前线程
                    if (shouldParkAfterFailedAcquire(p, node) &&
                        parkAndCheckInterrupt())
                        //parkAndCheckInterrupt返回值是是否中断唤醒，如果是，则interrupted赋值为true，当当前线程获取锁的时候，需要返回interrupted，并且再外层处理中断
                        interrupted = true;
                }
            } finally {
                if (failed)
                    cancelAcquire(node);
            }
        }
    }
    ```

    **shouldParkAfterFailedAcquire方法用于将前驱节点设置为SIGNAL状态，用户后续唤醒操作**

    ```java
    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    	//获取当前节点的前一个节点的状态，通过前一个节点的waitStatus判断是否挂起当前线程
        int ws = pred.waitStatus;
        //如果前一个节点的waitStatus == GIGNAL，则挂起
        if (ws == Node.SIGNAL)
            return true;
        if (ws > 0) {
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            // 将当前节点的前驱节点设置为SIGNAL状态，用户后续唤醒操作
            // 程序第一次执行到这返回false，还会进行外层第二次循环，最终从第6行返回true
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }
    ```

    如果shouldParkAfterFailedAcquire返回为true，说明前一个节点的waitstatus已经是SIGNAL了。此时**执行parkAndCheckInterrupt将当前线程挂起**

    ```java
    private final boolean parkAndCheckInterrupt() {
    	//将当前线程挂起
       LockSupport.park(this);
       //挂起后，可以因为中断将线程唤醒，所以唤醒后直接判断是否是中断唤醒，interrupted返回中断状态，并将线程的中断标志重置。
       return Thread.interrupted();
    }
    ```

当ThreadA、ThreadB、ThreadC执行完lock方法后的队列为：

![image-20210827201203813](https://gitee.com/jobim/blogimage/raw/master/img/20210827201203.png)



**总结：**

1. 调用lock()方法，首先tryAcquire尝试获取同步状态，成功则直接返回（ 修改state同步状态属性并设置当前对象所占用的线程）；否则，进入下一环节；

2. 线程获取同步状态失败，**就构造一个结点，加入同步队列中**，这个过程要保证线程安全（通过CAS）；

3. 加入队列中的结点线程进入自旋状态，若是老二结点（即前驱结点为头结点），才有机会尝试去获取同步状态；否则，**当其前驱结点的状态为SIGNAL，线程进入阻塞状态**，直到被中断或者被前驱结点唤醒。



<hr/>

**unlock()函数**

```java
// ReentrantLock中unlock()方法
public void unlock() {
    sync.release(1);
}

// Sync的release()方法
public final boolean release(int arg) {
    //释放锁，即对state做减法操作，直到state == 0，表示释放成功。
    if (tryRelease(arg)) {
        //执行到这儿，说已经释放锁，所以需要唤醒线程。
        Node h = head;
        //这里判断h.waitStatus != 0，0是默认状态。在挂起线程的时候，判断了前一个节点是否是SIGNAL == -1，
        // 如果是0，则会将其改为SIGNAL。而唤醒线程的时候，会将 h.waitStatus修改为0
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```

**tryRelease()方法，独占锁的释放**

```java
//这个方法很简单，就是将state的状态值减去releases并重新赋值。
//当最终的state == 0返回true，标识锁释放成功。如果是重入锁，需要多次释放。
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    if (c == 0) {
        free = true;
        setExclusiveOwnerThread(null);
    }
    setState(c); //设置state为0
    return free;
}
```

**unparkSuccessor()方法进行进行队列的线程唤醒**，当head指向的头结点不为null，并且该节点的状态值不为0的话才会执行unparkSuccessor()方法

```java
private void unparkSuccessor(Node node) {

    // 如果头结点的waitStatus < 0，则将其赋值为0
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    //头节点的后继节点
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        //后继节点不为null时唤醒该线程
        LockSupport.unpark(s.thread);
}
```

* 当ThreadA释放锁后，ThreadB获得锁后的队列状态：

  ![image-20210827202836349](https://gitee.com/jobim/blogimage/raw/master/img/20210827202836.png)

* 释放锁的时候会唤醒头结点后继节点，但由于是非公平锁，此时如果有新的线程争抢锁也有可能获得锁。

* 如果队列中的节点如果获取了锁，会**把之前的哨兵节点移除队列，而该节点会作为哨兵节点**。



流程总结：

![image-20210828105203156](https://gitee.com/jobim/blogimage/raw/master/img/20210828105203.png)



## 2.3 ReentrantLock



**ReentrantLock的特点**

* 可重入：指同一个线程如果首次获得了这把锁，那么因为它是这把锁的拥有者，因此有权利再次获取这把锁
* 可打断：使用lockInterruptibly()进行加锁可打断
* 支持公平锁
* 支持多个条件变量
* 可超时的获取锁：通过tryLock(long timeout, TimeUnit unit)在一段时间内尝试获取锁



**ReentrantLock的使用**

```java
public void test(){
    private Lock lock = new ReentrantLock();
    lock.lock(); // 上锁操作
    try{
        doSomeThing();
    }catch (Exception e){
        // ignored
    }finally {
        lock.unlock(); //必须在finally中释放锁
    }
}
```





**ReentrantLock的内部结构：**

* `ReentrantLock`底层基于`AbstractQueuedSynchronizer`实现的（AQS上面已讲）。
* ReentrantLock**实现了Lock接口**。其内部定义了专门的组件Sync， Sync继承`AbstractQueuedSynchronizer`提供释放资源的实现，**NonfairSync和FairSync是基于Sync扩展的子类**，即ReentrantLock的**非公平模式与公平模式**![image-20210828104948203](https://gitee.com/jobim/blogimage/raw/master/img/20210828104948.png)



**ReentrantLock的常用方法：**

```java
// 获取锁
void lock();

//获取锁-响应中断 
void lockInterruptibly() throws InterruptedException;

// 返回获取锁是否成功状态
boolean tryLock();

// 返回获取锁是否成功状态，可超时的获取锁-响应中断 
boolean tryLock(long time, TimeUnit unit) throws InterruptedException;

// 释放锁
void unlock();

// 创建条件变量
Condition newCondition();
```





**公平锁和公平锁：**

* 非公平锁底层使用的是`NonfairSync`，公平锁底层调用的是`FairSync`

* 两者的流程基本一致，唯一的区别是`CAS `执行前，多了一步`hasQueuedPredecessors`函数判断

  ![image-20210828112731955](https://gitee.com/jobim/blogimage/raw/master/img/20210828112732.png)





**Condition条件变量：**

* Condition类能实现synchronized和wait、notify搭配的功能，另外比后者更灵活，Condition可以实现多路通知功能，也就是在一个Lock对象里可以创建多个Condition（即对象监视器）实例，线程对象可以注册在指定的Condition中，从而**可以有选择的进行线程通知，在调度线程上更加灵活**。

* API介绍：

  ```java
  // 造成当前线程在接到信号或被中断之前一直处于等待状态。  
  void await()
  
  // 造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。  
  boolean await(long time, TimeUnit unit)
  
  // 造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。  
  long awaitNanos(long nanosTimeout)
  
  // 造成当前线程在接到信号之前一直处于等待状态。  
  void awaitUninterruptibly()
  
  // 造成当前线程在接到信号、被中断或到达指定最后期限之前一直处于等待状态。  
  boolean awaitUntil(Date deadline)
  
  // 唤醒一个等待线程。  
  void signal()
  
  // 唤醒所有等待线程。  
  void signalAll()
  ```

* 代码示例：

  ```java
  @Slf4j(topic = "c.TestCondition")
  public class Test01 {
  
      static ReentrantLock lock = new ReentrantLock();
      static Condition waitCigaretteQueue = lock.newCondition();
      static Condition waitBreakFastQueue = lock.newCondition();
      static volatile boolean hasCigarette = false;
      static volatile boolean hasBreakfast = false;
  
      public static void main(String[] args) {
          new Thread(() -> {
              try {
                  lock.lock();
                  while (!hasCigarette) {
                      try {
                          waitCigaretteQueue.await();
                      } catch (InterruptedException e) {
                          e.printStackTrace();
                      }
                  }
                  log.debug("等到了它的烟");
              } finally {
                  lock.unlock();
              }
          }).start();
  
          new Thread(() -> {
              try {
                  lock.lock();
                  while (!hasBreakfast) {
                      try {
                          waitBreakFastQueue.await();
                      } catch (InterruptedException e) {
                          e.printStackTrace();
                      }
                  }
                  log.debug("等到了它的早餐");
              } finally {
                  lock.unlock();
              }
          }).start();
          sleep(1);
          sendBreakfast();
          sleep(1);
          sendCigarette();
      }
  
      private static void sendCigarette() {
          lock.lock();
          try {
              log.debug("送烟来了");
              hasCigarette = true;
              waitCigaretteQueue.signal();
          } finally {
              lock.unlock();
          }
      }
  
      private static void sendBreakfast() {
          lock.lock();
          try {
              log.debug("送早餐来了");
              hasBreakfast = true;
              waitBreakFastQueue.signal();
          } finally {
              lock.unlock();
          }
      }
  }
  ```

  执行结果：通过Condition实现了唤醒指定条件的线程

  ![image-20210828115849654](https://gitee.com/jobim/blogimage/raw/master/img/20210828115849.png)



**Synchronized和ReentrantLock的区别？**

| 类别       | synchronized                                                 | ReentrantLock                                                |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 底层实现   | synchronized 的实现涉及到锁的升级，具体为无锁、偏向锁、自旋锁、向OS申请重量级锁,是**JVM**层面的锁，是**Java关键字**，通过monitor对象来完成（monitorenter与monitorexit）。 | ReentrantLock是一个类，底层基于AQS，通过CAS+CLH队列+LockSupport的park()和unpark() |
| 锁的释放   | synchronized 不需要用户去手动释放锁，synchronized 代码执行完后系统会自动让线程释放对锁的占用 | ReentrantLock则需要用户去手动释放锁，如果没有手动释放锁，就可能导致死锁现象。一般通过lock()和unlock()方法配合try/finally语句块来完成 |
| 是否可中断 | synchronized是不可中断类型的锁，除非加锁的代码中出现异常或正常执行完成 | ReentrantLock则可以中断，可通过trylock(long timeout,TimeUnit unit)设置超时方法或者将lockInterruptibly()放到代码块中，调用interrupt方法进行中断 |
| 是否公平锁 | synchronized为非公平锁                                       | ReentrantLock则即可以选公平锁也可以选非公平锁。为空默认false非公平锁，true为公平锁 |
| 锁类型     | 可重入 不可中断 非公平<br/>不可中断，除非抛出异常或者正常运行完成。 | 可重入  可中断  可公平（两者皆可）<br/>可通过 lock.lockInterruptibly()进行中断 |
| 线程调度   | synchronized通过Object类的wait()/notify()/notifyAll()方法要么随机唤醒一个线程要么唤醒全部线程 | ReentrantLock通过绑定Condition结合await()/singal()方法实现线程的精确唤醒 |



## 2.4 ReentrantReadWriteLock读写锁

* 读写锁指一个资源能够被**多个读线程访问**，或者被**一个写线程访问**，但是**不能同时存在读写线程**

* ReentrantReadWriteLock中有两个静态内部类：**ReadLock读锁和WriteLock写锁**，这两个锁实现了Lock接口。ReentrantReadWriteLock支持可重入，同步功能依赖自定义同步器（AbstractQueuedSynchronizer）实现，读写状态就是其同步器的同步状态



**写锁的获取和释放：**

* 写锁WriteLock是支持重进入的排他锁。如果当前线程已经获取了写锁，则增加写状态。如果当前线程在获取读锁时，读锁已经被获取或者该线程不是已获取写锁的线程，则当前线程进入等待状态。读写锁确保写锁的操作对读锁可见。写锁释放每次减少写状态，当前写状态为0时表示写锁已被释放。



**读锁的获取与释放：**

* **读锁ReadLock是支持重进入的共享锁（共享锁为shared节点，对于shared节点会进行一连串的唤醒，直到遇到一个读节点）**，它能够被多个线程同时获取，在没有其他写线程访问（写状态为0）时，**读锁总是能够被成功地获取，而所做的也只是增加读状态**（线程安全）。如果当前线程已经获取了读锁，则增加读状态。如果当前线程在获取读锁时，写锁已经被获取，则进入等待状态。



**锁降级：**

* 锁降级指的是写锁降级成为读锁。锁降级是指**当前拥有的写锁的同时，再获取到读锁，随后释放写锁的过程**

* 官网锁降级案例：

  ```java
  class CachedData {
      Object data;
      // 是否有效，如果失效，需要重新计算 data
      volatile boolean cacheValid;
      final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
      void processCachedData() {
          rwl.readLock().lock();
          if (!cacheValid) {
              // 获取写锁前必须释放读锁
              rwl.readLock().unlock();
              rwl.writeLock().lock();
              try {
                  // 判断是否有其它线程已经获取了写锁、更新了缓存, 避免重复更新
                  if (!cacheValid) {
                      data = ...
                      cacheValid = true;
                  }
                  // 降级为读锁, 释放写锁, 这样能够让其它线程读取缓存
                  rwl.readLock().lock();
              } finally {
                  rwl.writeLock().unlock();
              }
          }
          // 自己用完数据, 释放读锁 
          try {
              use(data);
          } finally {
              rwl.readLock().unlock();
          }
      }
  }
  ```

  

**读写锁代码示例：**

```java
@Slf4j(topic = "c.TestReadWriteLock")
public class TestReadWriteLock {

    public static void main(String[] args) throws InterruptedException {
        DataContainer dataContainer = new DataContainer();

        new Thread(() -> {
            dataContainer.write();
        }, "t1").start();

        new Thread(() -> {
            dataContainer.read();
        }, "t2").start();

        new Thread(() -> {
            dataContainer.read();
        }, "t3").start();


    }
}

@Slf4j(topic = "c.DataContainer")
class DataContainer {
    private Object data;
    private ReentrantReadWriteLock rw = new ReentrantReadWriteLock();
    private ReentrantReadWriteLock.ReadLock r = rw.readLock();
    private ReentrantReadWriteLock.WriteLock w = rw.writeLock();

    public Object read() {
        r.lock();
        log.debug("获取读锁...");
        try {
            log.debug("读取");
            sleep(1);
            return data;
        } finally {
            log.debug("释放读锁...");
            r.unlock();
        }
    }

    public void write() {
        w.lock();
        log.debug("获取写锁...");
        try {
            log.debug("写入");
            sleep(1);
        } finally {
            log.debug("释放写锁...");
            w.unlock();
        }
    }
}
```

执行结果：

![image-20210828150545167](https://gitee.com/jobim/blogimage/raw/master/img/20210828150545.png)





JDK 8之后加入了**StampedLock（邮戳锁）**，StampedLock 支持 tryOptimisticRead() 方法（乐观读），读取完毕后需要做一次 戳校验 如果校验通过，表示这期间确实没有写操作，数据可以安全使用，如果校验没通过，需要重新获取读锁，保证数据安全。**StampedLock采取乐观获取锁后，其他线程尝试获取写锁时不会被阻塞（可以解决读写锁，锁饥饿问题）**



## 2.5 Semaphore

* `Semaphore`又称"信号量"，也是一个非常有用的工具类，它相当于是一个并发控制器，**限制可同时访问某一资源或资源池的线程数**。Semaphore 内部维护了一组虚拟的许可，许可的数量可以通过构造函数的参数指定。访问特定资源前，必须使用**acquire()方法获得许可**，如果许可数量为0，该线程则一直阻塞，直到有可用许可。访问资源后，使用**release()方法释放许**可。

* `Semaphore(int permits,boolean fair)`提供了2个参数。permits 代表资源池的长度；fair 代表 公平许可 或 非公平许可。

* 原理图：线程1、线程2、线程3、线程4、分别调用semaphore.acquire()，令变数为3，整个过程队列信息变化如下图：

  ![image-20210828152857457](https://gitee.com/jobim/blogimage/raw/master/img/20210828152857.png)



代码示例：

* 场景：一个固定长度的资源池，当池为空时，请求资源会失败。使用 **`Semaphore`**可以实现当池为空时，请求会阻塞，非空时解除阻塞。也可以使用**`Semaphore`**将任何一种容器变成有界阻塞容器

  ```java
  public class SemaphoreDemo {
      public static void main(String[] args) {
          // 创建一个无界线程池
          ExecutorService exec = Executors.newCachedThreadPool();
          // 配置只能5个线程同时访问
          final Semaphore semaphore = new Semaphore(3);
          // 模拟10个客户端访问
          for (int i = 0; i < 5; i++) {
              int num = i;
              Runnable task = (() ->{
                  try {
                      // 获取许可
                      semaphore.acquire();
                      System.out.println("获得许可: " + num);
                      //休眠随机秒(表示正在执行操作)
                      TimeUnit.SECONDS.sleep((int)(Math.random()*10+1));
                      // 访问完后，释放许可
                      semaphore.release();
                      // availablePermits()指还剩多少个许可
                      System.out.println("----------当前还有多少个许可:" + semaphore.availablePermits());
                  } catch (InterruptedException e) {
                      e.printStackTrace();
                  }
              });
  
              exec.execute(task);
          }
          // 退出线程池
          exec.shutdown();
      }
  }
  ```

* 执行结果：

  ![image-20210828153456243](https://gitee.com/jobim/blogimage/raw/master/img/20210828153456.png)





## 2.6 CountdownLatch

* CountDownLatch是一个同步工具类，用来协调多个线程之间的同步，或者说起到线程之间的通信（而不是用作互斥的作用）。

* CountDownLatch能够**使一个线程在等待另外一些线程完成各自工作之后，再继续执行**。使用一个计数器进行实现。计数器初始值为线程的数量。**当每一个线程完成自己任务后，计数器的值就会减一。当计数器的值为0时，表示所有的线程都已经完成一些任务，然后在CountDownLatch上等待的线程就可以恢复执行接下来的任务。**

**CountDownLatch的用法**

* 某一线程在开始运行前等待n个线程执行完毕。将CountDownLatch的计数器初始化为new CountDownLatch(n)，每当一个任务线程执行完毕，就将计数器减1，countdownLatch.countDown()，当计数器的值变为0时，在CountDownLatch上await()的线程就会被唤醒。一个典型应用场景就是启动一个服务时，主线程需要等待多个组件加载完毕，之后再继续执行。



**CountDownLatch的不足**

* CountDownLatch是一次性的，计算器的值只能在构造方法中初始化一次，之后没有任何机制再次对其设置值，当CountDownLatch使用完毕后，它不能再次被使用。



* 常用方法说明：

  ```java
  CountDownLatch(int count); //构造方法，创建一个值为count 的计数器。
  await();//阻塞当前线程，将当前线程加入阻塞队列。
  await(long timeout, TimeUnit unit);//在timeout的时间之内阻塞当前线程,时间一过则当前线程可以执行，
  countDown();//对计数器进行递减1操作，当计数器递减至0时，当前线程会去唤醒阻塞队列里的所有线程。
  ```



代码示例：

```java
@Slf4j(topic = "c.TestCountDownLatch")
public class TestCountDownLatch {

    public static void main(String[] args) {

        CountDownLatch latch = new CountDownLatch(3);

        ExecutorService service = Executors.newFixedThreadPool(4);

        service.submit(() -> {
            log.debug("begin...");
            sleep(1);
            latch.countDown(); //t1线程执行结束,对计数器进行递减1
            log.debug("end...{}", latch.getCount());
        },"t1");
        service.submit(() -> {
            log.debug("begin...");
            sleep(1.5);
            latch.countDown(); //t2线程执行结束,对计数器进行递减1
            log.debug("end...{}", latch.getCount());
        },"t2");
        service.submit(() -> {
            log.debug("begin...");
            sleep(2);
            latch.countDown(); //t3线程执行结束,对计数器进行递减1
            log.debug("end...{}", latch.getCount());
        },"t3");
        service.submit(()->{
            try {
                log.debug("waiting...");
                latch.await(); // t4线程等待其他三个线程执行结束
                log.debug("wait end...");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"t4");
    }

}
```

执行结果：

![image-20210828154656327](https://gitee.com/jobim/blogimage/raw/master/img/20210828154656.png)





* 相比于join，join属于底层API使用起来相对繁琐。而且对于使用线程池的方法，不能使用join等待线程结束了。
* 对于等待线程执行结束的操作，不需要返回值的时候可以使用CountdownLatch，需要返回值的时候可以使用Future

## 2.7 CyclicBarrier



* CyclicBarrier也叫循环栅栏（同步屏障），CyclicBarrier可以协同多个线程，让**多个线程在这个屏障前等待，直到所有线程都达到了这个屏障时，再一起继续执行后面的动作**。构造时设置『计数个数』，每个线程执行到某个需要“同步”的时刻调用 **await() 方法进行等待，等待数+1**，当等待的线程数满足『计数个数』时，继续执行



代码示例：两个线程同时执行，循环三次

```java
@Slf4j(topic = "c.TestCyclicBarrier")
public class TestCyclicBarrier {

    public static void main(String[] args) {
        ExecutorService service = Executors.newFixedThreadPool(2);
        
        // 注意：线程数和CyclicBarrier的计数相同才会到达预期的效果
        CyclicBarrier barrier = new CyclicBarrier(2, ()-> {
            log.debug("task1, task2 finish...");
        });// 可以重复被使用，当计数变为0之后，会重新恢复为2
        
        for (int i = 0; i < 3; i++) { // task1  task2  task1
            service.submit(() -> {
                log.debug("task1 begin...");
                sleep(1);
                try {
                    barrier.await(); // 2-1=1
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
            });
            service.submit(() -> {
                log.debug("task2 begin...");
                sleep(2);
                try {
                    barrier.await(); // 1-1=0
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
            });
        }
        
        service.shutdown();
    }
    
}
```



执行结果：

![image-20210828155742108](https://gitee.com/jobim/blogimage/raw/master/img/20210828155742.png)



**CyclicBarrier和CountDownLatch的区别？**

- CountDownLatch：一个或者多个线程，**等待其他多个线程完成某件事情之后才能执行**；
- CyclicBarrier：多个线程互相等待，直到到达同一个同步点，再继续**一起执行**。而且可以重用

CountDownLatch是计数器，线程完成一个记录一个，只不过计数不是递增而是递减，而CyclicBarrier更像是一个阀门，需要所有线程都到达，阀门才能打开，然后继续执行。





# 三、线程安全集合类



**早期线程安全的集合：**

1. **Vector**
   Vector和ArrayList类似，是长度可变的数组，与ArrayList不同的是，Vector是线程安全的，它给几乎所有的**public方法都加上了synchronized关键字**。由于加锁导致性能降低，在不需要并发访问同一对象时，这种强制性的同步机制就显得多余，所以现在Vector已被弃用

2. **HashTable**
   HashTable和HashMap类似，不同点是HashTable是线程安全的，它给几乎所有**public方法都加上了synchronized关键字**，还有一个不同点是HashTable的K，V都不能是null，但HashMap可以，它现在也因为性能原因被弃用了





**Collections包装方法：**



* Collections 类中提供了多个 synchronizedXxx() 方法，该方法可使将指定集合包装成线程同步的集合，从而可以解决多线程并发访问集合时的线程安全问题

  ```java
  List<E> synArrayList = Collections.synchronizedList(new ArrayList<E>());
  
  Set<E> synHashSet = Collections.synchronizedSet(new HashSet<E>());
  
  Map<K,V> synHashMap = Collections.synchronizedMap(new HashMap<K,V>());
  
  ...
  ```

* 其内部利用装饰模式根据传入的Collection生成特定同步的SynchronizedCollection，生成的集合每个同步操作都是持有mutex这个锁，所以再进行操作时就是线程安全的集合了。

  ![image-20210826171326355](https://gitee.com/jobim/blogimage/raw/master/img/20210826171326.png)





**此外java.util.concurrent包中的集合为线程安全集合**



## 3.1 ConcurrentHashMap



### 3.1.1 JDK 7分析

#### 源码分析

在JDK1.7版本中，ConcurrentHashMap的数据结构是**由一个Segment数组和多个HashEntry组成**，如下图所示：

![image-20210826175725272](https://gitee.com/jobim/blogimage/raw/master/img/20210826175725.png)



* ConcurrentHashMap的**底层数据结构**：
  * ConcurrentHashMap内有一个**final修饰的Segment数组**

    ```java
    final Segment<K,V>[] segments; //Segments 数组默认大小为16，这个容量初始化指定后就不能改变了（相当于并发度）
    ```

  * **Segment的数据结构：**

    ```java
    static final class Segment<K,V> extends ReentrantLock implements Serializable {
        transient volatile HashEntry<K,V>[] table; //存储结构
        transient int count; //Segment中元素的数量
        transient int modCount; 
        transient int threshold; //Segment里面元素的数量超过这个值就会对Segment进行扩容
        final float loadFactor; //负载因子，用于确定threshold
    }
    ```

  * **HashEntry结构：**Segment中的元素是以HashEntry的形式存放在链表数组中的

    ```java
    static final class HashEntry<K,V> {
        final int hash;
        final K key;
        volatile V value;
        volatile HashEntry<K,V> next;
    }
    ```





* **put流程**

  * put操作的步骤：
    * 首先，计算key的hash值
    * 其次，根据hash值找到需要操作的Segment的数组位置
    * Segment为空，调用ensureSegment()方法；否则，直接调用查询到的Segment的put方法插入值

    ```java
    public V put(K key, V value) {
        Segment<K,V> s;
        // concurrentHashMap不允许key/value为空
        if (value == null)
            throw new NullPointerException();
        int hash = hash(key);
        // 计算出 segment 下标
        int j = (hash >>> segmentShift) & segmentMask;
    
        // 获得 segment 对象, 判断是否为 null, 是则创建该 segment
        if ((s = (Segment<K,V>)UNSAFE.getObject
                (segments, (j << SSHIFT) + SBASE)) == null) {
            // 这时不能确定是否真的为 null, 因为其它线程也发现该 segment 为 null,
            // 因此在 ensureSegment 里用 cas 方式保证该 segment 安全性
            s = ensureSegment(j);
        }
        // 进入 segment 的put 流程
        return s.put(key, hash, value, false);
    }
    ```

  * 调用**Segment的put方法**

    在Segment的put方法中，首先需要调用tryLock()方法获取锁，然后通过hash算法定位到对应的HashEntry，然后遍历整个链表，如果没有查到key值，则直接插元素即可；而如果查询到对应的key，则插入元素。如果超过了该 segment 的阈值，则需要调用rehash()方法对Segment中保存的table进行扩容，扩容为原来的2倍，并在扩容之后插入对应的元素。插入一个key/value对后，需要将统计Segment中元素个数的count属性加1。最后，插入成功之后，需要使用unLock()释放锁。

    ```java
    final V put(K key, int hash, V value, boolean onlyIfAbsent) {
        // 尝试加锁
        HashEntry<K,V> node = tryLock() ? null :
                // 如果不成功, 进入 scanAndLockForPut 流程
                // 如果是多核 cpu 最多 tryLock 64 次, 进入 lock 流程
                // 在尝试期间, 还可以顺便看该节点在链表中有没有, 如果没有顺便创建出来
                scanAndLockForPut(key, hash, value);
    
        // 执行到这里 segment 已经被成功加锁, 可以安全执行
        V oldValue;
        try {
            HashEntry<K,V>[] tab = table;
            // 再利用 hash 值，求应该放置的数组下标
            int index = (tab.length - 1) & hash;
            // 返回数组中对应位置的元素（链表头部）
            HashEntry<K,V> first = entryAt(tab, index);
            for (HashEntry<K,V> e = first;;) {
                if (e != null) {
                    // 如果已经存在值，覆盖旧值
                    K k;
                    if ((k = e.key) == key ||
                            (e.hash == hash && key.equals(k))) {
                        oldValue = e.value;
                        if (!onlyIfAbsent) {
                            e.value = value;
                            ++modCount;
                        }
                        break;
                    }
                    e = e.next;
                }
                else {
                    // 新增
                    // 1) 之前等待锁时, node 已经被创建, next 指向链表头
                    if (node != null) // 非空，则表示为新创建的值
                        node.setNext(first);
                    else
                        // 2) 创建新 node
                        node = new HashEntry<K,V>(hash, key, value, first);
                    int c = count + 1;
                    // 3) 如果超过了该 segment 的阈值，这个 segment 需要扩容
                    if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                        rehash(node);
                    else
                        // 将 node 作为链表头，头插法
                        setEntryAt(tab, index, node);
                    ++modCount;
                    count = c;
                    oldValue = null;
                    break;
                }
            }
        } finally {
            unlock(); // 最终释放锁
        }
        return oldValue;
    }
    ```



* **get方法**

  get 时并未加锁，用了 UNSAFE 方法保证了可见性，扩容过程中，get 先发生就从旧表取内容，get 后发生就从新表取内容

  ```java
  public V get(Object key) {
      Segment<K,V> s; // manually integrate access methods to reduce overhead
      HashEntry<K,V>[] tab;
      // 1. hash 值
      int h = hash(key);
      long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
      // 2. 根据 hash 找到对应的 segment
      if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null &&
          (tab = s.table) != null) {
          // 3. 找到segment 内部数组相应位置的链表，遍历
          for (HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
                   (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
               e != null; e = e.next) {
              K k;
              if ((k = e.key) == key || (e.hash == h && key.equals(k)))
                  return e.value;
          }
      }
      return null;
  }
  ```



#### 总结



* ConcurrentHashMap采用的是**锁分段技术**，内部为S**egment数组来进行细分**，而**每个Segment又通过HashEntry数组**来进行组装。

* ConcurrentHashMap 定位一个元素的过程需要进行**两次Hash操作**，**第一次 Hash 定位到 Segment，第二次 Hash 定位到元素所在的链表的头部**（这一种结构的带来的副作用是 Hash 的过程要比普通的 HashMap 要长），但是带来的好处是写操作的时候可以只对元素所在的 Segment 进行操作即可，**不会影响到其他的 Segment**，这样，在最理想的情况下，ConcurrentHashMap 可以**最高同时支持 Segment 数量大小的写操作**







### 3.1.2 JDK 8分析

#### 源码分析

* **重要属性和内部类结构**

  ```java
  // 默认为 0
  // 当初始化时, 为 -1
  // 当扩容时, 为 -(1 + 扩容线程数)
  // 当初始化或扩容完成后，为 下一次的扩容的阈值大小
  private transient volatile int sizeCtl;
  // 整个 ConcurrentHashMap 就是一个 Node[]
  static class Node<K,V> implements Map.Entry<K,V> {
      final int hash;
      final K key;
      volatile V val;
      volatile Node<K,V> next;
  }
  
  // hash 表
  transient volatile Node<K,V>[] table;
  // 扩容时的 新 hash 表
  private transient volatile Node<K,V>[] nextTable;
  // 扩容时如果某个 bin 迁移完毕, 用 ForwardingNode 作为旧 table bin 的头结点
  static final class ForwardingNode<K,V> extends Node<K,V> {}
  // 用在 compute 以及 computeIfAbsent 时, 用来占位, 计算完成后替换为普通 Node
  static final class ReservationNode<K,V> extends Node<K,V> {}
  // 通过TreeNode作为存储结构代替Node来转换成黑红树
  static final class TreeNode<K,V> extends Node<K,V> {}
  // 作为 treebin 的头节点, 存储 root 和 first。相当于TreeBin就是封装TreeNode的容器
  static final class TreeBin<K,V> extends Node<K,V> {}
  ```



* 初始化操作，ConcurrentHashMap 为惰性初始化，第一次调用put操作是时先调用initTable()方法来进行初始化过程

  ```java
  private final Node<K,V>[] initTable() {
      Node<K,V>[] tab; int sc;
      //空的table才能进入初始化操作
      while ((tab = table) == null || tab.length == 0) {
          if ((sc = sizeCtl) < 0) //sizeCtl<0表示其他线程已经在初始化了或者扩容了，挂起当前线程
              Thread.yield();
              // 尝试将 sizeCtl 设置为 -1（表示初始化 table）
          else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
              // 获得锁, 创建 table, 这时其它线程会在 while() 循环中 yield 直至 table 创建
              try {
                  if ((tab = table) == null || tab.length == 0) {
                      int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                      Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n]; //初始化
                      table = tab = nt;
                      sc = n - (n >>> 2); //记录下次扩容的大小
                  }
              } finally {
                  sizeCtl = sc;
              }
              break;
          }
      }
      return tab;
  }
  ```

  

* **put操作**

  * 如果没有初始化就先调用initTable()方法来进行初始化过程
  * 然后通过计算hash值来确定放在数组的哪个位置
    * 如果没有hash冲突就直接CAS插入，如果hash冲突的话，则取出这个节点来
    * 如果取出来的节点的hash值是MOVED(-1)的话，则表示当前正在对**这个数组进行扩容，复制到新的数组，则当前线程也去帮助复制**
    * 如果这个节点，不为空，也不在扩容，则**通过synchronized来加锁，进行添加操作**，然后判断当前取出的节点位置存放的是链表还是树
      * 如果**是链表的话**，则遍历整个链表，直到取出来的节点的key和要放的key进行比较，如果key相等，并且key的hash值也相等的话，则说明是同一个key，则覆盖掉value，否则的话则添加到链表的末尾
      * 如果**是树的话**，则调用putTreeVal方法把这个元素添加到树中去
      * 最后在添加完成之后，调用addCount（）方法统计size，判断在该节点处共有多少个节点（注意是添加前的个数），如果达到8个以上了的话，则调用treeifyBin方法来尝试将处的链表转为树，或者扩容数组

  ```java
  public V put(K key, V value) {
      return putVal(key, value, false);
  }
  final V putVal(K key, V value, boolean onlyIfAbsent) {
      // 由此可以得出不支持null键和null值
      if (key == null || value == null) throw new NullPointerException();
      // 得到 hash 
      int hash = spread(key.hashCode());
      // 用于记录相应链表的长度
      int binCount = 0;
      for (Node<K,V>[] tab = table;;) {//对table迭代
          Node<K,V> f; int n, i, fh;
          // 如果数组"空"，进行数组初始化。运用的是懒汉式初始化
          if (tab == null || (n = tab.length) == 0)
              // 初始化数组，对table进行初始化操作
              tab = initTable();
   
          // 找该 hash 值对应的数组下标，得到第一个节点 f
          else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
              // 如果数组该位置为空，
              // 用一次 CAS 操作将这个新值放入其中即可，这个 put 操作差不多就结束了，可以拉到最后面了
              // 如果 CAS 失败，那就是有并发操作，进到下一个循环就好了
              if (casTabAt(tab, i, null,
                           new Node<K,V>(hash, key, value, null)))
                  break;                   // no lock when adding to empty bin
          }
          // 如果正在扩容，就先进行扩容操作
          else if ((fh = f.hash) == MOVED)
              // 帮助数据迁移
              tab = helpTransfer(tab, f);
   
          else { // 到这里就是说，f 是该位置的头结点，而且不为空
   
              V oldVal = null;
              // 如果以上条件都不满足，那就要进行加锁操作，也就是存在hash冲突，锁住链表或者红黑树的头结点
              synchronized (f) {
                  if (tabAt(tab, i) == f) {
                      if (fh >= 0) { // 头结点的 hash 值大于 0，说明是链表
                          // 用于累加，记录链表的长度
                          binCount = 1;
                          // 遍历链表
                          for (Node<K,V> e = f;; ++binCount) {
                              K ek;
                              // 如果发现了"相等"的 key，判断是否要进行值覆盖，然后也就可以 break 了
                              if (e.hash == hash &&
                                  ((ek = e.key) == key ||
                                   (ek != null && key.equals(ek)))) {
                                  oldVal = e.val;
                                  if (!onlyIfAbsent)
                                      e.val = value;
                                  break;
                              }
                              // 到了链表的最末端，将这个新值放到链表的最后面
                              Node<K,V> pred = e;
                              if ((e = e.next) == null) {
                                  pred.next = new Node<K,V>(hash, key,
                                                            value, null);
                                  break;
                              }
                          }
                      }
                      else if (f instanceof TreeBin) { // 红黑树
                          Node<K,V> p;
                          binCount = 2;
                          // 调用红黑树的插值方法插入新节点
                          if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                         value)) != null) {
                              oldVal = p.val;
                              if (!onlyIfAbsent)
                                  p.val = value;
                          }
                      }
                  }
              }
              // binCount != 0 说明上面在做链表操作
              if (binCount != 0) {
                  // 判断是否要将链表转换为红黑树，临界值和 HashMap 一样，也是 8
                  if (binCount >= TREEIFY_THRESHOLD)
                      // 这个方法和 HashMap 中稍微有一点点不同，那就是它不是一定会进行红黑树转换，
                      // 如果当前数组的长度小于 64，那么会选择进行数组扩容，而不是转换为红黑树
                      // 这个方法上面已经说过了
                      treeifyBin(tab, i);
                  if (oldVal != null)
                      return oldVal;
                  break;
              }
          }
      }
      // 增加 size 计数 
      addCount(1L, binCount);
      return null;
  }
  ```

  





* **get操作**

  * 根据 hash 值找到数组对应位置: (n - 1) & h
  * 如果该位置为 null，那么直接返回 null 就可以了
  * 如果该位置处的节点刚好就是我们需要的，返回该节点的值即可
  * 如果该位置节点的 hash 值小于 0，说明正在扩容，或者是红黑树。这时调用 find 方法来查找
  * 否则那就是链表，进行遍历比对即可
```java
  public V get(Object key) {
      Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
      // spread 方法能确保返回结果是正数
      int h = spread(key.hashCode());
      if ((tab = table) != null && (n = tab.length) > 0 &&
              (e = tabAt(tab, (n - 1) & h)) != null) {
          // 如果头结点已经是要查找的 key
          if ((eh = e.hash) == h) {
              if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                  return e.val;
          }
          // hash 为负数表示该 bin 在扩容中或是 treebin, 这时调用 find 方法来查找
          else if (eh < 0)
              return (p = e.find(h, key)) != null ? p.val : null;
          // 正常遍历链表, 用 equals 比较
          while ((e = e.next) != null) {
              if (e.hash == h &&
                      ((ek = e.key) == key || (ek != null && key.equals(ek))))
                  return e.val;
          }
      }
      return null;
  }
```



#### 总结

![image-20210826203806819](https://gitee.com/jobim/blogimage/raw/master/img/20210826203806.png)

在**JDK1.8中，是采用Node + CAS + Synchronized来保证并发安全进行实现**，synchronized只锁定当前链表或红黑二叉树的首节点，这样只要hash不冲突，就不会产生并发，



* put()操作时，如果没有初始化就先调用initTable()方法来进行初始化过程
* 然后通过计算hash值来确定放在数组的哪个位置
  * 如果没有hash冲突就直接CAS插入，如果hash冲突的话，则取出这个节点来
  * 如果取出来的节点的hash值是MOVED(-1)的话，则表示当前正在对**这个数组进行扩容，复制到新的数组，则当前线程也去帮助复制**
  * 如果这个节点，不为空，也不在扩容，则**通过synchronized来加锁，进行添加操作**，然后判断当前取出的节点位置存放的是链表还是树
    * 如果**是链表的话**，则遍历整个链表，直到取出来的节点的key和要放的key进行比较，如果key相等，并且key的hash值也相等的话，则说明是同一个key，则覆盖掉value，否则的话则添加到链表的末尾
    * 如果**是树的话**，则调用putTreeVal方法把这个元素添加到树中去
    * 最后在添加完成之后，调用addCount（）方法统计size，判断在该节点处共有多少个节点（注意是添加前的个数），如果达到8个以上了的话，则调用treeifyBin方法来尝试将处的链表转为树，或者扩容数组

* **在同一个位置的个数又达到了8个以上，如果数组的长度还小于64的时候，则会扩容数组。如果数组的长度大于等于64了的话，在会将该节点的链表转换成树。**在最后增加map的总node数量时，若总数超过table长度的0.75倍则会进行扩容，扩容时会将**下标倒序分为几块任务，可由其余线程帮助完成扩容**

* get()时会根据key定位到下标，然后遍历链表或数组查找对应的node，如果**下标内是代表rehash的特殊node，则会去临时扩容table内查询数据**
* remove()时会根据下标遍历下标内的链表或者红黑树，如果下标内是代表rehash的特殊node，则会帮组扩容



<hr/>



**JDK7与JDK8中ConcurrentHashMap的区别：**

* JDK1.7使用的是ReentrantLock+Segment+HashEntry，而到JDK1.8版本使用synchronized+CAS+数组（Node） +（ 链表 Node | 红黑树 TreeNode ）
* JDK7采用Segment的**分段锁机制**实现线程安全，对需要进行修改的Segment加锁。而JDK8采用**CAS+synchronized**保证线程安全，synchronized只锁定当前链表或红黑二叉树的首节点
* JDK1.8为什么使用内置锁synchronized来代替重入锁ReentrantLock
* JDK1.8使用红黑树来优化链表
  
  

**ConCurrentHashmap 每次扩容是原来容量的几倍?**

> 2倍在transfer方法里面会创建一个原数组的俩倍的node数组来存放原数据。





## 3.2 CopyOnWriteArrayList

`CopyOnWriteArrayList`是ArrayList的线程安全版本，使用了一种叫**写时复制**的方法，适用于读多写少的并发场景，当有新元素添加到`CopyOnWriteArrayList`时，**先从原有的数组中拷贝一份出来，然后在新的数组做写操作，写完之后，再将原来的数组引用指向到新数组。**

`CopyOnWriteArraySet`是HashSet的线程安全版本



* `CopyOnWriteArrayList`的**整个add操作都是在锁的保护下**进行的（JDK1.8）

  ```java
  public boolean add(E e) {
      final ReentrantLock lock = this.lock;
      lock.lock();
      try {
          // 获取旧的数组
          Object[] elements = getArray();
          int len = elements.length;
          // 拷贝新的数组（这里是比较耗时的操作，但不影响其它读线程）
          Object[] newElements = Arrays.copyOf(elements, len + 1);
          // 添加新元素
          newElements[len] = e;
          // 替换旧的数组
          setArray(newElements);
          return true;
      } finally {
          lock.unlock();
      }
  }
  ```

  Java 11的时候不再使用可重入锁而是synchronized



* 而CopyOnWriteArrayList的**读操作不加锁**

* 有线程并发的读，则分几种情况：
  1、如果写操作未完成，那么直接读取原数组的数据；
  2、如果写操作完成，但是引用还未指向新数组，那么也是读取原数组数据；
  3、如果写操作完成，并且引用已经指向了新的数组，那么直接从新数组中读取数据。

