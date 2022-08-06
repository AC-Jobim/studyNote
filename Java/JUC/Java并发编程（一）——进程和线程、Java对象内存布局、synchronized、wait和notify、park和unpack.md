# 一、进程和线程



<font size='5' color='red' >**进程和线程的区别？**</font>

**进程**：进程是程序的一次执行过程。是CPU资源分配的最小单位。每个进程都有自己独立的一块内存空间，一个进程可以有多个线程

**线程**：线程是CPU调度的最小单位，它可以和属于同一个进程的其他线程共享这个进程的全部资源



[进程和线程的区别](https://blog.csdn.net/ThinkWon/article/details/102021274)

* **根本区别**：进程是操作系统资源分配的基本单位，而线程是处理器任务调度和执行的基本单位

* **资源开销**：每个进程都有独立的代码和数据空间（程序上下文），程序之间的切换会有较大的开销；线程可以看做轻量级的进程，同一类线程共享**堆和方法区（1.8 转到直接内存的元空间）**，每个线程都有自己独立的**程序计数器、虚拟机栈和本地方法栈**，线程之间切换的开销小。

* **包含关系**：一般一个进程内有多个线程，执行过程不是一条线的，而是多条线（线程）共同完成的；线程是进程的一部分，所以线程也被称为轻权进程或者轻量级进程。

* **影响关系**：一个进程崩溃后，在保护模式下不会对其他进程产生影响。但是一个线程崩溃可能导致整个进程都死掉。所以多进程要比多线程健壮。



**线程私有的：**

- 程序计数器
- 虚拟机栈
- 本地方法栈

**线程共享的：**

- 堆
- 方法区（1.8 转到直接内存的元空间）
- 直接内存 (非运行时数据区的一部分)





<font size='5' color='red' >**并行和并发有什么区别？**</font>

- 并行是指两个或者多个事件在**同一时刻发生**
- 并发是指两个或多个事件在**同一时间间隔发生**



# 二、Java线程

## 2.1 创建线程的四种方式

1. 创建**继承于Thread类**的子类，并重写Thread类的run()方法

2. 创建一个**实现了Runnable接口**的类，并实现run()方法

3. 通过**Callable和FutureTask创建线程**

   1. 创建一个实现Callable的实现类，并实现call方法
   2. 将Callable接口实现类的对象作为传递到FutureTask构造器中，创建FutureTask的对象
   3. 将FutureTask的对象作为参数传递到Thread类的构造器中，创建Thread对象，并调用start()
   4. 调用FutureTask对象的get()方法来获得子线程执行结束后的返回值 

   ```java
   @Test
   public void test03() throws ExecutionException, InterruptedException {
       // 实现多线程的第三种方法可以返回数据
       FutureTask futureTask = new FutureTask<>(new Callable<Integer>() {
           @Override
           public Integer call() throws Exception {
               log.debug("多线程任务");
               Thread.sleep(100);
               return 100;
           }
       });
       // 主线程阻塞，同步等待 task 执行完毕的结果
       new Thread(futureTask,"分线程").start();
       log.debug("主线程");
       log.debug("{}",futureTask.get()); //获得分线程的返回值，get方法为阻塞方法
   }
   ```
   
4. 使用**线程池**

   ```java
   class NumberThread implements Runnable{
       @Override
       public void run() {
           for(int i = 0;i<10;i++){
               System.out.println(Thread.currentThread().getName()+":"+i);
           }
       }
   }
   
   class Number2Thread implements Callable {
       @Override
       public Object call() throws Exception {
           int sum = 0;
           for(int i = 1;i<=10;i++){
               System.out.println(Thread.currentThread().getName()+":"+i);
               sum+=i;
           }
           return sum;
       }
   }
   
   public class ThreadPool {
       public static void main(String[] args) {
           //1. 提供指定线程数量的线程池
           ExecutorService service = Executors.newFixedThreadPool(10);//创建一个可重用固定线程数为10的线程池
   
           //查看该对象是哪个类造的
           System.out.println(service.getClass());//class java.util.concurrent.ThreadPoolExecutor
           //设置线程池的属性
   //        service1.setCorePoolSize(15);
   //        service1.setKeepAliveTime();
   
           //2.执行指定的线程的操作。需要提供实现Runnable接口或Callable接口实现类的对象
           service.execute(new NumberThread());//适合使用于Runnable
           Future future = service.submit(new Number2Thread());//适合使用于Callable
           try {
               System.out.println(future.get());//输出返回值
           } catch (InterruptedException e) {
               e.printStackTrace();
           } catch (ExecutionException e) {
               e.printStackTrace();
           }
           //3.关闭连接池
           service.shutdown();
       }
   }
   ```

   



<font size='5' color='red' >**runnable 和 callable 有什么区别？**</font>

* 相同点
  * 都是接口
  * 都可以编写多线程程序
  * 都采用Thread.start()启动线程

* 主要区别
  * Runnable 接口 **run 方法无返回值**；Callable 接口 **call 方法有返回值，支持泛型**，和Future、FutureTask配合**可以用来获取异步执行的结果**
  * Runnable 接口 run 方法只能抛出运行时异常，且无法捕获处理；Callable 接口 call 方法**允许抛出异常，可以获取异常信息**
    注：Callalbe接口支持返回执行结果，需要调用FutureTask.get()得到，此方法会阻塞主进程的继续往下执行，如果不调用不会阻塞。



<font color='red' size='5' >**线程的 run()和 start()有什么区别？**</font>

* start() 方法用于启动线程，run() 方法用于执行线程的运行时代码。run() 可以重复调用，而 start() 只能调用一次。  多次调用会抛出 java.lang.IllegalThreadStateException 异常 

* new 一个 Thread，线程进入了新建状态。调用 **start() 方法**，会**启动一个线程并使线程进入了就绪状态**，当分配到时间片后就可以开始运行了。 start() 会执行线程的相应准备工作，然后自动执行 run() 方法的内容，这是真正的多线程工作。

* 而直接执行 run() 方法，会把 run 方法当成一个 **main 线程下的普通方法**去执行，并不会在某个线程中执行它，所以这并不是多线程工作。



## 2.2 线程的生命周期

![1617677433974](https://gitee.com/jobim/blogimage/raw/master/img/20210819122428.png)



1. **新建(new)**：新创建了一个线程对象。

2. **就绪(runnable)**：线程对象创建后，当调用线程对象的 start()方法，该线程处于就绪状态，等待被线程调度选中，获取cpu的使用权。

3. **运行(running)**：可运行状态(runnable)的线程获得了cpu时间片（timeslice），执行程序代码。注：就绪状态是进入到运行状态的唯一入口，也就是说，线程要想进入运行状态执行，首先必须处于就绪状态中；

4. **阻塞(block)**：处于运行状态中的线程由于某种原因，暂时放弃对 CPU的使用权，停止执行，此时进入阻塞状态，直到其进入到就绪状态，才有机会再次被 CPU 调用以进入到运行状态。

   阻塞的情况分三种：

   1. 等待阻塞：运行状态中的线程执行 wait()方法，JVM会把该线程放入等待队列(waitting queue)中，使本线程进入到等待阻塞状态；
   2. 同步阻塞：线程在获取 synchronized 同步锁失败(因为锁被其它线程所占用)，，则JVM会把该线程放入锁池(lock pool)中，线程会进入同步阻塞状态；
   3. 其他阻塞: 通过调用线程的 sleep()或 join()或发出了 I/O 请求时，线程会进入到阻塞状态。当 sleep()状态超时、join()等待线程终止或者超时、或者 I/O 处理完毕时，线程重新转入就绪状态。

5. **死亡(dead)**：线程run()、main()方法执行结束，或者因异常退出了run()方法，则该线程结束生命周期。死亡的线程不可再次复生。





**线程的六种状态：**

- 这是从 Java API 层面来描述的。根据`Thread.State 枚举，分为六种状态`



![image-20210819144942490](https://gitee.com/jobim/blogimage/raw/master/img/20210819144942.png)



- **`NEW (新建状态)`** 线程刚被创建，但是还没有调用 start() 方法

- **`RUNNABLE (运行状态)`** 当调用了 `start() 方法之后`，注意，Java API 层面的`RUNNABLE 状态涵盖了操作系统层面的 【就绪状态】、【运行中状态】和【阻塞状态】`（由于 BIO 导致的线程阻塞，在 Java 里无法区分，仍然认为 是可运行）
- **`BLOCKED (阻塞状态)` ， `WAITING (等待状态)` ， `TIMED_WAITING(定时等待状态)`** 都是 Java API 层面对【阻塞状态】的细分，如**sleep**就位**TIMED_WAITING**， **join**为**WAITING**状态。
- **`TERMINATED (结束状态)`** 当线程代码运行结束



## 2.3 线程的状态转换（API层次）





![image-20210821005907739](https://gitee.com/jobim/blogimage/raw/master/img/20210821005907.png)





假设有线程 `Thread t`

- 情况1：`NEW –> RUNNABLE`
  - 当调用`t.start()`方法时, `NEW --> RUNNABLE`
- 情况2：`RUNNABLE <–> WAITING`
  - t线程用synchronized(obj)获取了对象锁后
    - 调用 `obj.wait()`方法时，t 线程进入`waitSet`中, 从`RUNNABLE --> WAITING`
    - 调用`obj.notify()`，`obj.notifyAll()`，`t.interrupt()`时, 唤醒的线程都到entrySet阻塞队列成为BLOCKED状态, 在阻塞队列,和其他线程再进行竞争锁
      - **竞争锁成功**，t 线程从 `WAITING --> RUNNABLE`
      - **竞争锁失败**，t 线程从 `WAITING --> BLOCKED`
- 情况3：`RUNNABLE <–> WAITING`
  - **当前线程**调用 **`t.join()`** 方法时，**当前线程**从 `RUNNABLE --> WAITING` 
    - 注意是**当前线程**在t线程对象在**waitSet**上等待
  - **t 线程运行结束，或调用了当前线程的 interrupt() 时**，**当前线程**从 `WAITING --> RUNNABLE`
- 情况4：`RUNNABLE <–> WAITING`
  - 当前线程调用 **`LockSupport.park()`** 方法会让**当前线程**从`RUNNABLE --> WAITING`
  - 调用 **`LockSupport.unpark(目标线程)`** 或调用了线程 的 **interrupt()** ，会让目标线程从 `WAITING --> RUNNABLE`

- 情况5：`RUNNABLE <–> TIMED_WAITING `(带超时时间的wait)
  - t 线程用`synchronized(obj)`获取了对象锁后
    - 调用 **`obj.wait(long n)`** 方法时，t 线程从 `RUNNABLE --> TIMED_WAITING`
    - t 线程等待时间超过了 n 毫秒，或调用 obj.notify() ， obj.notifyAll() ， t.interrupt() 时; 唤醒的线程都到entrySet阻塞队列成为BLOCKED状态, 在阻塞队列,和其他线程再进行竞争锁
      - 竞争锁成功，t 线程从 **TIMED_WAITING --> RUNNABLE**
      - 竞争锁失败，t 线程从 **TIMED_WAITING --> BLOCKED**
- 情况6：`RUNNABLE <–> TIMED_WAITING`
  - 当前线程调用 **`t.join(long n)`** 方法时，当前线程从 `RUNNABLE --> TIMED_WAITING` 注意是当前线程在t 线程对象的**waitSet**等待
  - 当前线程**等待时间超过了 n 毫秒，或t 线程运行结束，或调用了当前线程的 interrupt() 时**，当前线程从 `TIMED_WAITING --> RUNNABLE`
- 情况7：`RUNNABLE <–> TIMED_WAITING`
  - 当前线程调用 `Thread.sleep(long n)` ，当前线程从 `RUNNABLE --> TIMED_WAITING`
  - 当前线程**等待时间超过了 n 毫秒或调用了线程的 interrupt()** ，当前线程从 `TIMED_WAITING --> RUNNABLE`
- 情况8：`RUNNABLE <–> TIMED_WAITING`
  - 当前线程调用 `LockSupport.parkNanos(long nanos) 或 LockSupport.parkUntil(long millis)` 时，当前线程从 `RUNNABLE --> TIMED_WAITING`
  - 调用**LockSupport.unpark(目标线程) 或调用了线程的interrupt() ，或是等待超时**，会让目标线程从 `TIMED_WAITING--> RUNNABLE`
- 情况9：`RUNNABLE <–> BLOCKED`
  - t 线程用 `synchronized(obj)` 获取了对象锁时如果竞争失败，从 `RUNNABLE –> BLOCKED`
  - 持 obj 锁线程的同步代码块执行完毕，会唤醒该对象上所有 BLOCKED 的线程重新竞争，如果其中 **t 线程竞争 成功**，从 `BLOCKED –> RUNNABLE` ，其它失败的线程仍然 BLOCKED
- 情况10：`RUNNABLE –> TERMINATED`
  - 当前线程所有代码运行完毕，进入 TERMINATED



![image-20210821005839779](https://gitee.com/jobim/blogimage/raw/master/img/20210821005840.png)



## 2.4 线程运行原理

**虚拟机栈与栈帧**

* `虚拟机栈`描述的是`Java方法执行的内存模型`：**每个方法被执行的时候**都会同时创建一个`栈帧(stack frame)`用于存储`局部变量表、操作数栈、动态链接、方法出口`等信息，是属于**线程的私有的**。当Java中使用多线程时，每个线程都会维护它自己的栈帧！每个线程只能有一个活动栈帧(在栈顶)，对应着当前正在执行的那个方法

**线程上下文切换（Thread Context Switch）**

> 因为以下一些原因导致 cpu 不再执行当前的线程，转而执行另一个线程的代码

- 线程的 cpu 时间片用完(每个线程轮流执行，看前面并行的概念)
- 垃圾回收
- 有更高优先级的线程需要运行
- 线程自己调用了 `sleep`、`yield`、`wait`、`join`、`park`、`synchronized`、`lock` 等方法

当`Thread Context Switch`发生时，需要由操作系统`保存当前线程的状态`，并`恢复另一个线程的状态`，Java 中对应的概念就是<font color='red'>**程序计数器**</font>（Program Counter Register），它的作用是记录当前线程执行字节码的位置，存储的是字节码指令地址

- `线程的状态`包括程序计数器、虚拟机栈中每个栈帧的信息，如局部变量、操作数栈、返回地址等
- Context Switch 频繁发生会`影响性能`



## 2.5 守护线程

* 在Java中有两类线程：User Thread(用户线程)、Daemon Thread(守护线程) 。用户线程一般用户执行用户级任务，而守护线程也就是“**后台线程**”，一般用来执行后台任务。

* **守护线程**，是指在程序运行的时候在后台提供一种通用服务的线程

- 当`Java进程`中有`多个线程`在执行时，**只有当所有非守护线程都执行完毕后，Java进程才会结束**。但当非守护线程全部执行完毕后，`守护线程无论是否执行完毕，也会一同结束。`普通线程t1可以调用`t1.setDeamon(true);` 方法变成守护线程 

> 注意：
>
> - `垃圾回收器线程`就是一种守护线程
> - Tomcat 中的 `Acceptor 和 Poller 线程`都是守护线程，所以 Tomcat 接收到 shutdown 命令后，不会等





# 三、Java对象内存布局和对象头



在 JVM 中，**Java对象保存在堆中时，由以下三部分组成**：

- **对象头（object header）**：包括了关于堆对象的布局、类型、GC状态、同步状态和标识哈希码的基本信息。Java对象和JVM内部对象都有一个共同的对象头格式。
- **实例数据（Instance Data）**：主要是存放类的数据信息，父类的信息，对象字段属性信息。
- **对齐填充（Padding）**：为了字节对齐，填充的数据，不是必须的。默认情况下，由于HotSpot VM的自动内存管理系统要求对象起始地址必须是8字节的整数倍。如果一个对象用不到8N个字节则需要对其填充。

即：对象示例 = 对象头 + 实例数据 + 对齐填充



<hr/>



**对象头分为两类信息：一类是Mark Word用（于存储对象自身的运行时数据），一类是Klass Point（类型指针）。**

* 第一部分是**Mark Word用于存储对象自身的运行时数据**，如哈希码(HashCode)、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等。 这部分数据的长度在32位和64位的虚拟机(未开启压缩指针)中分别为32个比特和64个比特。

* 第二部分是**Klass Point（类型指针）**，即**对象指向它的类型元数据的指针，Java虚拟机通过这个指针来确定该对象是哪个类的实例**。
* 此外，**如果对象是一个Java数组，那在对象头中还必须有一块用于记录数组长度的数据。**

即：对象头 = 对象标记markword + 类型指针



> 即：对象在堆内存中的整体结构布局
>
> ![image-20210820094733244](https://gitee.com/jobim/blogimage/raw/master/img/20210820094733.png)





**Mark Word：**

Mark Word在不同的锁状态下存储的内容不同。

> 在32位JVM中是这么存的（了解）
>
> ![image-20210820094527861](https://gitee.com/jobim/blogimage/raw/master/img/20210820094528.png)

在64位JVM中的存储结构：

![image-20210820094539021](https://gitee.com/jobim/blogimage/raw/master/img/20210820094539.png)

虽然它们在不同位数的JVM中长度不一样，但是基本组成内容是一致的。

- **锁标志位（lock）**：区分锁状态，11时表示对象待GC回收状态, 只有最后2位锁标识(11)有效。
- **biased_lock**：是否偏向锁，由于无锁和偏向锁的锁标识都是 01，没办法区分，这里引入一位的偏向锁标识位。
- **分代年龄（age）**：表示对象被GC的次数，当该次数到达阈值的时候，对象就会转移到老年代。
- **对象的hashcode（hash）**：运行期间调用System.identityHashCode()来计算，延迟计算，并把结果赋值到这里。当对象加锁后，计算的结果31位不够表示，在偏向锁，轻量锁，重量锁，hashcode会被转移到Monitor中。
- **偏向锁的线程ID（JavaThread）**：偏向模式的时候，当某个线程持有对象的时候，对象这里就会被置为该线程的ID。 在后面的操作中，就无需再进行尝试获取锁的动作。
- **epoch**：偏向锁在CAS锁操作过程中，偏向性标识，表示对象更偏向哪个锁。
- **ptr_to_lock_record**：轻量级锁状态下，指向栈中锁记录的指针。当锁获取是无竞争的时，JVM使用原子操作而不是OS互斥。这种技术称为轻量级锁定。在轻量级锁定的情况下，JVM通过CAS操作在对象的标题字中设置指向锁记录的指针。
- **ptr_to_heavyweight_monitor**：重量级锁状态下，指向对象监视器Monitor的指针。如果两个不同的线程同时在同一个对象上竞争，则必须将轻量级锁定升级到Monitor以管理等待的线程。在重量级锁定的情况下，JVM在对象的ptr_to_heavyweight_monitor设置指向Monitor的指针。





**代码示例证明对象头：（借助JOL工具）**



1. 查看new一个Object对象的对象头

   ![image-20210820095159299](https://gitee.com/jobim/blogimage/raw/master/img/20210820095159.png)
   
   
   
   | 字段        | 说明                                       |
   | ----------- | ------------------------------------------ |
   | OFFSET      | 偏移量，也就是到这个字段位置所占用的byte数 |
   | SIZE        | 后面类型的字节大小                         |
   | TYPE        | 是Class中定义的类型                        |
   | DESCRIPTION | DESCRIPTION是类型的描述                    |
   | VALUE       | VALUE是TYPE在内存中的值                    |
   
   可以看到这里**mark word占8byte（64bit），klass pointe 占4byte，另外剩余4byte是填充对齐的**，
   
   这是由于默认开启了**指针压缩** ，klass pointe 占4byte（默认其实是占用8byte）
   
   
   
2. 关闭指针压缩后，查看new一个Object对象的对象头。

   jdk8版本是默认开启指针压缩的，可以通过配置jvm参数开启关闭指针压缩，`-XX:-UseCompressedOops`。

   ![image-20210820095703415](https://gitee.com/jobim/blogimage/raw/master/img/20210820095703.png)

   如果关闭指针压缩重新打印对象的内存布局，可以发现总SIZE变大了，从下图中可以看到，对象头所占用的内存大小变为16byte（128bit），其中 mark word占8byte，klass pointe 占8byte，无对齐填充。





**一般而言64位JDK8按照默认情况下，new一个对象占多少内存空间？**

以下面的对象为例：其中int占4个字节，char占1个字节。

```java
class MyObject{
    int i = 5;
    char a = 'a';
}
```

所以是 **8**（对象头）+ **8**（类型指针，关闭指针压缩的情况） + **5** + **3**（类型填充） = **24字节**（虚拟机要求对象起始地址必须是8字节的整数倍。）

> **注意：引用本身的大小和操作系统的位数有关，在64位平台上，占8个字节，在32位平台上占4个字节**

好的博客：[Java对象的内存布局](https://www.cnblogs.com/jajian/p/13681781.html)



# 四、synchronized与锁升级



## 4.1 synchronized关键字

**方法上的 synchronized**

* **普通synchronized方法相当于给当前对象（this）加锁**

  ```java
  class Test{
      public synchronized void test() {
  
      }
  }
  等价于
  class Test{
      public void test() {
          synchronized(this) { // 普通synchronized方法相当于给当前类对象加锁
  
          }
      }
  }
  ```

* **静态synchronized方法，相当于给当前类的class对象加锁**

  ```java
  class Test{
      public synchronized static void test() {
      }
  }
  等价于
  class Test{
      public static void test() {
          synchronized(Test.class) { // 静态synchronized方法，相当于给当前类的class对象加锁
  
          }
      }
  }
  ```

  



**private 或 final的重要性： **提高线程的安全性

* 分析下面的程序：

  ```java
  class ThreadSafe {
      public final void method1(int loopNumber) {
          ArrayList<String> list = new ArrayList<>();
          for (int i = 0; i < loopNumber; i++) {
              method2(list);
              method3(list);
          }
      }
      private void method2(ArrayList<String> list) {
          list.add("1");
      }
      public void method3(ArrayList<String> list) {
          list.remove(0);
      }
  }
  class ThreadSafeSubClass extends ThreadSafe{
      @Override
      public void method3(ArrayList<String> list) {
          new Thread(() -> {
              list.remove(0);
          }).start();
      }
  }
  ```

  本来`ThreadSafe`类为线程安全类，但由于子类ThreadSafeSubClass重写了method3()方法，导致`ThreadSafe`类不在线程安全。

  由于method3()方法为public, 此时子类可以重写父类的方法, 在子类中开线程来操作list对象, 此时就会出现线程安全问题: 子类和父类共享了list对象

**总结：**

* 如果改为private, 子类就不能重写父类的私有方法, 也就不会出现线程安全问题; 所以所private修饰符是可以避免线程安全问题.
* 所以如果不想子类, 重写父类的方法的时候, 我们可以将父类中的方法设置为private, final修饰的方法, 此时子类就无法影响父类中的方法了



## 4.2 synchronized的锁升级

JDK1.6 对锁的实现引入了大量的优化，如偏向锁、轻量级锁、自旋锁、适应性自旋锁、锁消除、锁粗化等技术来减少锁操作的开销。

### 4.2.1 偏向锁

<font color='orange'>为什么要引入偏向锁？</font>

* 因为经过HotSpot的作者大量的研究发现，大多数时候是不存在锁竞争的，常常是一个线程多次获得同一个锁，因此如果每次都要竞争锁会增大很多没有必要付出的代价，为了降低获取锁的代价，才引入的偏向锁。



**偏向锁的升级和撤销：**

* 当线程1访问代码块并获取锁对象时，会在**java对象头和栈帧中记录偏向的锁的threadID**，由无锁升级为偏向锁，因为偏向锁不会主动释放锁，因此以后线程1再次获取锁的时候，需要**比较当前线程的threadID和Java对象头中的threadID**是否一致。

* 如果一致（还是线程1获取锁对象），表示偏向锁是偏向于当前线程的，则无需使用CAS来加锁、解锁了，直接进入同步；

* 如果不一致，则进行CAS操作，企图将当前线程ID替换进Mark Word。**如果当前对象锁状态处于匿名偏向锁状态（可偏向未锁定），则会替换成功**（将Mark Word中的Thread id由匿名0改成当前线程ID。

* 如果对象锁已经被其他线程占用，则会替换失败，开始进行偏向锁撤销。

  * 偏向锁的撤销**需要等待全局安全点**（safe point，代表了一个状态，**在该状态下所有线程都是暂停的**），暂停持有偏向锁的线程，检查持有偏向锁的线程状态（遍历当前JVM的所有线程，如果能找到，则说明偏向的线程还存活），如果线程还存活，则检查线程是否在执行同步代码块中的代码，如果是，则升级为轻量级锁，进行CAS竞争锁。
  * 如果持有偏向锁的线程未存活，或者持有偏向锁的线程未在执行同步代码块中的代码，则进行校验是否允许重偏向。
    * 如果不允许重偏向，则撤销偏向锁，将Mark Word设置为无锁状态（未锁定不可偏向状态），然后升级为轻量级锁，进行CAS竞争锁
    * 如果允许重偏向，设置为匿名偏向锁状态,CAS将偏向锁重新指向线程A（在对象头和线程栈帧的锁记录中存储当前线程ID）；唤醒暂停的线程，从安全点继续执行代码

  
  
* 锁升级过程中Mark Word的改变。从无锁升级到偏向锁，Mark Word的后三位会**从001变为101**。并且Mark Word将执行持有偏向锁的线程id。

  ![image-20210820111238627](https://gitee.com/jobim/blogimage/raw/master/img/20210820111238.png)



**偏向锁的撤销：**

偏向锁的撤销需要等待全局安全点（即没有字节码正在执行），它会暂停拥有偏向锁的线程，撤销后偏向锁恢复到未锁定状态或轻量级锁状态。



**偏向锁相关jvm参数：**

* 偏向锁是默认开启的，但是默认偏向锁开始时间比**应用程序启动有四秒的延迟**
  * 可以使用**`XX:BiasedLockingStartupDelay=0`来禁用延迟**
  * 可以使用**`-XX:-UseBiasedLocking`来禁止偏向锁**

  >jvm默认和偏向锁有关的参数
  >
  >![image-20210819222155513](https://gitee.com/jobim/blogimage/raw/master/img/20210819222155.png)



**代码测试：**

* 禁用延迟之后可以看到，线程获取锁对象时，Mark Word的标志位变成了101

  ![image-20210820112530916](https://gitee.com/jobim/blogimage/raw/master/img/20210820112531.png)





**相关了解**

* **偏向锁的撤销情况**。当调用对象的`hashcode方法`的时候就会`撤销这个对象的偏向锁`，**因为使用偏向锁时没有位置存`hashcode`的值了**

  ![image-20210820111845723](https://gitee.com/jobim/blogimage/raw/master/img/20210820111845.png)



* **批量重偏向**
  * 如果对象被多个线程访问，但是没有竞争 , 这时偏向T1的对象仍有机会重新偏向T2。重偏向会重置Thread ID
  * 当撤销偏向锁阈值超过 20 次后，jvm 会这样觉得，我是不是偏向错了呢，于是会在给这些对象加锁时重新偏向至加锁线程。因为前19次是轻量，释放之后为无锁不可偏向，但是20次后面的是偏向t2，释放之后依然是偏向t2。

* **批量撤销偏向锁**。当撤销偏向锁阈值超过 40 次后，jvm 会这样觉得，自己确实偏向错了，根本就不该偏向。于是整个类的所有对象都会变为不可偏向的，新建的对象也是不可偏向的



### 4.2.2 轻量级锁

**轻量级锁的本质就是自旋锁**



<font color='orange'>为什么要引入轻量级锁？</font>

* 轻量级锁考虑的是**竞争锁对象的线程不多**，而且**线程持有锁的时间也不长**的情景。因为阻塞线程需要CPU从用户态转到内核态，代价较大，如果刚刚阻塞不久这个锁就被释放了，那这个代价就有点得不偿失了，因此这个时候就干脆不阻塞这个线程，让它自旋这等待锁释放。



<hr/>



**轻量锁的升级时机** ： 当**关闭偏向锁功能**或**多线程竞争偏向锁会导致偏向锁升级为轻量级锁**



<hr/>



<font color='orange'>轻量级锁什么时候升级为重量级锁？</font>

* 线程1获取轻量级锁时会先把锁对象的**对象头MarkWord复制一份到线程1的栈帧中创建的用于存储锁记录的空间（Lock Record）**，然后使用**CAS把对象头中的内容替换为线程1存储的锁记录的地址**；

  * 如果在线程1复制对象头的同时（在线程1CAS之前），线程2也准备获取锁，复制了对象头到线程2的锁记录空间（Lock Record）中，但是在线程2CAS的时候，发现线程1已经把对象头换了，线程2的CAS失败，那么**线程2就尝试使用自旋锁来等待线程1释放锁**。

  * 但是如果自旋的时间太长也不行，因为自旋是要消耗CPU的，因此自旋的次数是有限制的，如果自旋次数到了线程1还没有释放锁，或者线程1还在执行，线程2还在自旋等待，这时又有一个线程3过来竞争这个锁对象，那么这个时候**轻量级锁就会膨胀为重量级锁**。**重量级锁把除了拥有锁的线程都阻塞，防止CPU空转。**



<hr/>



**自适应自旋锁**

* JDK 1.6引入了更加聪明的自旋锁，即自适应自旋锁。所谓自适应就意味着自旋的次数不再是固定的，它是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。

* **线程如果自旋成功了，那么下次自旋的次数会更加多，因为虚拟机认为既然上次成功了，那么此次自旋也很有可能会再次成功，那么它就会允许自旋等待持续的次数更多。反之，如果对于某个锁，很少有自旋能够成功，那么在以后要或者这个锁的时候自旋的次数会减少甚至省略掉自旋过程，以免浪费处理器资源。**



#### 4.2.2.1 轻量级锁流程解释

**轻量级锁加锁流程：**

1. 在获取轻量锁时会**创建锁记录（Lock Record）对象**，每个线程的栈帧都会包含一个锁记录的结构，内部可以存储锁定对象的Mark Word。

   ![image-20210820120321710](https://gitee.com/jobim/blogimage/raw/master/img/20210820120321.png)
   
2. 让锁记录中的**Object reference指向锁对象地址**，并且尝试**用CAS将栈帧中的锁记录的(lock record 地址 00)替换Object对象的Mark Word，将Mark Word 的值(01)存入锁记录(lock record地址)**------相互替换

   * 01 表示 无锁 (看Mark Word结构，数字的含义)
   * 00 表示 轻量级锁

   ![image-20210820120610936](https://gitee.com/jobim/blogimage/raw/master/img/20210820120611.png)



3. 如果**cas替换成功, 获得了轻量级锁**，那么对象的**对象头储存的就是锁记录的地址和状态00**。**线程中锁记录, 记录了锁对象的锁状态标志; 锁对象的对象头中存储了锁记录的地址和状态, 标志哪个线程获得了锁**
   ![image-20210820120834567](https://gitee.com/jobim/blogimage/raw/master/img/20210820120834.png)



4. 如果**cas替换失败**，有两种情况 : **① 锁膨胀 ② 执行了锁重入**

   * 如果是其它线程已经持有了该Object的轻量级锁，那么表示有竞争，自旋一定的时间后，将进入**锁膨胀阶段（膨胀我重量级锁）**。

   * 如果是**自己的线程已经执行了synchronized进行加锁**，那么**再添加一条 Lock Record 作为重入锁的计数** – 线程多次加锁, 锁重入。
     ![image-20210820121349323](https://gitee.com/jobim/blogimage/raw/master/img/20210820121349.png)

     ![image-20210820121420448](https://gitee.com/jobim/blogimage/raw/master/img/20210820121420.png)



<hr/>



**轻量级锁解锁流程：**

* 当`线程退出synchronized代码块`的时候，如果获取的是`取值为 null 的锁记录` ，表示有`锁重入`，这时重置锁记录，`表示重入计数减一`。
* 当线程退出synchronized代码块的时候，如果获取的锁记录取值不为 null，那么**使用cas将Mark Word的值恢复给对象, 将直接替换的内容还原。**
  * 成功则解锁成功 (轻量级锁解锁成功)
  * 失败，表示有竞争, 则**说明轻量级锁进行了锁膨胀或已经升级为重量级锁**，**进入重量级锁解锁流程** (Monitor流程)



<hr/>



**轻量级锁膨胀流程：**



- 如果在尝试加轻量级锁的过程中，CAS 操作无法成功，这时一种情况就是有其它线程为此对象加上了轻量级锁（有竞争），这时**需要进行锁膨胀，将轻量级锁变为重量级锁**。

- 当 Thread-1 进行轻量级加锁失败时，Thread-0 已经对该对象加了轻量级锁, 此时发生`锁膨胀`

  ![image-20210820122205892](https://gitee.com/jobim/blogimage/raw/master/img/20210820122205.png)

* 这时Thread-1加轻量级锁失败，进入锁膨胀流程

  * 因为**Thread-1线程加轻量级锁失败, 轻量级锁没有阻塞队列的概念, 所以此时就要为对象申请Monitor锁(重量级锁)**，让Object指向重量级锁地址 。
  * 然后自己进入Monitor的EntryList变成BLOCKED状态

  ![image-20210820122347534](https://gitee.com/jobim/blogimage/raw/master/img/20210820122347.png)

* 当 Thread-0 退出同步块解锁时，使用 cas 将 Mark Word 的值恢复给对象头，失败。这时会进入重量级解锁流程，即**按照 Monitor 地址找到 Monitor 对象，设置 Owner 为 null，唤醒 EntryList 中 BLOCKED 线程**





### 4.2.3 Monitor 原理 (重量级锁原理)

**Monitor也称为监视器或者管程**

**每个Java对象都可以关联一个(操作系统的)Monitor，如果使用synchronized给对象上锁（重量级），该对象头的MarkWord中就被设置为指向Monitor对象的指针** 

> 下图原理解释:
>
> - `当Thread2`访问到`synchronized(obj)`中的共享资源的时候
>
>   - 首先会将synchronized中的`锁对象`中`对象头`的`MarkWord`去尝试指向`操作系统`的`Monitor`对象. 让锁对象中的MarkWord和Monitor对象相关联. 如果关联成功, 将obj对象头中的`MarkWord`为指向`重量级锁的指针`，并且标志位变为10。
>
>     ![image-20210820101816924](https://gitee.com/jobim/blogimage/raw/master/img/20210820101817.png)
>
>   - 因为Monitor没有和其他的obj的MarkWord相关联, 所以`Thread2`就成为了该`Monitor`的Owner(所有者)。
>
>   - 又来了个`Thread1`执行synchronized(obj)代码, 它首先会看看能不能执行该`临界区`的代码; 它会检查obj是否关联了Montior, 此时已经有关联了, 它就会去看看该Montior有没有所有者(Owner), 发现有所有者了(Thread2); `Thread1`也会和该Monitor关联, 该线程就会进入到它的`EntryList(阻塞队列)`;
>
>   - 当`Thread2`执行完`临界区`代码后, Monitor的`Owner(所有者)`就空出来了. 此时就会`通知`Monitor中的EntryList阻塞队列中的线程, 这些线程通过`竞争`, 成为新的`所有者`
>     ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201219192811839.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM3OTg5OTgw,size_16,color_FFFFFF,t_70)



**总结：**

![1583652360228](https://img-blog.csdnimg.cn/img_convert/98c3189e41fd654fe34ead273ec76eba.png)

- 刚开始时`Monitor`中的`Owner为null`
- **当Thread-2 执行synchronized(obj){}代码时，首先会关联obj对象的Monitor，然后会将Monitor的所有者Owner 设置为 Thread-2，上锁成功，Monitor中同一时刻只能有一个Owner**
- 当Thread-2 占据锁时，如果线程Thread-3，Thread-4也来执行synchronized(obj){}代码，就会进入`EntryList`中变成`BLOCKED状态`
- **Thread-2 执行完同步代码块的内容，然后唤醒 EntryList 中等待的线程来竞争锁，`竞争时是非公平的 (仍然是抢占式)`**
- `图中 WaitSet 中的Thread-0，Thread-1是之前获得过锁，但条件不满足进入 WAITING 状态的线程，后面讲wait-notify 时会分析`



> 注意：它加锁就是依赖底层操作系统的 `mutex`相关指令实现, 所以会造成`用户态和内核态之间的切换`, 非常耗性能 !
>
> - 在JDK6的时候, 对synchronized进行了优化, 引入了`轻量级锁, 偏向锁`, 它们是在JVM的层面上进行加锁逻辑, 就没有了切换的消耗





**分析synchronized的字节码**

Synchronized代码块同步在需要同步的代码块开始的位置插入monitorenter指令，在同步结束的位置或者异常出现的位置插入monitorexit指令；JVM要保证`monitorenter`和`monitorexit`都是成对出现的，任何对象都有一个monitor与之对应，当这个对象的monitor被持有以后，它将处于锁定状态。

```java
static final Object lock = new Object();
static int counter = 0;
public static void main(String[] args) {
    synchronized (lock) {
        counter++;
    }
}
```

对应的字节码为：

```java
public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC 
    Code:
        stack=2, locals=3, args_size=1
            0: getstatic #2 	// <- lock引用 （synchronized开始）
            3: dup
            4: astore_1 		// lock引用 -> slot 1
            5: monitorenter 	// 将 lock对象 MarkWord 置为 Monitor 指针
            6: getstatic #3 	// <- i ，6-14行即为i++操作
            9: iconst_1 		// 准备常数 1
            10: iadd 			// +1
            11: putstatic #3 	// -> i
            14: aload_1 		// <- lock引用
            15: monitorexit 	// 将 lock对象 MarkWord 重置, 唤醒 EntryList
            16: goto 24
            19: astore_2 		// e -> slot 2
            20: aload_1 		// <- lock引用
            21: monitorexit 	// 将 lock对象 MarkWord 重置, 唤醒 EntryList
            22: aload_2 		// <- slot 2 (e)
            23: athrow 			// throw e
            24: return
        Exception table:
        from to target type
        6 16 19 any     //如果6-16行出现异常则转到19行，如果出现异常也可以释放锁
        19 22 19 any

```

当执行 `monitorenter` 指令时，线程试图获取锁也就是获取 **对象监视器 `monitor`** 的持有权。

在执行`monitorenter`时，会尝试获取对象的锁，如果锁的计数器为 0 则表示锁可以被获取，获取后将锁计数器设为 1 也就是加 1。

在执行 `monitorexit` 指令后，将锁计数器设为 0，表明锁被释放。如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止。





### 4.2.4 锁消除和锁粗化



**锁消除**

* Java虚拟机在JIT（Just In Time Compiler，一般翻译为即时编译器）编译时(可以简单理解为当某段代码即将第一次被执行时进行编译)，通过对运行上下文的扫描，经过逃逸分析，**去除不可能存在共享资源竞争的锁，通过这种方式消除没有必要的锁，可以节省毫无意义的请求锁时间**

* 关闭锁消除的开关：`java -XX:-EliminateLocks -jar benchmarks.jar`



**锁粗化**

* 按理来说，同步块的作用范围应该尽可能小，仅在共享数据的实际作用域中才进行同步，这样做的目的是为了使需要同步的操作数量尽可能缩小，缩短阻塞时间，如果存在锁竞争，那么等待锁的线程也能尽快拿到锁。 但是加锁解锁也需要消耗资源，如果存在一系列的连续加锁解锁操作，可能会导致不必要的性能损耗。
* **锁粗化就是将多个连续的加锁、解锁操作连接在一起，扩展成一个范围更大的锁，避免频繁的加锁解锁操作。**



### 4.2.5 总结

**这几种锁的优缺点：**

| 锁       | 优点                                                         | 缺点                                             | 适用场景                                                     |
| -------- | ------------------------------------------------------------ | ------------------------------------------------ | ------------------------------------------------------------ |
| 偏向锁   | 加锁和解锁不需要额外的消耗，和执行非同步方法比仅存在纳秒级的差距。 | 如果线程间存在锁竞争，会带来额外的锁撤销的消耗。 | 适用于基本没有线程竞争锁的同步场景。                         |
| 轻量级锁 | 竞争的线程不会阻塞，提高了程序的响应速度。                   | 如果始终得不到锁竞争的线程使用自旋会消耗CPU。    | 适用于少量线程竞争锁对象，且线程持有锁的时间不长，追求响应速度的场景。 |
| 重量级锁 | 线程竞争不使用自旋，不会消耗CPU。                            | 线程阻塞，响应时间缓慢。                         | 很多线程竞争锁，同步块执行时间较长。追求吞吐量的场景。       |

**锁升级的流程图：**

![image-20210820145139773](https://gitee.com/jobim/blogimage/raw/master/img/20210820145140.png)



synchronized 锁升级原理：

* 在锁对象的对象头里面有一个 threadid 字段，在**第一次访问的时候 threadid 为空，jvm 让其持有偏向锁，并将 threadid 设置为其线程 id**。
* 再次进入的时候会先判断 threadid 是否与其线程 id 一致，如果一致则可以直接使用此对象，如果不一致，则判断上一个线程是否退出同步代码块。
  * 如果**已经退出同步块**，则将对象头设置成无锁状态并撤销偏向锁，重新偏向。
  * 如果**处于同步块中**，它还没有执行完，其它线程来抢夺，该偏向锁会被取消掉并出现锁升级。此时轻量级锁由原持有偏向锁的线程持有，继续执行其同步代码，而正在竞争的线程会进入自旋等待获得该轻量级锁。
  * 执行一定次数之后，如果还没有正常获取到要使用的对象，此时就会把**锁从轻量级升级为重量级锁**。



好的博客：

[深入分析Synchronized原理(阿里面试题)](https://www.cnblogs.com/aspirant/p/11470858.html)

[Java并发——Synchronized关键字和锁升级，详细分析偏向锁和轻量级锁的升级](https://blog.csdn.net/tongdanping/article/details/79647337?ops_request_misc=%7B%22request%5Fid%22%3A%22162942175516780261948164%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=162942175516780261948164&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~blog~sobaiduend~default-1-79647337.pc_v2_rank_blog_default&utm_term=synchronized的锁升级&spm=1018.2226.3001.4450)

[Java并发编程(三) : synchronized底层原理、优化Monitor重量级锁、轻量级锁、自旋锁(优化重量级锁竞争)、偏向锁](https://blog.csdn.net/m0_37989980/article/details/111408759)





# 五、wait和notify 

**wait、notify原理**

* 当**线程0获得到了锁, 成为Monitor的Owner**, 但是此时它发现自己想要执行synchroized代码块的条件不满足；此时它就**调用obj.wait方法, 进入到Monitor中的WaitSet集合, 此时线程0的状态就变为WAITING**

* 处于BLOCKED和WAITING状态的线程都为阻塞状态，CPU都不会分给他们时间片。但是有所区别：

  * **BLOCKED状态的线程是在竞争锁对象时，发现Monitor的Owner已经是别的线程了，此时就会进入EntryList中，并处于BLOCKED状态**
  * **WAITING状态的线程是获得了对象的锁，但是自身的原因无法执行synchroized的临界区资源需要进入阻塞状态时，锁对象调用了wait方法而进入了WaitSet中，处于WAITING状态**

* **处于BLOCKED状态的线程会在锁被释放的时候被唤醒**

* **处于WAITING状态的线程只有被锁对象调用了notify方法(obj.notify/obj.notifyAll)，才会被唤醒。然后它会进入到EntryList, 重新竞争锁** 

  ![image-20210821002936836](https://gitee.com/jobim/blogimage/raw/master/img/20210821002936.png)





<hr/>



**API介绍：**

下面的四个方法都是`Object`中的方法; 通过`锁对象`来调用

- `wait()`: 方法会释放对象的锁，进入 `WaitSet `等待区，从而让其他线程就机会获取对象的锁。无限制等待，直到notify 为止
- `wait(long n)` : 当该等待线程没有被notify, 等待时间到了之后, 也会自动唤醒

- `notify()`: 让获得对象锁的线程, 使用锁对象调用`notify`去**waitSet的等待线程中挑一个唤醒**
- `notifyAll()` : 让获得对象锁的线程, 使用锁对象调用`notifyAll`去**唤醒waitSet中所有的等待线程**

注意：它们都是`线程之间进行协作的手段`, 都属于`Object对象的方法`, **必须获得此对象的锁, 才能调用这些方法**

```java
public class Test1 {
	final static Object LOCK = new Object();
	public static void main(String[] args) throws InterruptedException {
        //只有在对象被锁住后才能调用wait方法
		synchronized (LOCK) {
			LOCK.wait();
		}
	}
}
```



**wait()使用注意：防止出现虚假唤醒机制**

* 当对共享变量进行判断的时候，为了防止出现虚假唤醒机制，不能使用if来进行判断，而应该使用**while**。因为当线程被唤醒时候必须再进行一次判断。

  ```java
  synchronized(lock) {
      while(条件不成立) {
          lock.wait();
      }
      // 干活
  }
  //另一个线程
  synchronized(lock) {
      lock.notifyAll();
  }
  ```



<hr/>





<font color='red' size='5'>**sleep() 和 wait() 有什么区别？**</font>

相同点：

* 一旦执行方法，都可以使得当前的线程进入阻塞状态。

不同点：

* 两个方法声明的位置不同：**sleep() 是 Thread线程类的静态方法**，**wait() 是 Object类的方法**。
* 是否释放锁：如果两个方法都使用在同步代码块或同步方法中，**sleep() 不释放锁；wait() 释放锁**。
* 用途不同：wait 通常被用于线程间交互/通信，sleep 主要是为了暂停当前线程，把cpu片段让出给其他线程，减缓当前线程的执行。但是如果当前线程获取到的有锁，sleep不会让出锁
* 调用的要求不同：**sleep()可以在任何需要的场景下调用。 wait()必须使用在同步代码块或同步方法中**





## 5.1 消费者、生产者模式

- 我们下面写的例子是`线程间通信`的`消息队列`，要注意区别,像`RabbitMQ`等消息框架是`进程间通信`的。

```java
@Slf4j(topic = "c.Test21")
public class TestConsume {

    public static void main(String[] args) {
        MessageQueue queue = new MessageQueue(2);

        for (int i = 0; i < 3; i++) {
            int id = i;
            new Thread(() -> {
                queue.put(new Message(id , "值"+id));
            }, "生产者" + i).start();
        }

        new Thread(() -> {
            while(true) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                Message message = queue.take();
            }
        }, "消费者").start();
    }

}

// 消息队列类 ， java 线程之间通信
@Slf4j(topic = "c.MessageQueue")
class MessageQueue {
    // 消息的队列集合
    private LinkedList<Message> list = new LinkedList<>();
    // 队列容量
    private int capcity;

    public MessageQueue(int capcity) {
        this.capcity = capcity;
    }

    // 获取消息
    public Message take() {
        // 检查队列是否为空
        synchronized (list) {
            while(list.isEmpty()) {
                try {
                    log.debug("队列为空, 消费者线程等待");
                    list.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            // 从队列头部获取消息并返回
            Message message = list.removeFirst();
            log.debug("已消费消息 {}", message);
            list.notifyAll();
            return message;
        }
    }

    // 存入消息
    public void put(Message message) {
        synchronized (list) {
            // 检查对象是否已满
            while(list.size() == capcity) {
                try {
                    log.debug("队列已满, 生产者线程等待");
                    list.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            // 将消息加入队列尾部
            list.addLast(message);
            log.debug("已生产消息 {}", message);
            list.notifyAll();
        }
    }
}

final class Message {
    private int id;
    private Object value;

    public Message(int id, Object value) {
        this.id = id;
        this.value = value;
    }

    public int getId() {
        return id;
    }

    public Object getValue() {
        return value;
    }

    @Override
    public String toString() {
        return "Message{" +
                "id=" + id +
                ", value=" + value +
                '}';
    }
}
```



执行结果：

![image-20210821005250053](https://gitee.com/jobim/blogimage/raw/master/img/20210821005250.png)





# 六、LockSupport之park、unpack 



- `park/unpark`都是`LockSupport`类中的的方法

* `park`用于暂停某个线程，`unpark`用于恢复某个线程的运行。

注意：先调用`unpark`后,再调用`park`, 此时`park`不会暂停线程



```java
@slf4j
public class Test {
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            log.debug("start...");
            sleep(2);
            log.debug("park...");
            LockSupport.park();
            log.debug("resume...");
        }, "t1");
        t1.start();
        sleep(1);
        log.debug("unpark...");
        LockSupport.unpark(t1);
    }
}
```

输出：

```
18:43:50.765 c.TestParkUnpark [t1] - start... 
18:43:51.764 c.TestParkUnpark [main] - unpark... 
18:43:52.769 c.TestParkUnpark [t1] - park... 
18:43:52.769 c.TestParkUnpark [t1] - resume...
```



<hr/>



**与Object的wait&notify区别?**

* **wait，notify 和 notifyAll 必须配合 Object Monitor 一起使用**，而 park，unpark 不需要
* park & unpark 是**以线程为单位来【阻塞】和【唤醒】线程**，而 notify 只能随机唤醒一个等待线程，notifyAll是唤醒所有等待线程，就不那么【精确】
* park&unpark可以先unpark，而wait&notify不能先notify



<hr/>



**park、unpark 原理：**

* **先调用`park`再调用`upark`的过程**

  * 先调用park的情况

    * 当前线程调用 Unsafe.park() 方法
    * 检查 `_counter`, 本情况为0, 这时, 获得 `_mutex` 互斥锁_
    * 线程进入 `_cond` 条件变量**阻塞**
    * _设置_ **_counter = 0**

    ![image-20210821100225059](https://gitee.com/jobim/blogimage/raw/master/img/20210821100225.png)

   * 调用unpark

    * 调用Unsafe.unpark(Thread_0)方法，设置`_counter` 为 1
    * 唤醒 `_cond` 条件变量中的 Thread_0
    * Thread_0 恢复运行
    * 设置 `_counter` 为 0

    ![image-20210821100510477](https://gitee.com/jobim/blogimage/raw/master/img/20210821100510.png)

* **先调用upark再调用park的过程**

  * 调用 Unsafe.unpark(Thread_0)方法，设置 `_counter` 为 1
  * 当前线程调用 Unsafe.park() 方法
  * 检查 `_counter`，本情况为 1，**这时线程 无需阻塞，继续运行**
  * 设置 _counter 为 0
  * 注意： **_counter的值最大为1，所以unpark给线程最多1个"许可"**

  ![image-20210821100459825](https://gitee.com/jobim/blogimage/raw/master/img/20210821100500.png)



* **总结：**
  * park和unpark会调用Unsafe类中的native方法
  * 每个线程都会和一个park对象关联起来，由三部分组成 _counter ， _cond 和 _mutex。核心部分是counter，我们可以理解为一个标记位。
  * 当调用park时会**看counter是否为0，为0则进入阻塞队列。为1则继续运行并将counter置为0。**
  * 当调用**unpark时，会将counter置为1，若之前的counter值为0，还唤醒阻塞的线程。**







