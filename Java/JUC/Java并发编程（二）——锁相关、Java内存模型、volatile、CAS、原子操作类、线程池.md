# 一、Java锁相关

## 1.1 悲观锁和乐观锁

**悲观锁：**

* 悲观锁是就是悲观思想，即认为写多，遇到并发写的可能性高，每次去拿数据的时候都认为别人会修改，所以每次在读写数据的时候都会上锁，这样别人想读写这个数据就会 block 直到拿到锁。java中的悲观锁就是Synchronized,AQS框架下的锁则是先尝试cas乐观锁去获取锁，获取不到，才会转换为悲观锁，如 RetreenLock。传统的关系型数据库里边就用到了很多这种锁机制，**比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。**

**乐观锁：**

* 乐观锁的思想与悲观锁的思想相反，它总认为资源和数据不会被别人所修改，所以读取不会上锁，但是乐观锁在进行写入操作的时候会判断当前数据是否被修改过。乐观锁的实现方案一般来说有两种： `版本号机制` 和 `CAS实现` 。乐观锁多适用于多度的应用类型，这样可以提高吞吐量。
* 在Java中`java.util.concurrent.atomic`包下面的原子变量类就是使用了乐观锁的一种实现方式 CAS 实现的。





## 1.2 公平锁和非公平锁

**公平锁：**

* 所谓的公平是指在锁等待队列中获取到锁先后关系，先到先得的思想。优先队列中的线程获取锁。
* 优点：所有的线程都能得到资源，不会饿死在队列中。
* 缺点：吞吐量会下降很多，队列里面除了第一个线程，其他的线程都会阻塞，cpu唤醒阻塞线程的开销会很大

**非公平锁：**

* 加锁时不考虑排队等待问题，直接尝试获取锁，获取不到自动到队尾等待
  * 非公平锁性能比公平锁高，因为公平锁需要在多核的情况下维护一个队列
  * Java 中的 synchronized 是非公平锁，ReentrantLock 默认的 lock()方法采用的也是非公平锁。

- 优点：可以减少CPU唤醒线程的开销，整体的吞吐效率会高点，CPU也不必取唤醒所有线程，会减少唤起线程的数量。（一些线程可以直接抢占锁，不用被阻塞）
- 缺点：可能导致队列中间的线程一直获取不到锁或者长时间获取不到锁，导致饿死。



## 1.3 死锁（重点）



**什么是死锁？**

* 死锁是指两个或多个以上的进程在执行过程中，因争夺资源而造成一种**互相等待**的现象，若无外力干涉那他们都将无法推进下去。



**形成死锁的四个必要条件是什么?**

* **互斥条件**：线程(进程)对于所分配到的资源具有排它性，即一个资源只能被一个线程(进程)占用，直到被该线程(进程)释放
* **请求与保持条件（占有且等待）**：一个线程(进程)因请求被占用资源而发生阻塞时，对已获得的资源保持不放。
* **不剥夺条件**：线程(进程)已获得的资源在末使用完之前不能被其他线程强行剥夺，只有自己使用完毕后才释放资源。
* **循环等待条件**：当发生死锁时，所等待的线程(进程)必定会形成一个环路（类似于死循环），造成永久阻塞



**如何预防线程死锁?**

* **破坏互斥条件**

  这个条件很多时候没有办法破坏，因为我们用锁本来就是想让他们互斥的（临界资源需要互斥访问）。当然也可以把某些独占设备在逻辑上改成共享设备。

* **破坏请求与保持条件**

  一次性申请所有的资源。

* **破坏不剥夺条件**

  方法一：**占用部分资源的线程进一步申请其他资源时，如果在一段时间申请不到，可以主动释放它占有的资源**。

  方法二：采用剥夺式的调度算法

* **破坏循环等待条件**

  规定线程获取锁的顺序

* **银行家算法**







**死锁的排查（重点）**

* 检测死锁可以使用 **jconsole工具**，或者**使用 jps 定位进程 id，再用 jstack 定位死锁**

* **方法一：使用jps查出java相关的后台进程，再用 jstack 定位死锁**
    > 1、使用 jps 定位进程 id：
    >
    > ![image-20210820193217653](https://gitee.com/jobim/blogimage/raw/master/img/20210820193217.png)
    >
    > 2、使用 jstack 定位死锁：
    >
    > ![image-20210820193546621](https://gitee.com/jobim/blogimage/raw/master/img/20210820193546.png)
    > ![image-20210820193802556](https://gitee.com/jobim/blogimage/raw/master/img/20210820193802.png)
    
* **方法二：使用 jconsole工具**
  
  > ![image-20210820193037386](https://gitee.com/jobim/blogimage/raw/master/img/20210820193037.png)





## 1.4 独占锁（写锁）和共享锁（读锁）

**独占锁：**

* 独占锁也叫排它锁，是指每次只能有一个线程能持有锁，ReentrantLock 就是以独占方式实现的互斥锁。ReentrantLock和Synchronized都是独占锁。

**共享锁：**

* 共享锁则允许多个线程同时获取锁，并发访问共享资源。ReentrantReadWriteLock其读锁是共享锁，写锁是独占锁。



## 1.5 自旋锁（spinlock）

**自旋锁（spinlock）**：是指当一个线程在获取锁的时候，如果锁已经被其它线程获取，那么该线程将循环等待，然后不断的判断锁是否能够被成功获取，直到获取到锁才会退出循环。



**自旋锁优缺点：**

* 自旋锁尽可能的减少线程的阻塞，这对于锁的竞争不激烈，且占用锁时间非常短的代码块来说性能能大幅度的提升，因为自旋的消耗会小于线程阻塞挂起再唤醒的操作的消耗，这些操作会导致线程发生两次上下文切换！
* 但是如果锁的竞争激烈，或者持有锁的线程需要长时间占用锁执行同步块，这时候就不适合使用自旋锁了，因为自旋锁在获取锁前一直都是占用 cpu 做无用功，占着 XX 不 XX，同时有大量线程在竞争一个锁，会导致获取锁的时间很长，线程自旋的消耗大于线程阻塞挂起操作的消耗，其它需要 cup 的线程又不能获取到 cpu，造成 cpu 的浪费。所以这种情况下我们要关闭自旋锁；



## 1.6 无锁、偏向锁、轻量锁、重量锁

synchronized锁一共有4种状态，级别从低到高依次是：无锁状态 --> 偏向锁状态 --> 轻量级锁状态 --> 重量级锁状态，这几种状态会随着竞争情况逐渐升级。锁可以升级但不能降级。

| 锁     | 获得锁的方式                                 | 优点                                                         | 缺点                                         | 适用场景               |
| ------ | -------------------------------------------- | ------------------------------------------------------------ | -------------------------------------------- | ---------------------- |
| 偏向锁 | 在对象头和栈帧中的锁记录里面存储偏向的线程ID | 加锁和解锁不需要额外的消耗，和执行非同步方法相比仅存在纳秒级的差距 | 如果线程存在锁竞争，会带来额外的锁撤销的消耗 | 只有一个线程访问同步块 |
| 轻量级锁 | CAS操作成功更新对象Mark Word指向Lock Record的指针 | 竞争的线程不会阻塞，提高了程序的响应速度 | 如果始终得不到锁竞争的线程，使用自旋会消耗CPU | 追求响应时间，同步块执行速度非常快 |
| 重量级锁 | 获取到对象的monitor | 线程竞争不使用自旋，不会消耗CPU | 线程阻塞，响应时间缓慢 | 追求吞吐量，同步块执行时间长 |







# 二、Java内存模型

[ 一篇文章搞懂java内存模型、JMM三大特征、volatile关键字 ](https://zhuanlan.zhihu.com/p/258393139)

## 2.1 是什么JMM？

JMM 即 Java Memory Model （Java内存模型），因为在不同的硬件生产商和不同的操作系统下，内存的访问有一定的差异，所以会造成相同的代码运行在不同的系统上会出现各种问题。所以**java内存模型(JMM)屏蔽掉各种硬件和操作系统的内存访问差异，以实现让java程序在各种平台下都能达到一致的并发效果。** **它从Java层面定义了 主存、工作内存抽象概念**，底层对应着CPU 寄存器、缓存、硬件内存、CPU 指令优化等。

即：JMM 定义了一套在多线程读写共享数据时（成员变量、数组）时，对数据的可见性、有序性、和原子性的规则和保障

Java内存模型规定**所有的变量都存储在主内存**中，包括实例变量，静态变量，但是不包括局部变量和方法参数。**每个线程都有自己的工作内存，线程的工作内存保存了该线程用到的变量和主内存的副本拷贝，线程对变量的操作都在工作内存中进行**。**线程不能直接读写主内存中的变量**。 

![image-20210822101628521](https://gitee.com/jobim/blogimage/raw/master/img/20210828172321.png)



## 2.2 JMM下三大特性

* **原子性** -  指的是一个操作是不可分割，不可中断的，一个线程在执行时不会被其他线程干扰 
* **可见性** -  可见性指当一个线程修改共享变量的值，其他线程能够立即知道被修改了 ，保证指令不会受 cpu 缓存的影响 
* **有序性** - 保证指令不会受 cpu 指令并行优化的影响



**原子性：**

* 原子性指的是一个操作是不可分割，不可中断的，一个线程在执行时不会被其他线程干扰。

* 分析下面代码的原子性

  ```java
int i = 2;
  int j = i;
  i++;
  ```
  
  第一句是基本类型赋值操作，必定是原子性操作。

  第二句先读取i的值，再赋值到j，两步操作，不能保证原子性。

  第三句，先读取i的值，再+1，最后赋值到i，三步操作了，不能保证原子性。

* JMM只能保证基本的原子性，如果要保证一个代码块的原子性，提供了monitorenter 和 moniterexit 两个字节码指令，也就是 synchronized 关键字。因此在 synchronized 块之间的操作都是原子性的。



**可见性：**

* **可见性指当一个线程修改共享变量的值，其他线程能够立即知道被修改了**。Java是利用volatile关键字来提供可见性的。 当变量被volatile修饰时，这个变量被修改后会立刻刷新到主内存，当其它线程需要读取该变量时，会去主内存中读取新值。而普通变量则不能保证这一点。

* 除了volatile关键字之外，final和synchronized也能实现可见性。

* **synchronized的原理**：`先清空工作内存` → 在主内存中拷贝最新变量的副本到工作内存 → 执行完代码 → `将更改后的共享变量的值刷新到主内存中 `→ 释放互斥锁



**有序性：**

* 为了提供性能，编译器和处理器通常会**对指令序列进行重新排序**。

* 指令重排可以保证单线程环境里面确保程序最终执行结果和代码顺序执行的结果一致。而多线程环境中线程交替执行,由于编译器优化重排的存在，两个线程中使用的变量能否保证一致性是无法确定的

* 在Java中，可以使用synchronized或者volatile保证多线程之间操作的有序性。实现原理有些区别：

  * volatile关键字是使用**内存屏障**达到禁止指令重排序，以保证有序性。

  * synchronized的原理是，一个线程lock之后，必须unlock后，其他线程才可以重新lock，使得被synchronized包住的代码块在多线程之间是串行执行的。



## 2.3 八种内存交互操作

* Java内存模型定义了八种内存交互操作



> 以下来自深入理解Java虚拟机第三版

![image-20210822114511039](https://gitee.com/jobim/blogimage/raw/master/img/20210822114511.png)

- read(读取)，作用于**主内存**的变量，把变量的值从主内存传输到线程的工作内存中，以便下一步的load操作使用。
- load(加载)，作用于**工作内存**的变量，把read操作主存的变量放入到工作内存的变量副本中。
- use(使用)，作用于**工作内存**的变量，把工作内存中的变量传输到执行引擎，每当虚拟机遇到一个需要使用到变量的值的字节码指令时将会执行这个操作。
- assign(赋值)，作用于**工作内存**的变量，它把一个从执行引擎中接受到的值赋值给工作内存的变量副本中，每当虚拟机遇到一个给变量赋值的字节码指令时将会执行这个操作。
- store(存储)，作用于**工作内存**的变量，它把一个从工作内存中一个变量的值传送到主内存中，以便后续的write使用。
- write(写入)：作用于**主内存**中的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中。
- lock(锁定)，作用于**主内存**中的变量，把变量标识为线程独占的状态。
- unlock(解锁)：作用于**主内存**的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。

JMM对8种内存交互操作制定的规则：

- 不允许read、load、store、write操作之一单独出现，也就是read操作后必须load，store操作后必须write。
- 不允许线程丢弃他最近的assign操作，即工作内存中的变量数据改变了之后，必须告知主存。
- 不允许线程将没有assign的数据从工作内存同步到主内存。
- 一个新的变量必须在主内存中诞生，不允许工作内存直接使用一个未被初始化的变量。就是对变量实施use、store操作之前，必须经过load和assign操作。
- 一个变量同一时间只能有一个线程对其进行lock操作。多次lock之后，必须执行相同次数unlock才可以解锁。
- **如果对一个变量进行lock操作，会清空所有工作内存中此变量的值。在执行引擎使用这个变量前，必须重新load或assign操作初始化变量的值。**
- 如果一个变量没有被lock，就不能对其进行unlock操作。也不能unlock一个被其他线程锁住的变量。
- **一个线程对一个变量进行unlock操作之前，必须先把此变量同步回主内存。**





## 2.4 先行发生原则happens-before

* 从 JDK5 开始，java 内存模型提出了 happens-before 的概念，通过这个概念来阐述操作之间的内存可见性。

* 如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须存在 happens-before 关系。这里提到的两个操作既可以是在一个线程之内，也可以是在不同线程之间。



- **程序顺序规则**：一个线程中的每个操作，happens-before于该线程中的任意后续操作。
- **管程锁定规则**：对一个锁的解锁，happens-before于随后对这个锁的加锁。
- **volatile变量规则**：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。
- **传递性**：如果A happens-before B，且B happens-before C，那么A happens-before C。
- **线程启动规则**：这条是关于线程启动的。它是指主线程 A 启动子线程 B 后，子线程 B 能够看到主线程在启动子线程 B 前的操作。
- **线程终止规则**：线程中的所有操作都先行发生于对此线程的终止检测，我们可以通过Thread::join()方法是否结束、Thread::isAlive()的返回值等手段检测线程是否已经终止执行。
- **线程中断规则**：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生
- **对象终结规则**：一个对象的初始化完成（构造函数执行结束）先行发生于它的finalize()方法的开始。即：对象没有完成初始化之前，是不能调用finalized()方法的



# 三、volatile关键字



## 3.1 volatile特性

**volatile关键字的特性？**

 volatile是Java虚拟机提供的**轻量级**的同步机制，主要的作用：

1. **保证线程间变量的可见性。**

   volatile修饰的变量，当一个线程改变了该变量的值，会将共享变量值立即刷新回主内存。而线程读取共享变量必须从主内存中读取。

2. **禁止CPU进行指令重排序。**

3. 不能保证操作的原子性 



**保证可见性：**

* volatile修饰的变量，汇编指令中会存在于一个`lock前缀的指令`。
* lock前缀的指令在多核处理器下会引发两件事情：
  * **会将当前处理器缓存行的数据写回到系统内存（让线程工作内存中的值写回主内存中）**
  * **同时触发缓存一致性协议，使在其他CPU里缓存了该内存地址的数据无效，即其他线程如果工作内存中存了该共享变量的值，就会失效。其他线程会重新从主内存中获取最新的值**

参考文章：[volatile是如何保证可见性和有序性的](https://blog.csdn.net/weixin_37990128/article/details/110955558)



**禁止CPU进行指令重排：**

* volatile 的底层实现原理是`内存屏障`

* **在每个volatile读操作后插入LoadLoad屏障，在读操作后插入LoadStore屏障** 

  ![image-20210822151205774](https://gitee.com/jobim/blogimage/raw/master/img/20210822151205.png)

* **在每个volatile写操作的前面插入一个StoreStore屏障，后面插入一个SotreLoad屏障**

  ![image-20210822151140159](https://gitee.com/jobim/blogimage/raw/master/img/20210822151140.png)

  

* ![image-20210822150451445](https://gitee.com/jobim/blogimage/raw/master/img/20210822150451.png)

* 即：
  * **volatile写之前的操作，都禁止重排序到volatile之后**
  * **volatile读之后的操作，都禁止重排序到volatile之前**





**如何解决volatile不保证原子性问题？**

* 在方法上加入 synchronized，虽然能够保证原子性，但是为了解决number++，而引入重量级的同步机制

* 如何不加synchronized解决number++在多线程下是非线程安全的问题？使用**AtomicInteger**。



## 3.2 内存屏障

* 内存屏障（也称内存栅栏，内存栅障，屏障指令等，是一类同步屏障指令，是CPU或编译器在对内存随机访问的操作中的一个同步点，使得此点之前的所有读写操作都执行后才可以开始执行此点之后的操作），避免代码重排序。
* 内存屏障其实就是一种JVM指令，对于操作volatile变量，Java内存模型的重排规则会**要求Java编译器在生成JVM指令时插入特定的内存屏障指令**，通过这些内存屏障指令，volatile实现了**Java内存模型中的可见性和有序性**，但volatile无法保证原子性。



**内存屏障可分为四类：**

- **LoadLoad 屏障**：对于这样的语句Load1，LoadLoad，Load2。在Load2及后续读取操作要读取的数据被访问前，保证Load1要读取的数据被读取完毕。
- **StoreStore屏障**：对于这样的语句Store1， StoreStore， Store2。**在Store2及后续写入操作执行前，保证Store1的写入操作以刷新到主内存。**
- **LoadStore 屏障**：对于这样的语句Load1， LoadStore，Store2。在Store2及后续写入操作被刷出前，保证Load1要读操作已读取完毕。
- **StoreLoad 屏障**：对于这样的语句Store1， StoreLoad，Load2。在Load2及后续所有读取操作执行前，保证Store1的写入对所有处理器可见。

> **内存屏障的源码：**
>
> ![image-20210822105330855](https://gitee.com/jobim/blogimage/raw/master/img/20210822105331.png)





**volatile变量规则：**

* 当第一个操作为volatile读时，不论第二个操作是什么，都不能重排序。这个操作保证了volatile读之后的操作不会被重排到volatile读之前。
* 当第二个操作为volatile写时，不论第一个操作是什么，都不能重排序。这个操作保证了volatile写之前的操作不会被重排到volatile写之后。



**volatile和内存屏障的关联**

* **volatile写**

  * **在每个 `volatile写`操作的 前⾯ 插⼊⼀个 `StoreStore屏障`**
  * **在每个 `volatile写`操作的后⾯插⼊⼀个 `StoreLoad屏障`**

  ![image-20210822110612406](https://gitee.com/jobim/blogimage/raw/master/img/20210822110612.png)

* **volatile写**

  * 在**每个 `volatile 读`操作的后⾯插⼊⼀个 `LoadLoad 屏障`**
  * 在**每个 `volatile 读`操作的后⾯插⼊⼀个 `LoadStore 屏障`**

  ![image-20210822111311578](https://gitee.com/jobim/blogimage/raw/master/img/20210822111311.png)



## 3.3 DCL双端锁



**double-checked locking(双重检查锁) 单例模式**

* **分析不使用volatile的问题：**

  ```java
  public class Singleton {
  
      private static Singleton instance = null;
  
      private Singleton(){ }
  
      public static Singleton getInstance() {
          if(instance == null) {
              synchronized(Singleton.class) {
                  if(instance == null) {
                      instance = new Singleton();
                  }
              }
          }
          return instance;
      }
  
  }
  ```

  **代码问题分析：**

  **instance = new Singleton();**可以分解成三行伪代码

  ```c
  //1：分配对象的内存空间
  memory = allocate();
  //2：初始化对象
  ctorInstance(memory);  
  //3：设置instance指向刚分配的内存地址
  instance = memory;     
  ```

  但上面三行伪代码中的2和3之间，可能会被重排序

  ```java
  //1：分配对象的内存空间
  memory = allocate(); 
  //3：设置instance指向刚分配的内存地址,注意：此时对象还没有被初始化
  instance = memory;
  //2：初始化对象
  ctorInstance(memory);
  ```

  如果线程A由于指令重排，**在设置instance指向刚分配的内存地址但还未初始化对象时**，此时instance已经不为null了，正好线程B调用该方法，将会获得**一个未初始化完毕的单例**



* **正确的DCL单例模式代码**

  ```java
  public class Singleton {
  
      //如果不使用volatile，则instance = new Singleton();中指令重排序可能导致程序出问题
      private static volatile Singleton instance = null;
  
      private Singleton(){ }
  
      public static Singleton getInstance() {
          if(instance == null) {
              synchronized(Singleton.class) {
                  if(instance == null) {
                      instance = new Singleton();
                  }
              }
          }
          return instance;
      }
  
  }
  ```

  



**静态内部类实现单例：**

```java
public class SingletonDemo {
    private SingletonDemo() { }

    private static class SingletonDemoHandler {
        private static SingletonDemo instance = new SingletonDemo();
    }

    public static SingletonDemo getInstance() {
        return SingletonDemoHandler.instance;
    }
}
```

* 类加载本身就是懒惰的，在没有调用getInstance方法时是没有执行SingletonDemoHandler内部类的类加载操作的。`静态内部类不会随着外部类的加载而加载, 这是静态内部类和静态变量的区别`。

* 同时也不会有并发问题，因为是**通过类加载创建的单例, JVM保证不会出现线程安全**。



# 四、CAS

## 4.1 CAS与volatile

**乐观锁与悲观锁的概念：**

* **悲观锁：**

  悲观锁就是我们常说的锁。对于悲观锁来说，它总是认为每次访问共享资源时会发生冲突，所以必须对每次数据操作加上锁，以保证临界区的程序同一时间只能有一个线程在执行。

* **乐观锁：**

  乐观锁又称为“无锁”，顾名思义，它是乐观派。乐观锁总是假设对共享资源的访问没有冲突，线程可以不停地执行，无需加锁也无需等待。 乐观锁的一种实现方式 CAS 实现的 。



**CAS与volatile：**

* compare and swap的缩写，中文翻译成：比较并交换,实现并发算法时常用到的一种技术。它包含三个操作数——内存位置、预期原值及更新值。某线程执行CAS操作的时候，将内存位置的值与预期原值比较：

  * 如果相**匹配**，那么**处理器会自动将该位置值更新为新值**
  * 如果**不匹配**，那么重新获取该内存位置的值，然后线程进行自旋，到下次循环才有机会执行。

  ![image-20210823111940045](https://gitee.com/jobim/blogimage/raw/master/img/20210823111940.png)

* **CAS需要与volatile结合使用**，因为获取共享变量时，需要**保证该变量的可见性**



**CAS的缺点：**

* 如果竞争激烈，则会导致循环时间长，开销大（因为执行的是do while，如果比较不成功一直在循环，最差的情况，就是某个线程一直取到的值和预期值都不一样，这样就会无限循环）
* 只能保证一个共享变量的原子操作
  * 当对一个共享变量执行操作时，我们可以通过循环CAS的方式来保证原子操作
  * 但是对于多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候只能用锁来保证原子性

* ABA 问题 





注意：

* **CAS是原子性操作，它是由硬件进行的比较-更新的原子性。**

* CAS 的底层是 **lock cmpxchg 指令**（X86 架构），在单核 CPU 和多核 CPU 下都能够保证【比较-交换】的 原子性。
* 在多核状态下，某个核执行到带lock的指令时，**CPU 会让总线锁住，当这个核把此指令执行完毕，再开启总线。这个过程中不会被线程的调度机制所打断**，保证了多个线程对内存操作的准确性，是原子的。



**如何解决ABA问题？**

* **ABA问题介绍**：一个线程1从内存中取出A,这个时候另一个线程2也从内存中取出A，并且线程2进行了一些操作将值变成了B，线程1此时还被阻塞，线程2又进行了一些操作，然后将B又变成了A,此时线程1获得资源，开始执行，但是在进行cas操作的时候发现内存中还是A,然后线程1执行成功。（说白了就是可能存在一个线程根本不知道数值发生了变化）
* 使用`AtomicStampedReference`（加版本号解决ABA问题）





## 4.2 自旋锁（spinlock）

* 是指尝试获取锁的线程不会立即阻塞，而是**采用循环的方式去尝试获取锁**
* 当线程发现锁被占用时，会不断循环判断锁的状态，直到获取。这样的好处是减少线程上下文切换的消耗，缺点是循环会消耗CPU



**自己实现一个自旋锁：**

```java
public class SpinLock {
    AtomicReference<Thread> atomicReference = new AtomicReference<>();

    public void myLock() {
        Thread thread = Thread.currentThread();
        //把atomicReference中的null修改成thread
        while(!atomicReference.compareAndSet(null,thread)) {
            //上锁成功就结束循环
        }
    }

    public void myUnLock() {
        Thread thread = Thread.currentThread();
        atomicReference.compareAndSet(thread,null); //解锁，将atomicReference中的当前线程对象改成null
    }
}
```





## 4.3 Unsafe类

* 是CAS的核心类，由于Java方法无法直接访问底层系统，需要通过本地（native）方法来访问，Unsafe相当于一个后门，基于该类可以直接操作特定内存的数据。



* 可以看到AtomicInteger底层调用的就是**Unsafe类中的compareAndSwapInt()方法**：

  ![image-20210823113529358](https://gitee.com/jobim/blogimage/raw/master/img/20210823113529.png)

  ![image-20210823114402991](https://gitee.com/jobim/blogimage/raw/master/img/20210823114403.png)



# 五、原子操作类



## 5.1 基本类型原子类



* **AtomicBoolean**：以原子更新的方式更新boolean；
* **AtomicInteger**：以原子更新的方式更新Integer；
* **AtomicLong**：以原子更新的方式更新Long；



**常用的API**：（以AtomicInteger为例）

* addAndGet(int delta) ：以原子方式将输入的数值与实例中原本的值相加，并返回最后的结果；
* incrementAndGet() ：以原子的方式将实例中的原值进行加1操作，并返回最终相加后的结果；
* getAndSet(int newValue)：将实例中的值更新为新值，并返回旧值；
* getAndIncrement()：以原子的方式将实例中的原值加1，返回的是自增前的旧值；
* compareAndSet(int expect, int update) ：如果输入的数值等于预期值，则以原子方式将该值设置为输入值（update）



代码示例：

```java
class MyNumber{
    @Getter
    private AtomicInteger atomicInteger = new AtomicInteger();
    public void addPlusPlus(){
        atomicInteger.incrementAndGet();
    }
}
  
public class AtomicIntegerDemo{
    public static void main(String[] args) throws InterruptedException{
        MyNumber myNumber = new MyNumber();
        CountDownLatch countDownLatch = new CountDownLatch(100);

        for (int i = 1; i <=100; i++) {
            new Thread(() -> {
                try{
                    for (int j = 1; j <=5000; j++){
                        myNumber.addPlusPlus();
                    }
                }finally {
                    countDownLatch.countDown();
                }
            },String.valueOf(i)).start();
        }

        countDownLatch.await();

        System.out.println(myNumber.getAtomiInteger().get());
    }
}
```



## 5.2 数组类型原子类

* AtomicIntegerArray：原子更新整型数组中的元素；
* AtomicLongArray：原子更新长整型数组中的元素；
* AtomicReferenceArray：原子更新引用类型数组中的元素



**常用API**：（以AtomicIntegerArray）

* addAndGet(int i, int delta)：以原子更新的方式将数组中索引为i的元素与输入值相加；
* getAndIncrement(int i)：以原子更新的方式将数组中索引为i的元素自增加1；
* compareAndSet(int i, int expect, int update)：将数组中索引为i的位置的元素进行更新



代码示例：

```java
public class AtomicIntegerArrayDemo {
    public static void main(String[] args) {
        AtomicIntegerArray atomicIntegerArray = new AtomicIntegerArray(new int[5]);
        //AtomicIntegerArray atomicIntegerArray = new AtomicIntegerArray(5);
        //AtomicIntegerArray atomicIntegerArray = new AtomicIntegerArray(new int[]{1,2,3,4,5});

        int tmpInt = 0;

        tmpInt = atomicIntegerArray.getAndSet(0,1122);
        System.out.println(tmpInt+"\t"+atomicIntegerArray.get(0)); // 0	1122
        atomicIntegerArray.getAndIncrement(1);
        atomicIntegerArray.getAndIncrement(1);
        tmpInt = atomicIntegerArray.getAndIncrement(1);
        System.out.println(tmpInt+"\t"+atomicIntegerArray.get(1)); // 2	3
    }
}
```





## 5.3 引用类型原子类

* **AtomicReference**：原子更新引用类型
* **AtomicStampedReference**：携带版本号的引用类型原子类（可以解决ABA问题）
* **AtomicMarkableReference**：原子更新带有标记位的引用类型（用来解决变量是否被修改）



代码示例：（加版本号解决ABA问题）

```java
public class Test1 {
    //指定版本号
    static AtomicStampedReference<String> ref = new AtomicStampedReference<>("A", 0);

    public static void main(String[] args) {
        new Thread(() -> {
            String pre = ref.getReference();
            //获得版本号
            int stamp = ref.getStamp(); // 此时的版本号还是第一次获取的
            try {
                other();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            //把ref中的A改为C,并比对版本号，如果版本号相同，就执行替换，并让版本号+1
            System.out.println("change A->C stamp " + stamp + " " + ref.compareAndSet(pre, "C", stamp, stamp + 1));
        }).start();
    }

    static void other() throws InterruptedException {
        new Thread(() -> {
            int stamp = ref.getStamp();
            System.out.println("change A->B stamp " + stamp + " " + ref.compareAndSet(ref.getReference(), "B", stamp, stamp + 1));
        }).start();
        Thread.sleep(500);
        new Thread(() -> {
            int stamp = ref.getStamp();
            System.out.println("change B->A stamp " + stamp + " " + ref.compareAndSet(ref.getReference(), "A", stamp, stamp + 1));
        }).start();
    }
}
```



## 5.4 对象的属性修改原子类

* **AtomicIntegeFieldUpdater**：原子更新对象中int类型字段的值
* **AtomicLongFieldUpdater**：原子更新对象中Long类型字段的值
* **AtomicReferenceFieldUpdater**：原子更新引用类型里的字段



**使用步骤：**

1. 因为对象的属性修改类型原子类都是抽象类，所以每次使用都必须使用静态方法newUpdater()创建一个更新器，并且需要设置想要更新的类和属性。
2. 更新类的属性必须使用`public volatile`进行修饰；



代码示例：

```java
public class AtomicFieldTest {
    public static void main(String[] args) {
        Student stu = new Student();
        // 获得原子更新器
      	// 参数1 持有属性的类 参数2 被更新的属性的类
      	// newUpdater中的参数：第三个为属性的名称
        AtomicReferenceFieldUpdater updater = AtomicReferenceFieldUpdater.newUpdater(Student.class, String.class, "name");
        // 期望的为null, 如果name属性没有被别的线程更改过, 默认就为null, 此时匹配, 就可以设置name为张三
        System.out.println(updater.compareAndSet(stu, null, "张三")); // true
        System.out.println(updater.compareAndSet(stu, stu.name, "王五")); // true
        System.out.println(stu); // Student{name='王五'}
    }
}

class Student {
    public volatile String name;

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                '}';
    }
}
```





## 5.5 原子操作增强类（重点）

* LongAdder：LongAdder只能用来计算加法，且从零开始计算
* LongAccumulator：LongAccumulator提供了自定义的函数操作
* DoubleAdder
* DoubleAccumulator



**常用API：**

* add()：将当前value加x
* increment()：将当前的value加1
* decrement()：将当前的value减1
* sum()：放回当前值。注意：在没有并发更新value的情况下，sum会返回一个精确值，在存在并发的情况下，sum不保证返回精确值。
* reset()：将value重置为0，可用来代替重新new一个LongAdder，单词方法只可以在没有并发更新的情况下使用。





代码示例：

```java
public class LongAdderAPIDemo {
    public static void main(String[] args) {
        LongAdder longAdder = new LongAdder();

        longAdder.increment();
        longAdder.increment();
        longAdder.increment();

        System.out.println(longAdder.longValue());
        
        //初始化的时候传入自定义函数操作
        LongAccumulator longAccumulator = new LongAccumulator((x, y) -> x * y,2);

        longAccumulator.accumulate(1);
        longAccumulator.accumulate(2);
        longAccumulator.accumulate(3);

        System.out.println(longAccumulator.longValue());

    }
}
```





## 5.6 LongAdder原理 （待补）

- **LongAdder的基本思路就是分散热点，将value值分散到一个Cell数组中，不同线程会命中到数组的不同槽中，各个线程只对自己槽中的那个值进行CAS操作**
- **在无竞争的情况，跟AtomicLong一样，对同一个base进行操作。**
- **而在有竞争时，设置多个`累加单元`(但不会超过cpu的核心数)（数组cells），对线程id进行hash得到hash值，再根据hash值映射到这个数组cells的某个下标，再对该下标所对应的值进行自增操作。这样它们在累加时操作的不同的 Cell 变量，`因此减少了 CAS 重试失败`，从而提高性能。当所有线程操作完毕，将数组cells的所有值和无竞争值base都加起来作为最终结果。**





尚硅谷第8天的视频





# 六、线程池



## 6.1 ThreadPoolExecutor

**线程池有什么优点？**

- **降低资源消耗**。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
- **提高响应速度**。当任务到达时，任务可以不需要等到线程创建就能立即执行。
- **提高线程的可管理性**。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控，也可以延时执行、定时循环执行的策略



**线程池的继承关系：**

![image-20210824111417034](https://gitee.com/jobim/blogimage/raw/master/img/20210824111417.png)

线程池的真正实现类是：`ThreadPoolExecutor`





**线程池的5种状态：**

- **ThreadPoolExecutor 使用 int 的高 3 位来表示线程池状态，低 29 位表示线程数量**

  | 状态名称   | 高3位的值 | 描述                                          |
  | ---------- | --------- | --------------------------------------------- |
  | RUNNING    | 111       | 接收新任务，同时处理任务队列中的任务          |
  | SHUTDOWN   | 000       | 不接受新任务，但是处理任务队列中的任务        |
  | STOP       | 001       | 中断正在执行的任务，同时抛弃阻塞队列中的任务  |
  | TIDYING    | 010       | 任务执行完毕，活动线程为0时，即将进入终结阶段 |
  | TERMINATED | 011       | 终结状态                                      |



- 使用一个数来表示两个值的主要原因是：**可以通过一次CAS同时更改两个属性的值**

- 这些信息存储在一个原子变量 ctl 中，目的是将线程池状态与线程个数合二为一，这样就可以用一次 cas 原子操作进行赋值

  ```java
  // c 为旧值， ctlOf 返回结果为新值
  ctl.compareAndSet(c, ctlOf(targetState, workerCountOf(c))));
  
  // rs 为高 3 位代表线程池状态， wc 为低 29 位代表线程个数，ctl 是合并它们
  private static int ctlOf(int rs, int wc) { return rs | wc; }
  ```






<hr/>

**线程池的常用方法：**



* **void execute(Runnable command)** ：执行任务

* **&lt;T> Future&lt;T> submit(Callable&lt;T> task)**：提交任务 task，用返回值 Future 获得任务执行结果，Future的原理就是利用我们之前讲到的保护性暂停模式来接受返回结果的，主线程可以执行 FutureTask.get()方法来等待任务执行完成

* **shutdown()：关闭线程池**
  * **将线程池的状态改为 SHUTDOWN**
  * **不再接受新任务，但是会将阻塞队列中的任务执行完**

* **shutdownNow()：关闭线程池**
  * **将线程池的状态改为 STOP**
  * **不再接受新任务，也不会再执行阻塞队列中的任务**



### 6.1.1 ThreadPoolExecutor构造方法参数解析

**ThreadPoolExecutor构造方法：**

* 常用的构造方法

  ![image-20210824112721119](https://gitee.com/jobim/blogimage/raw/master/img/20210824112721.png)



* 最全的构造方法

  ```java
  public ThreadPoolExecutor(int corePoolSize,
                            int maximumPoolSize,
                            long keepAliveTime,
                            TimeUnit unit,
                            BlockingQueue<Runnable> workQueue,
                            ThreadFactory threadFactory,
                            RejectedExecutionHandler handler)
  ```

  

**ThreadPoolExecutor七大参数：** 

- **`corePoolSize`**：核心线程数，线程池中的常驻核心线程数。
- **`maximumPoolSize`**：线程池能够容纳同时执行的最大线程数。
- **`keepAliveTime`**：多余的空闲线程存活时间
  - 当线程池数量超过corePoolSize时，当空闲时间达到keepAliveTime值时，多余的空闲线程会被销毁，直到只剩下corePoolSize个线程为止
  - 默认情况下，只有当线程池中的线程数大于corePoolSize时，keepAliveTime才会起作用
- **`unit`**：keepAliveTime的单位
- **`workQueue`**：任务队列，被提交的但未被执行的任务（类似于银行里面的候客区）
  - LinkedBlockingQueue：链表阻塞队列
  - SynchronousBlockingQueue：同步阻塞队列
  -  SynchronousQueue：最多只有一个同步元素
  -  PriorityBlockingQueue：优先队列
- threadFactory：线程工厂（给线程取名字）
- **`handler`**：拒绝策略，表示当队列满了并且工作线程大于线程池的最大线程数（maximumPoolSize）时的拒绝策略。



**线程池大小确定**

- **CPU 密集型任务(N+1)：** 这种任务消耗的主要是 CPU 资源，**可以将线程数设置为 N（CPU 核心数）+1**，比 CPU 核心数多出来的一个线程是为了防止线程偶发的缺页中断，或者其它原因导致的任务暂停而带来的影响。一旦任务暂停，CPU 就会处于空闲状态，而在这种情况下多出来的一个线程就可以充分利用 CPU 的空闲时间。
- **I/O 密集型任务(2N)：** 这种任务应用起来，系统会用大部分的时间来处理 I/O 交互，而线程在处理 I/O 的时间段内不会占用 CPU 来处理，这时就可以将 CPU 交出给其它线程使用。因此在 I/O 密集型任务的应用中，我们**可以多配置一些线程**，具体的计算方法是 2N



### 6.1.2 拒绝策略

- 如果线程到达 maximumPoolSize 仍然有新任务这时会执行**拒绝策略**。拒绝策略 jdk 提供了 4 种实现

  ![image-20210824114610005](https://gitee.com/jobim/blogimage/raw/master/img/20210824114610.png)

`AbortPolicy 中止策略`：丢弃任务并抛出RejectedExecutionException异常。**这是默认策略**

`DiscardPolicy 丢弃策略`：放弃任务，但是不抛出异常。如果线程队列已满，则后续提交的任务都会被丢弃，且是静默丢弃。

`DiscardOldestPolicy 弃老策略`：丢弃队列最前面的任务，然后重新提交被拒绝的任务。

`CallerRunsPolicy 调用者运行策略`：由调用者运行该任务



### 6.1.3 任务队列

该线程池中的任务队列：维护着等待执行的Runnable对象

当所有的核心线程都在干活时，新添加的任务会被添加到这个队列中等待处理，如果队列满了，则新建非核心线程执行任务



**常用的workQueue类型：**

* **SynchronousQueue：**
  * **没有容量**，没有线程来取是放不进去的
  * **只有当线程取任务时，才会将任务放入该阻塞队列中**
  * 由于该队列没有容量，所以一般使用这个类型队列的时候，maximumPoolSize一般指定成Integer.MAX_VALUE，即无限大

* **LinkedBlockingQueue：**这个队列接收到任务的时候，如果当前线程数小于核心线程数，则新建线程(核心线程)处理任务；如果当前线程数等于核心线程数，则进入队列等待。由于这个队列没有最大值限制，即所有超过核心线程数的任务都将被添加到队列中，这也就导致了maximumPoolSize的设定失效，因为总线程数永远不会超过corePoolSize

* **ArrayBlockingQueue：**可以限定队列的长度，接收到任务的时候，如果没有达到corePoolSize的值，则新建线程(核心线程)执行任务，如果达到了，则入队等候，如果队列已满，则新建线程(非核心线程)执行任务。





## 6.2 线程池的处理流程

![image-20210824113358937](https://gitee.com/jobim/blogimage/raw/master/img/20210824113359.png)

1. 在创建了线程池后，等待提交过来的任务请求
2. 当调用execute()方法添加一个请求任务时，线程池会做出如下判断
   1. 如果**正在运行的线程池数量小于corePoolSize并且没有空闲线程，那么马上创建线程运行这个任务**
   2. 如果**正在运行的线程数量达到corePoolSize，那么将这个任务放入队列，直到有空闲的线程**
   3. 如果这时候**队列满了，并且正在运行的线程数量还小于maximumPoolSize，那么还是创建非核心线程**来运行这个任务；
     4. 如果**队列满了并且正在运行的线程数量达到maximumPoolSize，那么线程池会启动拒绝策略来执行**
3. 当一个线程完成任务时，它会从队列中取下一个任务来执行
4. 当高峰过去后，**超过corePoolSize 的非核心线程如果一段时间没有任务做，需要结束节省资源**，这个时间由keepAliveTime 和 unit 来控制



<font color='blue' size='4'>**为什么当线程池的核心线程满了后，是先加入到阻塞队列，而不是先创建新的线程？**</font>

* 线程池创建线程需要获取mainlock这个全局锁，会影响并发效率，所以使用阻塞队列把第一步创建核心线程与第三步创建最大线程隔离开来，起一个缓冲的作用

* 大于corePoolSize的线程在没工作的时候，超时是会被销毁的

 

## 6.3 常见四种线程池

如果不想通过ThreadPoolExecutor构造方法新建线程池，Java通过**Executors工具类**提供了四种线程池，这四种线程池都是直接或间接配置ThreadPoolExecutor的参数实现的



- **newFixedThreadPool** ： **该方法返回一个固定线程数量的线程池**。

  内部调用的构造方法

  ```java
  public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
      return new ThreadPoolExecutor(nThreads, nThreads,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>(),
                                    threadFactory);
  }
  ```

  **特点**

  1. **核心线程数 == 最大线程数**（没有救急线程被创建），因此也无需超时时间
  2. **阻塞队列是无界的，可以放任意数量的任务**
  3. **适用于任务量已知，相对耗时的任务**



- **newCachedThreadPool：** 该方法**返回一个可根据实际情况调整线程数量的线程池**。

  内部构造方法：

  ```java
  public static ExecutorService newCachedThreadPool() {
      return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                    60L, TimeUnit.SECONDS,
                                    new SynchronousQueue<Runnable>());
  }
  ```

  **特点：**

  - **没有核心线程**，最大线程数为`Integer.MAX_VALUE`，**所有创建的线程都是救急线程 (`可以无限创建`)**，空闲时生存时间为60秒
  - 阻塞队列使用的是SynchronousQueue
    - SynchronousQueue是一种特殊的队列
      - **没有容量**，没有线程来取是放不进去的
      - 只有当线程取任务时，才会将任务放入该阻塞队列中
  - **整个线程池表现为线程数会根据任务量不断增长，没有上限，当任务执行完毕，空闲 1分钟后释放线程。 适合任务数比较密集，但每个任务执行时间较短的情况**







- **newSingleThreadExecutor：** **方法返回一个只有一个线程的线程池**。若多余一个任务被提交到该线程池，任务会被保存在一个任务队列中，待线程空闲，按先入先出的顺序执行队列中的任务。

  内部构造方法：

  ```java
  public static ExecutorService newSingleThreadExecutor() {
      return new FinalizableDelegatedExecutorService
          (new ThreadPoolExecutor(1, 1,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>()));
  }
  ```

  **使用场景：**

  1. **希望多个任务排队执行。线程数固定为 1，任务数多于 1 时，会放入无界队列排队。** 任务执行完毕，这唯一的线程也不会被释放。
  2. 区别：
     1. `和自己创建单线程执行任务的区别`：**自己创建一个单线程串行执行任务，如果任务执行失败而终止那么没有任何补救措施，而newSingleThreadExecutor线程池还会新建一个线程，保证池的正常工作**
     2. Executors.newSingleThreadExecutor() 线程个数始终为1，不能修改,FinalizableDelegatedExecutorService 应用的是装饰器模式，只对外暴露了 ExecutorService 接口，因此不能调用 ThreadPoolExecutor 中特有的方法
     3. 和Executors.newFixedThreadPool(1) 初始时为1时的区别：Executors.newFixedThreadPool(1) 初始时为1，以后还可以修改，对外暴露的是 ThreadPoolExecutor 对象，可以强转后调用 setCorePoolSize 等方法进行修改



* ScheduledThreadPool()：支持定时及周期性任务执行的线程池



 **Executors 返回线程池对象的弊端如下：** 

> ![image-20210824121945230](https://gitee.com/jobim/blogimage/raw/master/img/20210824121945.png)
>
> 建议使用`ThreadPoolExecutor`来创建线程



## 6.4 自定义线程池



```java
/**
 * Description: 自定义一个简单的线程池
 *
 * @author guizy
 * @date 2020/12/30 20:47
 */

@Slf4j(topic = "guizy.TestPool")
public class TestPool {
    public static void main(String[] args) {
        ThreadPool threadPool = new ThreadPool(1, 1000, TimeUnit.MILLISECONDS, 1, new RejectPolicy<Runnable>() {
            @Override
            public void reject(BlockingQueue<Runnable> queue, Runnable task) {
                // 拒绝策略
                // 1、死等
                // queue.put(task);

                // 2、带超时等待
                queue.offer(task, 2000, TimeUnit.MILLISECONDS);

                // 3、让调用者放弃任务执行
                // log.debug("放弃-{}", task);

                // 4、让调用者抛弃异常
                // throw new RuntimeException("任务执行失败" + task);

                // 5、让调用者自己执行任务
                // task.run();
            }
        });
        // 创建5个任务
        for (int i = 0; i < 4; i++) {
            int j = i;
            threadPool.execute(new Runnable() {
                @Override
                public void run() {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    log.debug("{}", j);
                }
            });
        }
    }
}

@FunctionalInterface
interface RejectPolicy<T> {
    void reject(BlockingQueue<T> queue, T task);
}


/**
 * 线程池
 */
@Slf4j(topic = "guizy.TestPool")
class ThreadPool {

    // 阻塞任务队列
    private BlockingQueue<Runnable> taskQueue;

    // 线程集合
    private HashSet<Worker> workers = new HashSet<>();

    // 核心线程数
    private int coreSize;

    // 获取任务的超时时间
    private long timeout;

    private TimeUnit timeUnit;

    // 拒绝策略
    private RejectPolicy<Runnable> rejectPolicy;

    public ThreadPool(int coreSize, long timeout, TimeUnit timeUnit, int queueCapacity, RejectPolicy<Runnable> rejectPolicy) {
        this.coreSize = coreSize;
        this.timeout = timeout;
        this.timeUnit = timeUnit;
        this.taskQueue = new BlockingQueue<>(queueCapacity);
        this.rejectPolicy = rejectPolicy;
    }

    // 执行任务
    public void execute(Runnable task) {
        synchronized (workers) {
            // 当任务没有超过线程数, 说明当前worker线程可以消费这些任务, 不用将任务加入到阻塞队列中
            if (workers.size() < coreSize) {
                Worker worker = new Worker(task);
                log.debug("新增 worker {}, {}", worker, task);
                workers.add(worker);
                worker.start();
            } else {
                // taskQueue.put(task); // 这一种死等
                // 拒绝策略
                // 1、死等
                // 2、带超时等待
                // 3、让调用者放弃任务执行
                // 4、让调用者抛弃异常
                // 5、让调用者自己执行任务
                taskQueue.tryPut(rejectPolicy, task);

            }
        }
    }

    class Worker extends Thread {
        private Runnable task;

        public Worker(Runnable task) {
            this.task = task;
        }

        @Override
        public void run() {
            // 执行任务
            // 1): 当task不为空, 执行任务
            // 2): 当task执行完毕, 从阻塞队列中获取任务并执行
            //while (task != null || (task = taskQueue.take()) != null) {
            while (task != null || (task = taskQueue.poll(timeout, timeUnit)) != null) {
                try {
                    log.debug("正在执行...{}", task);
                    task.run();
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    task = null;
                }
            }

            // 将线程集合中的线程移除
            synchronized (workers) {
                log.debug("worker被移除 {}", this);
                workers.remove(this);
            }
        }
    }

}

/**
 * 用于存放任务的阻塞队列
 *
 * @param <T> Runnable, 任务抽象为Runnable
 */
@Slf4j(topic = "guizy.TestPool")
class BlockingQueue<T> {
    // 1、任务队列
    private Deque<T> queue = new ArrayDeque<>();

    // 2、锁
    private ReentrantLock lock = new ReentrantLock();

    // 3、生产者的条件变量 (当阻塞队列塞满任务的时候, 没有空间, 此时进入条件变量中等待)
    private Condition fullWaitSet = lock.newCondition();

    // 4、消费者的条件变量 (当没有任务可以消费的时候, 进入条件变量中等待)
    private Condition emptyWaitSet = lock.newCondition();

    // 5、阻塞队列的容量
    private int capacity;

    public BlockingQueue(int capacity) {
        this.capacity = capacity;
    }


    // 从阻塞队列中获取任务, 如果没有任务, 会等待指定的时间
    public T poll(long timeout, TimeUnit unit) {
        lock.lock();
        try {
            // 将timeout统一转换为纳秒
            long nanos = unit.toNanos(timeout);
            while (queue.isEmpty()) {
                try {
                    // 表示超时, 无需等待, 直接返回null
                    if (nanos <= 0) {
                        return null;
                    }
                    // 返回值的时间(剩余时间) = 等待时间 - 经过时间 所以不存在虚假唤醒(时间还没等够就被唤醒,然后又从新等待相同时间)
                    nanos = emptyWaitSet.awaitNanos(nanos);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            T t = queue.removeFirst();
            fullWaitSet.signal(); // 唤醒生产者进行生产, 此时阻塞队列没有满
            return t;
        } finally {
            lock.unlock();
        }
    }

    // 从阻塞队列中获取任务, 如果没有任务,会一直等待
    public T take() {
        lock.lock();
        try {
            // 阻塞队列是否为空
            while (queue.isEmpty()) {
                // 进入消费者的条件变量中等待,此时没有任务供消费
                try {
                    emptyWaitSet.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            // 阻塞队列不为空, 获取队列头部任务
            T t = queue.removeFirst();
            fullWaitSet.signal(); // 唤醒生产者进行生产, 此时阻塞队列没有满
            return t;
        } finally {
            lock.unlock();
        }
    }

    // 往阻塞队列中添加任务
    public void put(T task) {
        lock.lock();
        try {
            // 阻塞队列是否满了
            while (queue.size() == capacity) {
                try {
                    log.debug("等待进入阻塞队列...");
                    fullWaitSet.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            queue.addLast(task);
            log.debug("加入任务阻塞队列 {}", task);
            emptyWaitSet.signal(); // 此时阻塞队列中有任务了, 唤醒消费者进行消费任务
        } finally {
            lock.unlock();
        }
    }

    // 往阻塞队列中添加任务(带超时)
    public boolean offer(T task, long timeout, TimeUnit timeUnit) {
        lock.lock();
        try {
            long nanos = timeUnit.toNanos(timeout);
            while (queue.size() == capacity) {
                try {
                    if (nanos <= 0) {
                        return false;
                    }
                    log.debug("等待进入阻塞队列 {}...", task);
                    nanos = fullWaitSet.awaitNanos(nanos);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            log.debug("加入任务阻塞队列 {}", task);
            queue.addLast(task);
            emptyWaitSet.signal(); // 此时阻塞队列中有任务了, 唤醒消费者进行消费任务
            return true;
        } finally {
            lock.unlock();
        }
    }

    // 获取队列大小
    public int size() {
        lock.lock();
        try {
            return queue.size();
        } finally {
            lock.unlock();
        }
    }

    public void tryPut(RejectPolicy<T> rejectPolicy, T task) {
        lock.lock();
        try {
            // 判断队列是否满
            if (queue.size() == capacity) {
                rejectPolicy.reject(this, task);
            } else {
                // 有空闲
                log.debug("加入任务队列 {}", task);
                queue.addLast(task);
                emptyWaitSet.signal();
            }
        } finally {
            lock.unlock();
        }
    }
}
```



- 阻塞队列BlockingQueue用于暂存来不及被线程执行的任务
  - 也可以说是平衡生产者和消费者执行速度上的差异
  - 里面的获取任务和放入任务用到了 **`生产者消费者模式`**

- 线程池中对线程Thread进行了再次的封装，封装为了Worker
  - 在调用 **任务对象 (Runnable、Callable)** 的run方法时，线程会去执行该任务，执行完毕后还会**到阻塞队列中获取新任务来执行**

- 线程池中执行任务的主要方法为`execute`方法
  - 执行时要判断正在执行的线程数是否大于了线程池容量



