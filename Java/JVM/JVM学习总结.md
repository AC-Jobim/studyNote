

好的博客：[JVM ](https://nyimac.gitee.io/2020/07/03/JVM学习/)

# 一、JVM介绍

1. `Java虚拟机`是一台`执行Java字节码`的`虚拟计算机`，它拥有独立的运行机制，其运行的Java字节码也未必由Java语言编译而成。
2. JVM平台的各种语言可以共享**Java虚拟机**带来的`跨平台性、优秀的垃圾回收器，以及可靠的即时编译器。`
3. Java技术的核心就是Java虚拟机（JVM，Java Virtual Machine），因为`所有的Java程序都运行在Java虚拟机内部。`
4. Java虚 拟机就是**二进制字节码**的运行环境，负责**装载字节码**到其内部，**解释/编译为对应平台上的机器指令执行**。每一条Java指令，Java虚拟机规范中都有详细定义，如怎么取操作数，怎么处理操作数，处理结果放在哪里。



**特点：**

- `一次编译，到处运行`
- `自动内存管理`
- `自动垃圾回收功能`





**JVM是运行在操作系统之上的，它与硬件没有直接的交互**

![image-20210829104842042](https://gitee.com/jobim/blogimage/raw/master/img/20210829104842.png)



**JVM、JRE 、JDK的区别**

![image-20210829105606361](https://gitee.com/jobim/blogimage/raw/master/img/20210829105606.png)





**如何反编译字节码文件**

- 在 .class 文件的同级目录下，执行反编译`javap -v StackStruTest.class`

  ```java
  javap -v HelloWorld.class
  ```

  



# 二、内存结构

JVM包含两个子系统和两个组件，两个子系统为Class loader(类装载)、Execution engine(执行引擎)；两个组件为Runtime data area(运行时数据区)、Native Interface(本地方法接口)。

* Class loader(类装载)：根据给定的全限定名类名(如：java.lang.Object)来装载class文件到Runtime data area中的method area。

* Execution engine（执行引擎）：执行classes中的指令。

* Native Interface(本地接口)：与native libraries交互，是其它编程语言交互的接口。

* Runtime data area(运行时数据区域)：这就是我们常说的JVM的内存。

**作用** ：**首先通过编译器把 Java 代码转换成字节码，类加载器（ClassLoader）再把字节码加载到内存中，将其放在运行时数据区（Runtime data area）的方法区内，而字节码文件只是 JVM 的一套指令集规范，并不能直接交给底层操作系统去执行，因此需要特定的命令解析器执行引擎（Execution Engine），将字节码翻译成底层系统指令，再交由 CPU 去执行，而这个过程中需要调用其他语言的本地库接口（Native Interface）来实现整个程序的功能。**



**JVM的内存模型：**

> ![image-20210829110646063](https://gitee.com/jobim/blogimage/raw/master/img/20210829110646.png)
>
> ![image-20210911095646649](https://gitee.com/jobim/blogimage/raw/master/img/20210911095646.png)





**线程私有的：**

- 程序计数器
- 虚拟机栈
- 本地方法栈

**线程共享的：**

- 堆
- 方法区（1.8 转到直接内存的元空间）
- 直接内存 (非运行时数据区的一部分)



## 2.1 程序计数器

**作用**

* 用于保存JVM中下一条所要执行的指令的地址

**特点**

- 线程私有
  - CPU会为每个线程分配时间片，当当前线程的时间片使用完以后，CPU就会去执行另一个线程中的代码
  - 程序计数器是**每个线程**所**私有**的，当另一个线程的时间片用完，又返回来执行当前线程的代码时，通过**程序计数器可以知道应该执行哪一句指令**

* **程序计数器是唯一一个不会出现 OutOfMemoryError 的内存区域，它的生命周期随着线程的创建而创建，随着线程的结束而死亡。**





## 2.2 虚拟机栈

* 每个线程**在创建时都会创建一个虚拟机栈**，其**内部保存一个个的栈帧（Stack Frame）**，对应着一次次的Java方法调用，**栈是线程私有的**

* 栈中的数据都是以**栈帧**（Stack Frame）的格式存在。`栈帧是一个内存区块`，是一个`数据集`，维系着`方法执行过程中的各种数据信息`。（栈帧中拥有的数据：局部变量表、操作数栈、动态链接、方法出口信息）
* 每一次函数调用都会有一个对应的栈帧被压入 Java 栈，每一个函数调用结束后，都会有一个栈帧被弹出

* **局部变量表主要存放了编译器可知的各种数据类型**（boolean、byte、char、short、int、float、long、double）、**对象引用**（reference 类型，它不同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）。



**Java 虚拟机栈会出现两种异常：StackOverFlowError 和 OutOfMemoryError。**

- **StackOverFlowError：** 若 Java 虚拟机栈的内存大小不允许动态扩展，那么当线程请求栈的深度超过当前 Java 虚拟机栈的最大深度的时候，就抛出 StackOverFlowError 异常。
- **OutOfMemoryError：** 若 Java 虚拟机栈的内存大小允许动态扩展，且当线程请求栈时内存用完了，无法再动态扩展了，此时抛出 OutOfMemoryError 异常。



我们可以`使用参数` **`-Xss`** 选项来设置`线程的最大栈空间`，**栈的大小直接决定了函数调用的最大可达深度。**

```java
-Xss1024m		// 栈内存为 1024MBS
-Xss1024k		// 栈内存为 1024KB
```



## 2.3 本地方法栈

* 和虚拟机栈所发挥的作用非常相似，区别是： **虚拟机栈为虚拟机执行 Java 方法 （也就是字节码）服务，而本地方法栈则为虚拟机使用到的 Native 方法服务，用来管理`本地方法(Native Method)`的调用。** 在 HotSpot 虚拟机中和 Java 虚拟机栈合二为一。

* 本地方法被执行的时候，在本地方法栈也会创建一个栈帧，用于存放该本地方法的局部变量表、操作数栈、动态链接、出口信息。

* 方法执行完毕后相应的栈帧也会出栈并释放内存空间，也会出现 StackOverFlowError 和 OutOfMemoryError 两种异常。



## 2.4 堆

Java 虚拟机所管理的内存中最大的一块，Java 堆是所有线程共享的一块内存区域，在虚拟机启动时创建。**此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都在这里分配内存。**

Java 堆是垃圾收集器管理的主要区域，因此也被称作**GC 堆（Garbage Collected Heap）**.从垃圾回收的角度，由于现在收集器基本都采用分代垃圾收集算法，所以 Java 堆还可以细分为：新生代和老年代：再细致一点有：Eden 空间、From Survivor、To Survivor 空间等。**进一步划分的目的是更好地回收内存，或者更快地分配内存。**

[![img](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-3%E5%A0%86%E7%BB%93%E6%9E%84.png)](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-3堆结构.png)

上图所示的 eden 区、s0 区、s1 区都属于新生代，tentired 区属于老年代。大部分情况，对象都会首先在 Eden 区域分配，在一次新生代垃圾回收后，如果对象还存活，则会进入 s0 或者 s1，并且对象的年龄还会加 1(Eden 区->Survivor 区后对象的初始年龄变为 1)，当它的年龄增加到一定程度（默认为 15 岁），就会被晋升到老年代中。对象晋升到老年代的年龄阈值，可以通过参数 `-XX:MaxTenuringThreshold` 来设置。



![image-20210902162502975](https://gitee.com/jobim/blogimage/raw/master/img/20210902162503.png)









## 2.5 方法区



**方法区演进过程：**

* **在 JDK7 及以前，习惯上把方法区，称为永久代。JDK8开始，使用元空间取代了永久代，元空间使用的是本地内存（JVM 内存之外的部分叫作本地内存）**。JDK 1.8后，元空间存放在**堆外内存中**

  ![image-20210829120658242](https://gitee.com/jobim/blogimage/raw/master/img/20210829120658.png)







* 《深入理解Java虚拟机》书中对`方法区（Method Area）存储内容`描述如下：它用于存储已被虚拟机 **加载的类型信息、常量、静态变量、即时编译器编译后的代码缓存**等。

* 方法区与 Java 堆一样，是各个线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。





**常用参数：**

* JDK 1.8 之前永久代还没被彻底移除的时候通常通过下面这些参数来调节方法区大小

  ```
  -XX:PermSize=N //方法区 (永久代) 初始大小
  -XX:MaxPermSize=N //方法区 (永久代) 最大大小,超过这个值将会抛出 OutOfMemoryError 异常:java.lang.OutOfMemoryError: PermGen
  ```

  

* JDK 1.8 的时候，方法区（HotSpot 的永久代）被彻底移除了（JDK1.7 就已经开始了），取而代之是元空间，元空间使用的是本地内存。需要使用下面的参数：

  ```
  -XX:MetaspaceSize=N //设置 Metaspace 的初始（和最小大小）
  -XX:MaxMetaspaceSize=N //设置 Metaspace 的最大大小
  ```

  与永久代很大的不同就是，如果不指定大小的话，随着更多类的创建，虚拟机会耗尽所有可用的系统内存。





**常量池、运行时常量池、字符串常量池?**

* **运行时常量池在方法区中**；
* **字节码文件内部包含了常量池**（二进制字节码的组成：类的基本信息、常量池、类的方法定义、包含了虚拟机指令）；
* **字符串常量池在JDK7的时候放到了`堆空间`中**



### 2.5.1 常量池



* **通过反编译来查看类的信息**

  * 二进制字节码包含（类的基本信息，常量池，类方法定义，包含了虚拟机的指令）.首先看看常量池是什么，反编译如下代码：

    ```java
    public class HelloWorld {
        public static void main(String[] args) {
            System.out.println("Hello World!");
        }
    }
    ```

  * 使用 javap -v HelloWorld.class 命令反编译查看结果

    * **类的基本信息：**

      ![image-20210828190216013](https://gitee.com/jobim/blogimage/raw/master/img/20210828190216.png)

    * **常量池：**

      ![image-20210828190129214](https://gitee.com/jobim/blogimage/raw/master/img/20210828190129.png)

    * **类方法定义：**

      ![image-20210828190052962](https://gitee.com/jobim/blogimage/raw/master/img/20210828190053.png)

  



### 2.5.2 运行时常量池

- 当该**类被加载以后**，它的常量池信息就会**放入运行时常量池**，并把里面的**符号地址变为真实地址**





### 2.5.3 字符串常量池

* **字符串常量池在JDK7的时候放到了堆空间中**

- 常量池中的字符串仅是符号，只有**在被用到时才会转化为对象**
- 利用串池的机制，来避免重复创建字符串对象
- 字符串**变量**拼接的原理是**StringBuilder**
- 字符串**常量**拼接的原理是**编译器优化**
- 可以使用**intern方法**，将这个字符串对象尝试放入串池，如果有则并不会放入，如果没有则放入串池。最后会把串池中的对象返回





**字符串常量池 StringTable 为什么要调整位置？**

* **`JDK7`中将`StringTable`放到了`堆空间`中**。因为永久代的回收效率很低，在Full GC的时候才会执行永久代的垃圾回收，而Full GC是老年代的空间不足、永久代不足时才会触发。





* 使用拼接**字符串变量对象**创建字符串的过程（字节码分析）

  ```java
  public class StringTableStudy {
  	public static void main(String[] args) {
  		String a = "a";
  		String b = "b";
  		String ab = "ab";
  		//拼接字符串对象来创建新的字符串
  		String ab2 = a+b; 
  	}
  }
  ```

  反编译后的结果

  ```
  	 Code:
        stack=2, locals=5, args_size=1
           0: ldc           #2                  // String a
           2: astore_1
           3: ldc           #3                  // String b
           5: astore_2
           6: ldc           #4                  // String ab
           8: astore_3
           9: new           #5                  // class java/lang/StringBuilder
          12: dup
          13: invokespecial #6                  // Method java/lang/StringBuilder."<init>":()V
          16: aload_1
          17: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String
  ;)Ljava/lang/StringBuilder;
          20: aload_2
          21: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String
  ;)Ljava/lang/StringBuilder;
          24: invokevirtual #8                  // Method java/lang/StringBuilder.toString:()Ljava/lang/Str
  ing;
          27: astore        4
          29: returnCopy
  ```

  通过拼接的方式来创建字符串的**过程**是：StringBuilder().append(“a”).append(“b”).toString()，StringBuilder的toString方法底层时通过new String();

  最后的toString方法的返回值是一个**新的字符串**，但字符串的**值**和拼接的字符串一致，但是两个不同的字符串，**一个存在于串池之中，一个存在于堆内存之中**

  ```java
  String ab = "ab";
  String ab2 = a+b;
  //结果为false,因为ab是存在于串池之中，ab2是由StringBuffer的toString方法所返回的一个对象，存在于堆内存之中
  System.out.println(ab == ab2); //false
  ```

* 使用**拼接字符串常量对象**的方法创建字符串

  ```java
  public class StringTableStudy {
  	public static void main(String[] args) {
  		String a = "a";
  		String b = "b";
  		String ab = "ab";
  		String ab2 = a+b;
  		//使用拼接字符串的方法创建字符串
  		String ab3 = "a" + "b";
  	}
  }
  ```

  反编译后的结果：

  ```
   	  Code:
        stack=2, locals=6, args_size=1
           0: ldc           #2                  // String a
           2: astore_1
           3: ldc           #3                  // String b
           5: astore_2
           6: ldc           #4                  // String ab
           8: astore_3
           9: new           #5                  // class java/lang/StringBuilder
          12: dup
          13: invokespecial #6                  // Method java/lang/StringBuilder."<init>":()V
          16: aload_1
          17: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String
  ;)Ljava/lang/StringBuilder;
          20: aload_2
          21: invokevirtual #7                  // Method java/lang/StringBuilder.append:(Ljava/lang/String
  ;)Ljava/lang/StringBuilder;
          24: invokevirtual #8                  // Method java/lang/StringBuilder.toString:()Ljava/lang/Str
  ing;
          27: astore        4
          //ab3初始化时直接从串池中获取字符串
          29: ldc           #4                  // String ab
          31: astore        5
          33: returnCopy
  ```

  可以看到第29行，ab3直接从池中获取值，**因为内容常量，javac在编译期会进行优化，结果已在编译期确定为ab**。

  

  **总结：**

  * 使用**拼接字符串常量**的方法来创建新的字符串时，因为**内容是常量，javac在编译期会进行优化，结果已在编译期确定为ab**，而创建ab的时候已经在串池中放入了“ab”，所以ab3直接从串池中获取值，所以进行的操作和 ab = “ab” 一致。

  * 使用**拼接字符串变量**的方法来创建新的字符串时，因为内容是变量，只能**在运行期确定它的值，所以需要使用StringBuilder来创建**



* **StringTable调优**

  * 因为StringTable是由HashTable实现的，所以可以**适当增加HashTable桶的个数**，来减少字符串放入串池所需要的时间

    ```
    -XX:StringTableSize=xxxxCopy
    ```

  * 考虑是否需要将字符串对象入池，可以通过**intern方法减少重复入池**





## 2.6 直接内存



- 属于操作系统，不是虚拟机运行时数据区的一部分
- 常用于NIO操作，用于数据缓冲区
- 不受JVM内存回收管理，使用了Unsafe类来完成直接内存的分配回收



直接内存回收原理（待补）



# 二、垃圾回收

## 2.1 标记阶段

**垃圾标记阶段：`判断对象是否存活`**

- 已经死亡的对象, 就会被垃圾回收器进行回收
- 判断对象存活一般有两种方式：**`引用计数算法`和`可达性分析算法`。**

### 2.1.1 引用计数法

- 对于一个对象A，只要有任何一个对象引用了A，则A的引用计数器就加1；当引用失效时，引用计数器就减1。`只要对象A的引用计数器的值为0，即表示对象A不可能再被使用，可进行回收。`

- **缺点: 无法解决循环引用的问题, 所以Java没有采用这种方式**

  如下图所示，循环引用时，两个对象的计数都为1，导致两个对象都无法被释放。

  ![弊端](https://gitee.com/jobim/blogimage/raw/master/img/20210828211217.png)

  



### 2.1.2 可达性分析算法

* JVM中的垃圾回收器通过**可达性分析**来探索所有存活的对象
* 可达性分析算法是**以根对象集合（GCRoots）为起始点**，看能否沿着GC Root对象为起点的引用链找到该对象。如果目标对象**没有任何引用链相连，则是不可达的，就意味着该对象己经死亡，可以标记为垃圾对象**。



**GC Roots可以是哪些元素？**



* 虚拟机栈中引用的对象，比如：各个线程被调用的方法中使用到的参数、局部变量等。
* 本地方法栈内JNI（一般说的Native方法）引用的对象。
* 方法区中类静态属性引用的对象，比如：Java类的引用类型静态变量
* 方法区中常量引用的对象、字符串常量池（StringTable）里的引用
* 所有被同步锁synchronized持有的对象
* Java虚拟机内部的引用。



### 2.1.3 强、软、弱、虚四大引用

1. **强引用（StrongReference）**

   强引用是使用最普遍的引用。**如果一个对象具有强引用，那垃圾回收器绝不会回收它。**

   ```java
   Object o=new Object();   //  强引用
   o=null;     // 帮助垃圾收集器回收此对象
   ```

2. **软引用（SoftReference）**

   如果一个对象只具有软引用，则**内存空间足够，垃圾回收器就不会回收它**；如果**内存空间不足了，就会回收这些对象的内存**。只要垃圾回收器没有回收它，该对象就可以被程序使用。软引用可用来实现内存敏感的高速缓存。  

   **软引用的使用：**

   ```java
   public class Demo1 {
   	public static void main(String[] args) {
   		final int _4M = 4*1024*1024;
   		//使用软引用对象 list和SoftReference是强引用，而SoftReference和byte数组则是软引用
   		List<SoftReference<byte[]>> list = new ArrayList<>();
   		SoftReference<byte[]> ref= new SoftReference<>(new byte[_4M]);
   	}
   }
   ```

   **软引用结合引用队列使用：**（查看引用队列中有无软引用，如果有，则将该软引用从存放它的集合中移除（这里为一个list集合））

   ```java
   public class Demo1 {
   	public static void main(String[] args) {
   		final int _4M = 4*1024*1024;
   		//使用引用队列，用于移除引用为空的软引用对象
   		ReferenceQueue<byte[]> queue = new ReferenceQueue<>();
   		//使用软引用对象 list和SoftReference是强引用，而SoftReference和byte数组则是软引用
   		List<SoftReference<byte[]>> list = new ArrayList<>();
   		SoftReference<byte[]> ref= new SoftReference<>(new byte[_4M]);
   
   		//遍历引用队列，如果有元素，则移除
   		Reference<? extends byte[]> poll = queue.poll();
   		while(poll != null) {
   			//引用队列不为空，则从集合中移除该元素
   			list.remove(poll);
   			//移动到引用队列中的下一个元素
   			poll = queue.poll();
   		}
   	}
   }
   ```

3. **弱引用（WeakReference）**

   对于只有弱引用的对象来说，只要垃圾回收机制一运行，不管JVM的内存空间是否足够，都会回收该对象占用的内存。 

   ```java
   //垃圾回收机制一运行,会回收该对象占用的内存
   WeakReference<MyObject> weakReference = new WeakReference<>(new Object());
   ```

4. **虚引用（PhantomReference）**

   顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。如果一个对象仅持有虚引用，**那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收**，它不能单独使用也不能通过它访问对象。

   虚引用必须和引用队列 (ReferenceQueue)联合使用,当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在回收对象的内存之前，把这个虚引用加入到与之关联的引用队列中。

   ```java
   ReferenceQueue<MyObject> referenceQueue = new ReferenceQueue();
   //和引用队列进行关联，当虚引用对象被回收后，会进入ReferenceQueue队列中
   PhantomReference<MyObject> phantomReference = new PhantomReference<>(new MyObject(),referenceQueue);
   ```

   

注意：对于清除的时候是把起始和结束地址记录到一个空闲地址的列表里







## 2.2 垃圾回收算法

### 2.2.1 标记-清除算法

标记清除算法分为**“标记”和“清除”阶段**：**首先标记出所有不需要回收的对象，在标记完成后统一回收掉所有没有被标记的对象。**

![image-20210830154442969](https://gitee.com/jobim/blogimage/raw/master/img/20210830154443.png)



**何为清除？**

* 这里所谓的清除并不是真的置空，而是把需要清除的对象地址保存在空闲的地址列表里。下次有新对象需要加载时，判断垃圾的位置空间是否够，如果够，就存放覆盖原有的地址。



**标记-清除算法的缺点：**

* **容易产生大量的内存碎片**，可能无法满足大对象的内存分配，一旦导致无法分配对象，那就会导致jvm启动gc，一旦启动gc，我们的应用程序就会暂停，这就导致应用的响应速度变慢



### 2.2.2 标记-复制算法

* **将内存分为等大小的两个区域，FROM和TO（TO中为空）**。先**将被GC Root引用的对象从FROM放入TO中，再回收不被GC Root引用的对象。然后交换FROM和TO**。这样也可以避免内存碎片的问题，但是会占用双倍的内存空间。

  ![image-20210830160413661](https://gitee.com/jobim/blogimage/raw/master/img/20210830160413.png)



**优点**

* 没有标记和清除过程，实现简单，运行高效
* 复制过去以后保证空间的连续性，**不会出现“碎片”问题。**
* 特别适合垃圾对象很多，存活对象很少的场景



**缺点：**

* 可用的内存大小缩小为原来的一半，**对象存活率高时会频繁进行复制**





###  2.2.3 标记-整理算法

在**新生代中可以使用复制算法**，但是在老年代就不能选择复制算法了，因为老年代的对象存活率会较高，这样会有较多的复制操作，导致效率变低。标记-清除算法可以应用在老年代中，但是它效率不高，在内存回收后容易产生大量内存碎片。

因此就出现了一种**标记-整理算法（Mark-Compact）算法**，与标记-清除算法不同的是，在**标记可回收的对象后将所有存活的对象压缩到内存的一端**，使他们紧凑的排列在一起，**然后对端边界以外的内存进行回收**。

![image-20210830160525612](https://gitee.com/jobim/blogimage/raw/master/img/20210830160525.png)



**优点：**解决了标记-清理算法存在的内存碎片问题。

**缺点：**仍需要进行局部对象移动，一定程度上降低了效率。



### 2.2.4 分代收集算法



* 根据对象存活周期的不同将内存分为几块。一般将 **java 堆分为新生代和老年代**，这样我们就可以根据各个年代的特点选择合适的垃圾收集算法。

* 比如**在新生代中，每次收集都会有大量对象死去，所以可以选择”标记-复制“算法**，只需要付出少量对象的复制成本就可以完成每次垃圾收集。

* 而**老年代的对象存活几率是比较高**的，而且没有额外的空间对它进行分配担保，所以我们**必须选择“标记-清除”或“标记-整理”算法**进行垃圾收集。



**为什么要使用分代收集算法？**

- 不同的对象生命周期是不一样的, 因此, 不同生命周期的对象, 可以`采用不同的收集方式, 以便提高回收效率`
- 分为`年轻代、老年代`, 对这两种分代收集







**简述分代垃圾回收器是怎么工作的？**

* 分代回收器有两个分区：**老年代和新生代**。新生代默认的空间占比总空间的 1/3，老年代的默认占比是 2/3.

* 新创建的对象都被放在了**新生代的伊甸园**中

  ![image-20210830162451768](https://gitee.com/jobim/blogimage/raw/master/img/20210830162451.png)

* 当伊甸园中的内存不足时，就会进行一次垃圾回收，这时的回收叫做 **Minor GC**

* Minor GC 会将**伊甸园和幸存区FROM**存活的对象先复制到 **幸存区 TO**中， 并让其**寿命加1**，再**交换两个幸存区**

  ![image-20210830162913474](https://gitee.com/jobim/blogimage/raw/master/img/20210830162913.png)

* 再次创建对象，若新生代的伊甸园又满了，则会**再次触发 Minor GC**（会触发 **stop the world**， 暂停其他用户线程，只让垃圾回收线程工作），这时不仅会回收伊甸园中的垃圾，**还会回收幸存区中的垃圾**，再将活跃对象复制到幸存区TO中。回收以后会交换两个幸存区，并让幸存区中的对象**寿命加1**

  ![image-20210830163024539](https://gitee.com/jobim/blogimage/raw/master/img/20210830163024.png)

* 如果幸存区中的对象的**寿命超过某个阈值**（最大为15，4bit），就会被**放入老年代**中。大对象也会直接进入老年代。

  ![image-20210830163048070](https://gitee.com/jobim/blogimage/raw/master/img/20210830163048.png)

* 如果新生代老年代中的内存都满了，就会先触发Minor GC，再触发**Full GC**（使用标记整理或者标记清除算法），扫描**新生代和老年代中**所有不再使用的对象并回收

  







**System.gc() 方法 或 Runtime.getRuntime().gc()**

- **`主动触发Full GC, 同时对老年代和新生代进行回收，尝试释放被丢弃对象占用的内存`**



<font color='blue' size='4'>**JVM中年轻代里的对象什么情况下进入老年代？**</font>

* **默认的设置下，当对象的年龄达到15岁时，也就是躲过15次GC的时候，他就会转移到老年代里去**，具体是多少岁进入老年代，可以通过JVM参数“-XX:MaxTenuringThreshold”来设置，默认是15岁

* **大对象将会直接到老年代可以通过 -xx:+PretenuerSizeThreshold 来控制大对象的大小**
* 在一次安全Minor GC 中，仍然存活的对象不能在另一个Survivor完全容纳，则**会通过担保机制进入老年代**

* **动态对象年龄判断**，这里跟这个对象年龄有另外一个规则可以让对象进入老年代，不用等到15次GC过后才可以。

  规则运行：**年龄1+年龄2+年龄n的多个年龄对象总和超过了Survivor区的50%（默认），此时就会把年龄n以上的对象都放入老年代**

> 对于一些的 GC 算法，还可能直接在老年代上面分配，例如 G1 GC 中的 **humongous allocations（大对象分配）**，就是对象在超过 Region 一半大小的时候，直接在老年代的连续空间分配





## 2.3 垃圾回收器（难点）

### 2.3.1 垃圾收集器概述

**<font color='blue'>垃圾回收器的分类：</font>**

* 按线程数分（垃圾回收线程数），可以分为**串行垃圾回收器和并行垃圾回收器**。

  ![image-20210920113801064](https://gitee.com/jobim/blogimage/raw/master/img/20210920113801.png)
  **串行回收指的是在同一时间段内只允许有一个CPU用于执行垃圾回收操作，此时工作线程被暂停，直至垃圾收集工作结束。**

  * 在诸如单CPU处理器或者较小的应用内存等硬件平台不是特别优越的场合，串行回收器的性能表现可以超过并行回收器和并发回收器。所以，串行回收默认被应用在客户端的Client模式下的JVM中
  * 在并发能力比较强的CPU上，并行回收器产生的停顿时间要短于串行回收器

  **和串行回收相反，并行收集可以运用多个CPU同时执行垃圾回收，因此提升了应用的吞吐量，不过并行回收仍然与串行回收一样，采用独占式，使用了“Stop-the-World”机制。**

* 按照工作模式分，可以分为**并发式垃圾回收器和独占式垃圾回收器**。

  * 并发式垃圾回收器与应用程序线程交替工作，以尽可能减少应用程序的停顿时间。
  * 独占式垃圾回收器（Stop the World）一旦运行，就停止应用程序中的所有用户线程，直到垃圾回收过程完全结束。

  ![image-20210920114119563](https://gitee.com/jobim/blogimage/raw/master/img/20210920114119.png)

* 按碎片处理方式分，可分为**压缩式垃圾回收器和非压缩式垃圾回收器**。
  * 压缩式垃圾回收器会在回收完成后，对存活对象进行压缩整理，消除回收后的碎片。再分配对象空间使用指针碰撞
  * 非压缩式的垃圾回收器不进行这步操作，分配对象空间使用空闲列表

* 按工作的内存区间分，又可分为**年轻代垃圾回收器和老年代垃圾回收器**。



**<font color='blue'>评估 GC 的性能指标</font>**

>**吞吐量：运行用户代码的时间占总运行时间的比例（总运行时间 = 程序的运行时间 + 内存回收的时间）**
>
>垃圾收集开销：吞吐量的补数，垃圾收集所用时间与总运行时间的比例。
>
>**暂停时间：执行垃圾收集时，程序的工作线程被暂停的时间。Stop The World**
>
>收集频率：相对于应用程序的执行，收集操作发生的频率。
>
>**内存占用：Java堆区所占的内存大小。**
>
>快速：一个对象从诞生到被回收所经历的时间。

**主要两点：吞吐量和暂停时间**

**吞吐量（throughput）**

1. 吞吐量就是CPU用于运行用户代码的时间与CPU总消耗时间的比值，即吞吐量=运行用户代码时间 /（运行用户代码时间+垃圾收集时间）
   - 比如：虚拟机总共运行了100分钟，其中垃圾收集花掉1分钟，那吞吐量就是99%。
2. 这种情况下，应用程序能容忍较高的暂停时间，因此，高吞吐量的应用程序有更长的时间基准，快速响应是不必考虑的
3. 吞吐量优先，意味着在单位时间内，STW的时间最短：0.2+0.2=0.4

[![img](https://cdn.jsdelivr.net/gh/youthlql/lqlp@v1.2.0/JVM/chapter_012/0003.png)](https://cdn.jsdelivr.net/gh/youthlql/lqlp@v1.2.0/JVM/chapter_012/0003.png)

**暂停时间（pause time）**

1. “暂停时间”是指一个时间段内应用程序线程暂停，让GC线程执行的状态。
   - 例如，GC期间100毫秒的暂停时间意味着在这100毫秒期间内没有应用程序线程是活动的
2. 暂停时间优先，意味着尽可能让单次STW的时间最短：0.1+0.1 + 0.1+ 0.1+ 0.1=0.5，但是总的GC时间可能会长

[![img](https://cdn.jsdelivr.net/gh/youthlql/lqlp@v1.2.0/JVM/chapter_012/0004.png)](https://cdn.jsdelivr.net/gh/youthlql/lqlp@v1.2.0/JVM/chapter_012/0004.png)



标准：**在最大吞吐量优先的情况下，降低停顿时间**



<font color='blue'> **7款经典收集器：**</font>

![image-20210920114636297](https://gitee.com/jobim/blogimage/raw/master/img/20210920114636.png)

1. 两个收集器间有连线，表明它们可以搭配使用：
   - Serial/Serial old
   - Serial/CMS （JDK9废弃）
   - ParNew/Serial Old （JDK9废弃）
   - ParNew/CMS
   - Parallel Scavenge/Serial Old （预计废弃）
   - Parallel Scavenge/Parallel Old
   - G1
2. 其中Serial Old作为CMS出现”Concurrent Mode Failure”失败的后备预案。
3. （红色虚线）由于维护和兼容性测试的成本，在JDK 8时将Serial+CMS、ParNew+Serial Old这两个组合声明为废弃（JEP173），并在JDK9中完全取消了这些组合的支持（JEP214），即：移除。
4. （绿色虚线）JDK14中：弃用Parallel Scavenge和Serial Old GC组合（JEP366）
5. （青色虚线）JDK14中：删除CMS垃圾回收器（JEP363）


>新生代收集器：Serial、ParNew、Parallel Scavenge；
>
>老年代收集器：Serial old、Parallel old、CMS；
>
>整堆收集器：G1
>
><hr/>
>
>串行回收器：Serial、Serial old
>
>并行回收器：ParNew、Parallel Scavenge、Parallel old
>
>并发回收器：CMS、G1



### 2.3.2 Serial 回收器：串行回收

**Serial 回收器：串行回收**

1. Serial收集器是最基本、历史最悠久的垃圾收集器了。JDK1.3之前回收新生代唯一的选择。
2. **Serial收集器用于执行年轻代垃圾收集，采用复制算法、串行回收和”Stop-the-World”机制的方式执行内存回收**。
3. **Serial Old收集器执行老年代垃圾收集，采用了串行回收、标记-压缩算法和”Stop the World”机制**
4. Serial Old在Server模式下主要有两个用途：①在jdk1.5与新生代的Parallel Scavenge配合使用②作为老年代CMS收集器的后备垃圾收集方案

这个收集器是一个单线程的收集器，“单线程”的意义：**它只会使用一个CPU（串行）或一条收集线程去完成垃圾收集工作。更重要的是在它进行垃圾收集时，必须暂停其他所有的工作线程，直到它收集结束（Stop The World）**

![image-20210920120210866](https://gitee.com/jobim/blogimage/raw/master/img/20210920120210.png)

**Serial 回收器的优势**

1. 优势：**简单而高效**（与其他收集器的单线程比），对于限定单个CPU的环境来说，Serial收集器由于没有线程交互的开销，专心做垃圾收集自然可以获得最高的单线程收集效率。运行在Client模式下的虚拟机是个不错的选择。
2. 在HotSpot虚拟机中，使用`-XX:+UseSerialGC`参数可以指定年轻代和老年代都使用串行收集器。
   - 等价于新生代用Serial GC，且老年代用Serial Old GC



### 2.3.3 ParNew 回收器：并行回收

1. 如果说Serial GC是年轻代中的单线程垃圾收集器，而**ParNew收集器则是Serial收集器的多线程版本（用于年轻代）**。
   - Par是Parallel的缩写，New：只能处理新生代
2. ParNew 收集器除了采用**并行回收**的方式执行内存回收外，两款垃圾收集器之间几乎没有任何区别。ParNew收集器在年轻代中同样也是**采用复制算法、”Stop-the-World”机制**。
3. ParNew收集器其实跟Parallel收集器很类似，区别主要在于**它可以和CMS收集器配合使用**

![image-20210920121329389](https://gitee.com/jobim/blogimage/raw/master/img/20210920121329.png)



**ParNew 回收器与 Serial 回收器比较：**

1. ParNew收集器运行在多CPU的环境下，由于可以充分利用多CPU、多核心等物理硬件资源优势，可以更快速地完成垃圾收集，提升程序的吞吐量。

2. **但是在单个CPU的环境下，ParNew收集器不比Serial收集器更高效**。虽然Serial收集器是基于串行回收，但是由于CPU不需要频繁地做任务切换，因此可以有效避免多线程交互过程中产生的一些额外开销。

   

**设置 ParNew 垃圾回收器：**

1. 在程序中，开发人员可以通过选项”-XX:+UseParNewGC”手动指定使用ParNew收集器执行内存回收任务。它表示年轻代使用并行收集器，不影响老年代。
2. -XX:ParallelGCThreads限制线程数量，默认开启和CPU数据相同的线程数。

### 2.3.4 Parallel 回收器：吞吐量优先

**Parallel Scavenge回收器和Parallel Old回收器：**

* HotSpot的年轻代中**`Parallel Scavenge`收集器同样也采用了复制算法、并行回收和”Stop the World”机制**。

> 那么Parallel收集器的出现是否多此一举？
>
> * 和ParNew收集器不同，Parallel Scavenge收集器的目标则是达到一个**可控制的吞吐量**（Throughput），它也被称为**吞吐量优先的垃圾收集器**。
> * 自适应调节策略也是Parall el Scavenge与ParNew一个重要区别。（动态调整内存分配情况，以达到一个最优的吞吐量或低延迟）

* **`Parallel Old`收集器采用了标记-压缩算法，但同样也是基于并行回收和”Stop-the-World”机制。**
* **在Java8中，默认是此垃圾收集器（Parallel Scavenge+Parallel Old）**

![image-20210920122043812](https://gitee.com/jobim/blogimage/raw/master/img/20210920122043.png)



**Parallel Scavenge 回收器参数设置：**

1. `-XX:+UseParallelGC` 手动指定年轻代使用Parallel并行收集器执行内存回收任务。
2. `-XX:+UseParallelOldGC`：手动指定老年代都是使用并行回收收集器。
   - 分别适用于新生代和老年代
   - 上面两个参数分别适用于新生代和老年代。默认jdk8是开启的。默认开启一个，另一个也会被开启。（互相激活）
3. `-XX:ParallelGCThreads`：设置年轻代并行收集器的线程数。一般地，最好与CPU数量相等，以避免过多的线程数影响垃圾收集性能。
   1. 在默认情况下，当CPU数量小于8个，ParallelGCThreads的值等于CPU数量。
   2. 当CPU数量大于8个，ParallelGCThreads的值等于3+[5*CPU_Count]/8]
4. `-XX:MaxGCPauseMillis`： 设置垃圾收集器最大停顿时间（即STW的时间）。单位是毫秒。
   1. 为了尽可能地把停顿时间控制在XX:MaxGCPauseMillis 以内，收集器在工作时会调整Java堆大小或者其他一些参数。
   2. 对于用户来讲，停顿时间越短体验越好。但是在服务器端，我们注重高并发，整体的吞吐量。所以服务器端适合Parallel，进行控制。
   3. 该参数使用需谨慎。
5. `-XX:GCTimeRatio`垃圾收集时间占总时间的比例，即等于 1 / (N+1) ，用于衡量吞吐量的大小。
   1. 取值范围(0, 100)。默认值99，也就是垃圾回收时间占比不超过1。
   2. 与前一个-XX:MaxGCPauseMillis参数有一定矛盾性，STW暂停时间越长，Radio参数就容易超过设定的比例。

6. `-XX:+UseAdaptiveSizePolicy` 设置Parallel Scavenge收集器具有**自适应调节策略**
   1. 在这种模式下，年轻代的大小、Eden和Survivor的比例、晋升老年代的对象年龄等参数会被自动调整，已达到在堆大小、吞吐量和停顿时间之间的平衡点。
   2. 在手动调优比较困难的场合，可以直接使用这种自适应的方式，仅指定虚拟机的最大堆、目标的吞吐量（GCTimeRatio）和停顿时间（MaxGCPauseMillis），让虚拟机自己完成调优工作。



### 2.3.5 CMS 回收器：低延迟（重点）

**CMS 回收器：**

1. 在JDK1.5时期，Hotspot推出了一款在**强交互应用中（就是和用户打交道的引用）**几乎可认为有划时代意义的垃圾收集器：CMS（Concurrent-Mark-Sweep）收集器，**这款收集器是HotSpot虚拟机中第一款真正意义上的并发收集器，它第一次实现了让垃圾收集线程与用户线程同时工作。**
2. CMS（Concurrent Mark Sweep）收集器的关注点是尽可能缩短垃圾收集时用户线程的停顿时间。停顿时间越短（低延迟）就越适合与用户交互的程序，良好的响应速度能提升用户体验。
3. **CMS的垃圾收集算法采用标记-清除算法，并且也会”Stop-the-World”**



> * 不幸的是，CMS作为老年代的收集器，却无法与JDK1.4.0中已经存在的新生代收集器Parallel Scavenge配合工作（因为实现的框架不一样，没办法兼容使用），所以在JDK1.5中使用CMS来收集老年代的时候，新生代只能选择ParNew或者Serial收集器中的一个。

**CMS 工作原理（过程）**

![image-20210921091755810](https://gitee.com/jobim/blogimage/raw/master/img/20210921091755.png)

CMS主要分为4个主要阶段，即初始标记阶段、并发标记阶段、重新标记阶段和并发清除阶段。(涉及STW的阶段主要是：初始标记 和 重新标标记）

1. **初始标记（Initial-Mark）阶段：**在这个阶段中，程序中所有的工作线程都将会因为“Stop-the-World”机制而出现短暂的暂停，**这个阶段的主要任务仅仅只是标记出GC Roots能直接关联到的对象，速度非常快**。
2. **并发标记（Concurrent-Mark）阶段：** 从**GC Roots的直接关联对象开始遍历整个对象图**的过程，这个过程耗时较长但是不需要停顿用户线程**，**可以与垃圾收集线程一起并发运行。但在这个阶段结束，这个闭包结构并不能保证包含当前所有的可达对象。因为用户线程可能会不断的更新引用域，所以GC线程无法保证可达性分析的实时性。所以这个算法里会跟踪记录这些发生引用更新的地方。
3. **重新标记（Remark）阶段：**重新标记阶段就是**为了修正并发标记期间因为用户程序继续运行而导致标记产生变动的那一部分对象的标记记录**，这个阶段的停顿时间一般会比初始标记阶段的时间稍长，远远比并发标记阶段时间短。主要用到**三色标记里的增量更新算法**做重新标记。
4. **并发清除（Concurrent-Sweep）阶段：**开启用户线程，同时GC线程开始对未标记的区域做清扫。这个阶段如果有新增对象会被标记为黑色不做任何处理(见下面三色标记算法详解)。

5. **并发重置：**重置本次GC过程中的标记数据。



**为什么 CMS 不采用标记-压缩算法呢？**

> 答案其实很简答，因为当并发清除的时候，如果使用标记整理回收内存的话，在用户线程执行过程中把对象的地址修改了，那原来的用户线程使用的内存还怎么用呢？要保证用户线程能继续执行，前提的它运行的资源不受影响。Mark Compact更适合“stop the world”这种场景下使用。



**CMS的主要优点：`并发收集、低停顿`。但是它有下面几个明显的缺点：**

* **CMS收集器对CPU资源非常敏感**。在并发阶段，它虽然不会导致用户停顿，但是会因为占用了一部分线程而导致应用程序变慢，总吞吐量会降低。
* **无法处理浮动垃圾**(在并发标记和并发清理阶段又产生垃圾，这种浮动垃圾只能等到下一次gc再清理了)；
* 它使用的回收算法-“**标记-清除**”算法会导致收集结束时**会有大量空间碎片产生**，当然通过参数XX:+UseCMSCompactAtFullCollection可以让jvm在执行完标记清除后再做整理
  * 执行过程中的不确定性，会存在上一次垃圾回收还没执行完，然后垃圾回收又被触发的情况，特别是在并发标记和并发清理阶段会出现，一边回收，系统一边运行，也许没回收完就再次触发full gc，也就是"concurrent mode failure"，此时会进入stop the world，用serial old垃圾收集器来回收。



**CMS 参数配置：**

- `-XX:+UseConcMarkSweepGC`：启用cms
2. `-XX:ConcGCThreads`：并发的GC线程数
3. `-XX:+UseCMSCompactAtFullCollection`：FullGC之后做压缩整理（减少碎片）
- `-XX:CMSFullGCsBeforeCompaction`：多少次FullGC之后压缩一次，默认是0，代表每次FullGC后都会压缩一次
5. `-XX:CMSInitiatingOccupancyFraction`: 当老年代使用达到该比例时会触发FullGC（默认是92，这是百分比）
6. `-XX:+UseCMSInitiatingOccupancyOnly`：只使用设定的回收阈值(-XX:CMSInitiatingOccupancyFraction设
定的值)，如果不指定，JVM仅在第一次使用设定值，后续则会自动调整
- `-XX:+CMSScavengeBeforeRemark`：在CMS GC前启动一次minor gc，目的在于减少老年代对年轻代的引
  用，降低CMS GC的标记阶段时的开销，一般CMS的GC耗时 80%都在标记阶段
- `-XX:+CMSParallellnitialMarkEnabled`：表示在初始标记的时候多线程执行，缩短STW
- `-XX:+CMSParallelRemarkEnabled`：在重新标记的时候多线程执行，缩短STW;



**Serial GC、Parallel GC、Concurrent Mark Sweep GC这三个GC有什么不同呢？**

1. 如果你想要最小化地使用内存和并行开销，请选Serial GC；
2. 如果你想要最大化应用程序的吞吐量，请选Parallel GC；
3. 如果你想要最小化GC的中断或停顿时间，请选CMS GC。

**JDK 后续版本中 CMS 的变化：**

1. JDK9新特性：CMS被标记为Deprecate了（JEP291）
   - 如果对JDK9及以上版本的HotSpot虚拟机使用参数-XX:+UseConcMarkSweepGC来开启CMS收集器的话，用户会收到一个警告信息，提示CMS未来将会被废弃。
2. JDK14新特性：删除CMS垃圾回收器（JEP363）移除了CMS垃圾收集器，
   - 如果在JDK14中使用XX:+UseConcMarkSweepGC的话，JVM不会报错，只是给出一个warning信息，但是不会exit。JVM会自动回退以默认GC方式启动JVM

#### 2.3.5.1 三色标记算法

> 好的博客：[JVM 三色标记 增量更新 原始快照](https://www.cnblogs.com/hongdada/p/14578950.html)

在并发标记的过程中，因为标记期间应用线程还在继续跑，对象间的引用可能发生变化，多标和漏标的情况就有可能发生。这里需要引入“三色标记”，把Gcroots可达性分析遍历对象过程中遇到的对象， 按照“是否访问过”这个条件标记成以下三种颜色：

* **黑色：** **表示对象已经被垃圾收集器访问过， 且这个对象的所有引用都已经扫描过**。 黑色的对象代表已经扫描过， 它是安全存活的， 如果有其他对象引用指向了黑色对象， 无须重新扫描一遍。 黑色对象不可能直接（不经过灰色对象） 指向某个白色对象。
* **灰色：** **表示对象已经被垃圾收集器访问过， 但这个对象上至少存在一个引用还没有被扫描过**。
* **白色：** **表示对象尚未被垃圾收集器访问过**。 显然在可达性分析刚刚开始的阶段， 所有的对象都是白色的， 若在分析结束的阶段， 仍然是白色的对象， 即代表不可达

> 并发标记可能出现的问题

<font color='blue'>**多标-浮动垃圾：**</font>

假设已经遍历到E（变为灰色了），此时应用执行了 `objD.fieldE = null` 

![7779607-7a5ce353116237e2](https://gitee.com/jobim/blogimage/raw/master/img/20210921092009.webp)

D > E 的引用断开

此刻之后，对象E/F/G是“应该”被回收的。然而因为**E已经变为灰色**了，其仍会被**当作存活对象**继续遍历下去。最终的结果是：这部分对象仍会被标记为存活，即**本轮GC不会回收这部分内存**。

这部分本应该回收，但是没有回收到的内存，被称之为“**浮动垃圾**”。浮动垃圾并不会影响应用程序的正确性，只是需要等到下一轮垃圾回收中才被清除。

另外，针对并发标记开始后的**新对象**，通常的做法是直接全部**当成黑色**，本轮不会进行清除。这部分对象期间可能会变为垃圾，这也算是浮动垃圾的一部分。



<font color='blue'>**漏标-读写屏障：**</font>





![image-20210921092931928](https://gitee.com/jobim/blogimage/raw/master/img/20210921092932.png)



漏标会导致被引用的对象被当成垃圾误删除，这是严重bug，必须解决，有两种解决方案： **增量更新（Incremental Update） 和原始快照（Snapshot At The Beginning，SATB） 。**

* **`增量更新`就是当黑色对象插入新的指向白色对象的引用关系时， 就将这个新插入的引用记录下来， 等并发扫描结束之后， 再将这些记录过的引用关系中的黑色对象为根， 重新扫描一次**。 这可以简化理解为， 黑色对象一旦新插入了指向白色对象的引用之后， 它就变回灰色对象了。通过写屏障进行实现，即在进行赋值操作的时候，通过写屏障**记录修改之后的新引用**。
* **`原始快照`就是当灰色对象要删除指向白色对象的引用关系时， 就将这个要删除的引用记录下来， 在并发扫描结束之后，再将这些记录过的引用关系中的灰色对象为根， 重新扫描一次，这样就能扫描到白色的对象，将白色对象直接标记为黑色**(目的就是让这种对象在本轮gc清理中能存活下来，待下一轮gc的时候重新扫描，这个对象也有可能是浮动垃圾)以上无论是对引用关系记录的插入还是删除， 虚拟机的记录操作都是通过写屏障实现的，**记录修改之前对象的引用**。

对于读写屏障，以Java HotSpot VM为例，其并发标记时对漏标的处理方案如下：

- **CMS：写屏障 + 增量更新**
- **G1：写屏障 + SATB（原始快照）**
- **ZGC：读屏障**







#### 2.3.5.2 记忆集与卡表

<font color='blue'>**跨代引用：**</font>

* **在新生代做GCRoots可达性扫描过程中可能会碰到跨代引用的对象，这种如果又去对老年代再去扫描效率太低了。为此，在新生代可以引入记录集（Remember Set）的数据结构（记录从非收集区到收集区的指针集合），避免把整个老年代加入GCRoots扫描范围**。事实上并不只是新生代、 老年代之间才有跨代引用的问题， 所有涉及部分区域收集（Partial GC） 行为的垃圾收集器， 典型的如G1、 ZGC和Shenandoah收集器， 都会面临相同的问题。
* 垃圾收集场景中，收集器只需通过记忆集判断出某一块非收集区域是否存在指向收集区域的指针即可，无需了解跨代引用指针的全部细节。



<font color='blue'>**记忆集：**</font>

* 记忆集也叫rememberSet，垃圾收集器在新生代中建立了记忆集这样的数据结构，用来避免把整个老年代加入到GC ROOTS的扫描范围中。对于记忆集来说，我们可以理解为他是一个抽象类，那么具体实现它的方法将由子类去完成。

<font color='blue'>**卡表：**</font>

* 卡表(Card Table)是一种对记忆集的具体实现。卡表是使用一个字节数组实现：CARD_TABLE[ ]，每个元素对应着其标识的内存区域一块特定大小的内存块，称为“卡页”。每个卡页中可包含多个对象，只要有一个对象的字段存在跨代指针，其对应的卡表的元素标识就变成1，表示该元素变脏，否则为0。GC时，**只要筛选本收集区的卡表中变脏的元素加入GCRoots里**。

* 那么JVM对于卡页的维护也是通过**写屏障**的方式，这也就是为什么刚刚我们跟进写屏障操作到最后会发现它会对卡表进行一系列的操作。



### 2.3.6 G1 回收器

G1 (Garbage-First)是一款面向服务器的垃圾收集器,主要针对配备多颗处理器及大容量内存的机器.。以极高概率**满足GC停顿时间要求**的同时,还具备高吞吐量性能特征。

![image-20210921134735047](https://gitee.com/jobim/blogimage/raw/master/img/20210921134735.png)





* G1将Java堆划分为多个大小相等的独立区域（**Region**），JVM目标是不超过2048个Region(JVM源码里TARGET_REGION_NUMBER 定义)，实际可以超过该值，但是不推荐。一般Region大小等于堆大小除以2048，比如堆大小为4096M，则Region大小为2M，当然也可以用参数"-XX:G1HeapRegionSize"手动指定Region大小，但是推荐默认的计算方式。

* G1保留了年轻代和老年代的概念，但不再是物理隔阂了，它们都是（可以不连续）Region的集合。
* 默认年轻代对堆内存的占比是5%，如果堆大小为4096M，那么年轻代占据200MB左右的内存，对应大概是100个Region，可以通过“-XX:G1NewSizePercent”设置新生代初始占比，在系统运行中，JVM会不停的给年轻代增加更多的Region，但是最多新生代的占比不会超过60%，可以通过“-XX:G1MaxNewSizePercent”调整。年轻代中的Eden和Survivor对应的region也跟之前一样，默认8:1:1，假设年轻代现在有1000个region，eden区对应800个，s0对应100个，s1对应100个。
* 一个Region可能之前是年轻代，如果Region进行了垃圾回收，之后可能又会变成老年代，也就是说Region的区域功能可能会动态变化。
* **对大对象的处理**，G1有专门分配大对象的Region叫**Humongous区**，而不是让大对象直接进入老年代的Region中。在G1中，大对象的判定规则就是一个大对象超过了一个Region大小的50%，比如按照上面算的，每个Region是2M，只要一个大对象超过了1M，就会被放入Humongous中，而且一个大对象如果太大，可能会横跨多个Region来存放。



G1收集器一次GC(主要值Mixed GC)的运作过程大致分为以下几个步骤：

- **初始标记**（initial mark，STW）：暂停所有的其他线程，并记录下gc roots直接能引用的对象，**速度很快** ；

- **并发标记（Concurrent-Mark）：** 从**GC Roots的直接关联对象开始遍历整个对象图**的过程

- **重新标记（Remark）：**重新标记阶段就是**为了修正并发标记期间因为用户程序继续运行而导致标记产生变动的那一部分对象的标记记录**。

- **筛选回收**（Cleanup，STW）：筛选回收阶段首先对各个Region的**回收价值和成本进行排序**，**根据用户所期望的GC停顿STW时间(可以用JVM参数 -XX:MaxGCPauseMillis指定)来制定回收计划**。、

  比如说老年代此时有1000个Region都满了，但是因为根**据预期停顿时间**，本次垃圾回收可能只能停顿200毫秒，那么通过之前回收成本计算得知，可能回收其中800个Region刚好需要200ms，那么就只会回收800个Region(**Collection Set**，要回收的集合)，**回收算法主要用的是复制算法**，**将一个region中的存活对象复制到另一个region中，这种不会像CMS那样回收完因为有很多内存碎片还需要整理一次，G1采用复制算法回收几乎不会有太多内存碎片**。

![image-20210921145033401](https://gitee.com/jobim/blogimage/raw/master/img/20210921145033.png)





<font color='blue'>**Collect Set（智能收集）：**</font>

* 在G1里面会维护一个Collect Set集合类似一个优先列表，每次根据允许的收集时间，优先选择回收价值最大的Region，这样不仅每次可以优先收集垃圾最多的Region，还可以根据用户的设定之间来计算收集哪些Region达到用户所期望的垃圾收集时间。

<font color='blue'>**已记忆集合（RSet）：**</font>

* RSet记录了其他Region中的对象引用本Region中对象的关系，属于points-into结构（谁引用了我的对象）。RSet的价值在于使得垃圾收集器不需要扫描整个堆找到谁引用了当前分区中的对象，只需要扫描RSet即可。

* 如下图所示，Region1和Region3中的对象都引用了Region2中的对象，因此在Region2的RSet中记录了这两个引用。

  ![image-20210921145206621](https://gitee.com/jobim/blogimage/raw/master/img/20210921145206.png)



**<font color='blue'>G1垃圾收集分类：</font>**

**YoungGC**

* YoungGC并不是说现有的Eden区放满了就会马上触发，G1会计算下现在Eden区回收大概要多久时间，如果回收时间远远小于参数 -XX:MaxGCPauseMills 设定的值，那么增加年轻代的region，继续给新对象存放，不会马上做Young GC，直到下一次Eden区放满，G1计算回收时间接近参数 -XX:MaxGCPauseMills 设定的值，那么就会触发Young GC

**MixedGC**

* 不是FullGC，老年代的堆占有率达到参数(**-XX:InitiatingHeapOccupancyPercent**)设定的值则触发，**回收所有的Young和部分Old**(根据期望的GC停顿时间确定old区垃圾收集的优先顺序)以及**大对象区**，正常情况G1的垃圾收集是先做MixedGC，主要使用复制算法，需要把各个region中存活的对象拷贝到别的region里去，拷贝过程中如果发现**没有足够的空region**能够承载拷贝对象就会触发一次Full GC

**Full GC**

* 停止系统程序，然后采用单线程进行标记、清理和压缩整理，好空闲出来一批Region来供下一次MixedGC使用，这个过程是非常耗时的。(Shenandoah优化成多线程收集了)

<font color='blue'>**可预测的停顿时间模型Pause Prediction Model：**</font>

* G1除了追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒。



**<font color='blue'>如何选择垃圾收集器：</font>**

1. 优先调整堆的大小让服务器自己来选择
2. 如果内存小于100M，使用串行收集器
3. 如果是单核，并且没有停顿时间的要求，串行或JVM自己选择
4. 如果允许停顿时间超过1秒，选择并行或者JVM自己选
5. 如果响应时间最重要，并且不能超过1秒，使用并发收集器
6. **4G以下可以用parallel，4-8G可以用ParNew+CMS，8G以上可以用G1，几百G以上用ZGC**



**<font color='blue'>安全点与安全区域：</font>**

**安全点：**

* **概念：程序执行时并非在所有地方都能停顿下来开始GC，只有在特定的位置才能停顿下来开始GC，这些位置称为“安全点**

* 这些特定的安全点位置主要有以下几种:
  * 方法返回之前
  * 调用某个方法之后
  * 抛出异常的位置
  * 循环的末尾

* 大体实现思想是当垃圾收集需要中断线程的时候， 不直接对线程操作， 仅仅简单地设置一个标志位， 各个线程执行过程时会不停地主动去轮询这个标志， 一旦发现中断标志为真时就自己在最近的安全点上主动中断挂起。 轮询标志的地方和安全点是重合的。

* 安全点应用场景，GC STW、偏向锁释放等

**安全区域：**

* 安全点机制保证了程序执行的时候，在不太长的时间就会遇到可进入gc的安全点。但是**如果线程处于sleep状态或者blocked状态的时候，这时线程无法响应jvm的中断请求，就需要安全区域**。

* 安全区域是指在一段代码片段中，引用关系不会发生变化，在该区域的任何地方发生gc都是安全的。
  当代码执行到安全区域时，首先标示自己已经进入了安全区域，那样如果在这段时间里jvm发起gc，就不用管标示自己在安全区域的那些线程了，在线程离开安全区域时，会检查系统是否正在执行gc，如果是那么就等到gc完成后再离开安全区域。



## 2.4 垃圾回收调优

**相关 JVM 参数**

| 含义       | 参数                         |
| ---------- | ---------------------------- |
| 堆初始大小 | -Xms                         |
| 堆最大大小 | -Xmx 或 -XX:MaxHeapSize=size |
| 新生代大小         | -Xmn 或 (-XX:NewSize=size + -XX:MaxNewSize=size )            |
| 幸存区比例（动态） | -XX:InitialSurvivorRatio=ratio 和 -XX:+UseAdaptiveSizePolicy |
| 幸存区比例 | -XX:SurvivorRatio=ratio            |
| 晋升阈值   | -XX:MaxTenuringThreshold=threshold |
| 晋升详情          | -XX:+PrintTenuringDistribution  |
| GC详情            | -XX:+PrintGCDetails -verbose:gc |
| FullGC 前 MinorGC | -XX:+ScavengeBeforeFullGC       |



### 2.4.1 新生代调优

* 新生代内存要设置合理
  * 新生代内存太小：频繁触发 Minor GC ，会 STW ，会使得吞吐量下降
  * 新生代内存太大：老年代内存占比有所降低，会更频繁地触发 Full GC。而且触发 Minor GC 时，清理新生代所花费的时间会更长
  * 新生代内存设置为内容纳`[并发量*(请求-响应)]`的数据为宜



- 晋升阈值配置得当，让长时间存活的对象尽快晋升

  ```java
  XX:MaxTenuringThreshold=threshold //设置最大晋升阈值
  -XX:+PrintTenuringDistribution //打印晋升的详细信息
  ```

  

### 2.4.2 老年代调优

以 CMS 为例：

- CMS 的老年代内存越大越好

- 先尝试不做调优，如果没有 Full GC 那么已经，否者先尝试调优新生代。

- 观察发现 Full GC 时老年代内存占用，将老年代内存预设调大 1/4 ~ 1/3

  ```java
  -XX:CMSInitiatingOccupancyFraction=percent //设置为75%
  ```

  



**StringTable调优**

* 因为StringTable是由HashTable实现的，所以可以**适当增加HashTable桶的个数**，来减少字符串放入串池所需要的时间

  ```
  -XX:StringTableSize=xxxxCopy
  ```

* 考虑是否需要将字符串对象入池，可以通过**intern方法减少重复入池**





* JDK 1.8 的时候，方法区（HotSpot 的永久代）被彻底移除了（JDK1.7 就已经开始了），取而代之是元空间，元空间使用的是本地内存。需要使用下面的参数：

  ```
  -XX:MetaspaceSize=N //设置 Metaspace 的初始（和最小大小）
  -XX:MaxMetaspaceSize=N //设置 Metaspace 的最大大小
  ```

  与永久代很大的不同就是，如果不指定大小的话，随着更多类的创建，虚拟机会耗尽所有可用的系统内存。





# 三、类加载

类加载器子系统负责从文件系统或者网络中加载Class文件，class文件在文件开头有特定的文件标识。
ClassLoader只负责class文件的加载，至于它是否可以运行，则由Execution Engine决定。
加载的类信息存放于一块称为方法区的内存空间。除了类的信息外，方法区中还会存放运行时常量池信息，可能还包括字符串字面量和数字常量（这部分常量信息是Class文件中常量池部分的内存映射）



## 3.1 类加载过程



**类加载的流程**：**`加载 --> 链接（验证 --> 准备 --> 解析） --> 初始化`**

![image-20210902131359326](https://gitee.com/jobim/blogimage/raw/master/img/20210902131359.png) 

### 3.1.1 加载阶段

1. **通过一个类的`全限定名`获取定义此类的`二进制字节流`**

2. 将这个字节流所代表的静态存储结构转化为**运行时数据结构的方法区**

3. **在内存中生成一个代表这个类的`java.lang.Class`对象**，作为`方法区这个类的各种数据的访问入口`

4. 如果这个类还有父类没有加载，**先加载父类**。内部采用 C++ 的 instanceKlass 描述 java 类，它的重要 ﬁeld 有：

   1. _java_mirror 即 java 的类镜像，例如对 String 来说，它的镜像类就是 String.class，作用是把 klass 暴露给 java 使用_
   2. super 即父类
   3. _ﬁelds 即成员变量_
   4. methods 即方法
   5. _constants 即常量池_
   6. class_loader 即类加载器
   7. _vtable 虚方法表_
   8. itable 接口方法

   ![image-20210902130113144](https://gitee.com/jobim/blogimage/raw/master/img/20210902130113.png)
   
   ![image-20211022153745945](https://gitee.com/jobim/blogimage/raw/master/img/20211022153746.png)

- 类的对象在对象头中保存了*.class的地址（类型指针）。让对象可以通过其找到方法区中的instanceKlass，从而获取类的各种信息





### 3.1.2 链接阶段

* **`链接`分为三个子阶段：`验证 --> 准备 --> 解析`**

  ![image-20210902131601952](https://gitee.com/jobim/blogimage/raw/master/img/20210902131602.png)



**`验证`：就是验证字节码文件(class文件)是否合法**

1. **目的在于确保Class文件的字节流中包含信息符合当前虚拟机要求，保证被加载类的正确性，不会危害虚拟机自身安全**
2. 主要包括四种验证，`文件格式验证`，元数据验证，字节码验证，符号引用验证。



**`准备`：**

* **为类变量(static修饰的变量)分配内存并且设置该类变量的默认初始值，即零值**。static变量在分配空间和赋值是在两个阶段完成的。分配空间在准备阶段完成，赋值在初始化阶段完成
  * 如果 static 变量是 ﬁnal 的**基本类型**，以及**字符串常量**，那么编译阶段值就确定了，**赋值在准备阶段完成**
  * 如果 static 变量是 ﬁnal 的，但属于**引用类型**，那么赋值也会在**初始化阶段完成**



**解析**

* **将常量池中的符号引用解析为直接引用**，包括类或接口的解析，字段解析，类方法解析，接口方法解析

- 未解析时，常量池中的看到的对象仅是符号，未真正的存在于内存中
- 解析以后，会将常量池中的符号引用解析为直接引用

### 3.1.3 初始化阶段

1. **初始化阶段就是执行`类构造器clinit()方法`的过程**
2. 此方法不需定义，是**javac编译器自动收集类中的所有类变量的赋值动作和静态代码块中的语句合并而来**。也就是说，当我们代码中包含static变量的时候，就会有clinit方法。&lt;clinit>()不同于类的构造器。（关联：构造器是虚拟机视角下的&lt;init>()）
3. &lt;clinit>()方法中的**指令按语句在源文件中出现的顺序执行**
4. 若该类具有父类，**JVM会保证子类的&lt;clinit>()执行前，父类的&lt;clinit>()已经执行完毕**
5. 虚拟机必须**保证一个类的&lt;clinit>()方法在多线程下被同步加锁**

**发生时机：**

* **类的初始化的懒惰的**，以下情况会初始化
  * main 方法所在的类，总会被首先初始化
  * 首次访问这个类的静态变量或静态方法时
  * 子类初始化，如果父类还没初始化，会引发
  * 子类访问父类的静态变量，只会触发父类的初始化
  * Class.forName
  * new 会导致初始化

* 以下情况不会初始化
  * 访问类的 static ﬁnal 静态常量（基本类型和字符串）
  * 类对象.class 不会触发初始化
  * 创建该类对象的数组
  * 类加载器的.loadClass方法
  * Class.forNamed的参数2为false时

> **验证类是否被初始化，可以看该类的静态代码块是否被执行**



* 测试类是否被初始化代码：

  ```java
  public class Load1 {
      static {
          System.out.println("main init");
      }
      public static void main(String[] args) throws ClassNotFoundException {
          // 1. 静态常量（基本类型和字符串）不会触发初始化
  //         System.out.println(B.b);
          // 2. 类对象.class 不会触发初始化
  //         System.out.println(B.class);
          // 3. 创建该类的数组不会触发初始化
  //         System.out.println(new B[0]);
          // 4. 不会初始化类 B，但会加载 B、A
  //         ClassLoader cl = Thread.currentThread().getContextClassLoader();
  //         cl.loadClass("cn.ali.jvm.test.classload.B");
          // 5. 不会初始化类 B，但会加载 B、A
  //         ClassLoader c2 = Thread.currentThread().getContextClassLoader();
  //         Class.forName("cn.ali.jvm.test.classload.B", false, c2);
  
  
          // 1. 首次访问这个类的静态变量或静态方法时
  //         System.out.println(A.a);
          // 2. 子类初始化，如果父类还没初始化，会引发
  //         System.out.println(B.c);
          // 3. 子类访问父类静态变量，只触发父类初始化
  //         System.out.println(B.a);
          // 4. 会初始化类 B，并先初始化类 A
  //         Class.forName("cn.ali.jvm.test.classload.B");
      }
  
  }
  
  class A {
      static int a = 0;
      static {
          System.out.println("a init");
      }
  }
  class B extends A {
      final static double b = 5.0;
      static boolean c = false;
      static {
          System.out.println("b init");
      }
  }
  ```

* 典型应用，内部类的单例模式

  ```java
  public class Singleton {
  
      private Singleton() { } 
      // 内部类中保存单例
      private static class LazyHolder { 
          static final Singleton INSTANCE = new Singleton(); 
      }
      // 第一次调用 getInstance 方法，才会导致内部类加载和初始化其静态成员 
      public static Singleton getInstance() { 
          return LazyHolder.INSTANCE; 
      }
  }
  ```

  

## 3.2 类加载器



* Java虚拟机设计团队有意把类加载阶段中的**“通过一个类的全限定名来获取描述该类的二进制字节流”**这个动作放到Java虚拟机外部去实现，以便让应用程序自己决定如何去获取所需的类。实现这个动作的代码被称为**“类加载器”**（ClassLoader）
* 对于任意一个类，都必须由加载它的**类加载器**和这个**类本身**一起共同确立其在Java虚拟机中的唯一性，每一个类加载器，都拥有一个独立的类名称空间。这句话可以表达得更通俗一些：**比较两个类是否“相等”，只有在这两个类是由同一个类加载器加载的前提下才有意义**，否则，即使这两个类来源于同一个Class文件，被同一个Java虚拟机加载，只要加载它们的类加载器不同，那这两个类就必定不相等

以JDK 8为例

| 名称                                      | 加载的类              | 说明              |
| ----------------------------------------- | --------------------- | ----------------- |
| Bootstrap ClassLoader（启动类加载器）     | JAVA_HOME/jre/lib     | 无法直接访问      |
| Extension ClassLoader(拓展类加载器)       | JAVA_HOME/jre/lib/ext | 上级为Bootstrap   |
| Application ClassLoader(应用程序类加载器) | classpath             | 上级为Extension   |
| 自定义类加载器                            | 自定义                | 上级为Application |








### 3.2.1 启动类加载器

* 启动类加载器主要加载的是JVM自身需要的类，这个类加载使用C++语言实现的，是虚拟机自身的一部分，它负责将 **<JAVA_HOME>/lib路径下的核心类库（rt.jar，resources.jar、charsets.jar）**或-Xbootclasspath参数指定的路径下的jar包加载到内存中(出于安全考虑，Bootstrap启动类加载器只加载包名为java、javax、sun等开头的类)。

### 3.2.2 拓展类加载器

* 拓展类加载器是指 Sun公司实现的**sun.misc.Launcher$ExtClassLoader类**，由Java语言实现的，是Launcher的静态内部类，它负责**加载<JAVA_HOME>/lib/ext目录下**或者由系统变量-Djava.ext.dir指定位路径中的类库。开发者可以直接使用标准扩展类加载器。

* 如果classpath和JAVA_HOME/jre/lib/ext 下有同名类，加载时会使用**拓展类加载器**加载。当应用程序类加载器发现拓展类加载器已将该同名类加载过了，则不会再次加载

### 3.2.3 应用程序类加载器（系统类加载器）

* 应用程序加载器是指 Sun公司实现的sun.misc.Launcher$AppClassLoader。它负责加载系统类路径`java -classpath`或`-D java.class.path` 指定路径下的类库，也就是我们经常用到的classpath路径，开发者可通过ClassLoader.getSystemClassLoader()方法直接获，故又称为系统类加载器。当应用程序**没有自定义类加载器时，默认采用该类加载器**。



### 3.2.4 自定义类加载器



**使用场景：**

- 系统的ClassLoader只会加载指定目录下的class文件，如果你想加载自己的class文件,那么就可以自定义一个ClassLoader。
- 通过接口来使用实现，希望解耦时，常用在框架设计
- 这些类希望予以隔离，不同应用的同名类都可以加载，不冲突，常见于 tomcat 容器



**步骤：**

1. **继承ClassLoader父类**
2. **重写 ﬁndClass()方法**。如果重写loadClass() 方法，将打破双亲委派机制。
3. **读取类文件的字节码**
4. 在findClass()方法中**调用ClassLoader超类的`defineClass`方法**，向虚拟机提供字节码
5. 使用者调用该类加载器的 loadClass 方法



**代码示例：**

```java
class MyClassLoader extends ClassLoader {

    @Override // name 就是类名称
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        String path = "e:\\myclasspath\\" + name + ".class";

        try {
            ByteArrayOutputStream os = new ByteArrayOutputStream();
            Files.copy(Paths.get(path), os);

            // 读取类文件，得到字节数组
            byte[] bytes = os.toByteArray();

            // 调用defineClass方法，将byte[] -> *.class
            return defineClass(name, bytes, 0, bytes.length);

        } catch (IOException e) {
            e.printStackTrace();
            throw new ClassNotFoundException("类文件未找到", e);
        }
    }
}

public class Load7 {

    public static void main(String[] args) throws Exception {
        MyClassLoader classLoader = new MyClassLoader();
        Class<?> c1 = classLoader.loadClass("F");
        Class<?> c2 = classLoader.loadClass("HelloWorld");

        System.out.println(c1 == c2);

        MyClassLoader classLoader2 = new MyClassLoader();
        Class<?> c3 = classLoader2.loadClass("F");
        Class<?> c4 = classLoader2.loadClass("HelloWorld");
        System.out.println(c1 == c3);

        c1.newInstance();
    }
}
```





## 3.3 双亲委派机制



* Java虚拟机对class文件采用的是按需加载(需要的时候才加载)的方式，也就是说当需要使用该类时才会将它的class文件加载到内存生成class对象。而且**加载某个类的class文件时，Java虚拟机采用的是双亲委派模式，即把请求交由父类处理，它是一种任务委派模式**



1. 如果**一个类加载器收到了类加载请求，首先判断被加载的类是否已经加载过，如果没有加载过，它并不会自己先去加载，而是把这个请求委托给父类的加载器去执行；**
2. 如果**父类加载器还存在其父类加载器**，则**进一步向上委托，依次递归**，请求最终将到达顶层的**启动类加载器**；
3. 如果父类加载器可以完成类加载任务，就成功返回，倘若**父类加载器无法完成此加载任务，子加载器才会尝试自己去加载**，这就是双亲委派模式。
4. **父类加载器一层一层往下分配任务，如果子类加载器能加载，则加载此类，如果将加载任务分配至系统类加载器也无法加载此类，则抛出异常**



![image-20210902160438712](https://gitee.com/jobim/blogimage/raw/master/img/20210902160439.png)





**原理分析：**

ClassLoader中的loadClass()方法，loadClass()方法是ClassLoader类自己实现的，该方法中的逻辑就是双亲委派模式的实现，

```java
protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
{
    synchronized (getClassLoadingLock(name)) {
        // 先从缓存查找该class对象，找到就不用重新加载
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            long t0 = System.nanoTime();
            try {
                if (parent != null) {
                    //如果找不到，则委托给父类加载器去加载。递归调用loadClass()方法
                    c = parent.loadClass(name, false);
                } else {
                    //如果没有父类，则委托给启动加载器去加载
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // ClassNotFoundException thrown if class not found
                // from the non-null parent class loader
            }

            if (c == null) {
                // If still not found, then invoke findClass in order
                // 如果都没有找到，则通过自定义实现的findClass去查找并加载
                c = findClass(name);

                // this is the defining class loader; record the stats
                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                sun.misc.PerfCounter.getFindClasses().increment();
            }
        }
        if (resolve) {//是否需要在加载时进行解析
            resolveClass(c);
        }
        return c;
    }
}
```





**双亲委派机制的优势**

1. 首先，保证了java核心库的安全性。如果你也写了一个java.lang.String类，那么JVM只会按照上面的顺序加载jdk自带的String类，而不是你写的String类。
2. 其次，还能保证同一个类不会被加载多次。**避免类的重复加载**



**ClassLoader中的loadClass()、findClass()、defineClass()区别？**

* loadClass() 就是主要进行类加载的方法，默认的双亲委派机制就实现在这个方法中；
* findClass() 根据名称或位置加载.class字节码；
* definclass() 把字节码··为java.lang.Class；



