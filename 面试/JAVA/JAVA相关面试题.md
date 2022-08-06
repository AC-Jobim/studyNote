

# Java和C++的区别

* 都是面向对象的语言，都支持封装、继承和多态
* Java不提供指针来直接访问内存，程序内存更加安全
* Java的类是单继承的，C++支持多重继承；虽然Java的类不可以多继承，但是接口可以多继承。
* Java有自动内存管理机制，不需要程序员手动释放无用内存



# Java语法

## Java中的访问修饰符

**分类：**

* private : 在同一类内可见。使用对象：变量、方法。 注意：不能修饰类（外部类）
* default (即缺省，什么也不写，不使用任何关键字）: 在同一包内可见，不使用任何修饰符。使用对象：类、接口、变量、方法。
* protected : 对同一包内的类和所有子类可见。使用对象：变量、方法。 注意：不能修饰类（外部类）。
* public : 对所有类可见。使用对象：类、接口、变量、方法

**访问修饰符图：**

![image-20210918235901190](https://gitee.com/jobim/blogimage/raw/master/img/20210918235901.png)



# 面向对象专题

## 怎么理解面向对象

面向对象最主要的三个特征：

- 继承
- 封装
- 多态

**封装**

封装是指把一个对象的状态信息（也就是属性）隐藏在对象内部，不允许外部对象直接访问对象的内部信息。但是可以提供一些可以被外界访问的方法来操作属性。但是如果一个类没有提供给外界访问的方法，那么这个类也没有什么意义了。

**继承**

通过使用继承，可以快速地创建新的类，可以提高代码的重用，程序的可维护性，节省大量创建新类的时间 ，提高我们的开发效率。

关于继承如下 3 点请记住：

1. 子类拥有父类对象所有的属性和方法（包括私有属性和私有方法），但是父类中的私有属性和方法子类是无法访问，只是拥有。
2. 子类可以拥有自己属性和方法，即子类可以对父类进行扩展。
3. 子类可以用自己的方式实现父类的方法。（以后介绍）。

**多态**

多态，顾名思义，表示一个对象具有多种的状态。具体表现为父类的引用指向子类的实例。

多态的特点:

- 对象类型和引用类型之间具有继承（类）/实现（接口）的关系；
- 对象类型不可变，引用类型可变；
- 方法具有多态性，属性不具有多态性；
- 引用类型变量发出的方法调用的到底是哪个类中的方法，必须在程序运行期间才能确定；
- 多态不能调用“只在子类存在但在父类不存在”的方法；
- 如果子类重写了父类的方法，真正执行的是子类覆盖的方法，如果子类没有覆盖父类的方法，执行的是父类的方法。





## 重载（Overload）和重写（Override）的区别？

**重载**：发生在同一个类中，方法名相同参数列表不同（参数类型不同、个数不同、顺序不同），与方法返回值和访问修饰符无关，即重载的方法不能根据返回类型进行区分。

**重写**：发生在父子类中，方法名、参数列表必须相同，返回值小于等于父类，抛出的异常小于等于父类，访问修饰符大于等于父类；如果父类方法访问修饰符为private则子类中就不是重写。





## 继承情况下在创建类的对象时的执行顺序

1. 父类静态成员和静态初始化快，按在代码中出现的顺序依次执行。
2. 子类静态成员和静态初始化块，按在代码中出现的顺序依次执行。
3. 父类的实例成员和实例初始化块，按在代码中出现的顺序依次执行。
4. 执行父类的构造方法。
5. 子类实例成员和实例初始化块，按在代码中出现的顺序依次执行。
6. 执行子类的构造方法。

 **静态代码块、构造代码块，构造方法的执行顺序：**
父类静态代码块→子类静态代码块→父类构造代码块→父类构造方法→子类构造代码块→子类构造方法 



# Java基本数据类型和引用数据类型

## Java8种基本数据类型

![image-20211013113918834](https://gitee.com/jobim/blogimage/raw/master/img/20211013113925.png)

**注意：`引用数据类型`本身的大小和操作系统的位数有关，在64位平台上，占8个字节，在32位平台上占4个字节**



**Java数组的内存结构？**

* 对于基本类型数组而言，数组元素的值直接存储在对应的数组元素中

* 引用类型数组的数组元素依然是引用类型的，因此数组元素里存储的还是引用，它指向另一块内存，这块内存里存储了该引用变量所引用的对象（包括数组和Java 对象）





## 基本数据类型的包装类型

**Int和Integer类型的比较？**

```java
//自动拆箱：和基本数据类型比较时自动变成基本数据类型
int c = 128;
Integer cc = 128;
Integer ccc = new Integer(128);
System.out.println(c == cc);//true
System.out.println(c == ccc);//true
System.out.println(cc == ccc);//false
```

**包装类数据缓存：**

* **byte、short、int和long所对应包装类的数据缓存范围为 -128~127（包括-128和127）**

* float和double所对应的包装类没有数据缓存范围
* **char所对应包装类的数据缓存范围为 0~127（包括0和127）**
* boolean所对应包装类的数据缓存为true和false

```java
Integer b1 = 127;
Integer b2 = 127;
System.out.println(b1 == b2); //true
b1 = 128;
b2 = 128;
System.out.println(b1 == b2); //false
```

**原理（以Integer为例）：Integer将[-128,127]之前的所有包装对象提前创建好，放到了方法区中，之后再使用到区间的数据就不需要在new了，直接从方法区中取出来。**

![image-20211013144700462](https://gitee.com/jobim/blogimage/raw/master/img/20211013144700.png)





## byte和short类型向加减问题

```java
public class Test {
	public static void main(String[] args) {
		short b1 = 1,b2 =2,b3,b6,b8;
		final byte b4 = 4,b5 = 6,b7;
		int b;
		//b3 = (b1 + b2);//编译出错
		b3 = (byte)(b1 + b2);
		b3 = (b4+b5);
		//b3 = (b4+b1);//编译出错
		//b7 = 128;//编译出错
		b = (b1 + b2);
	}
}
```
**1.b3 = (b1 + b2);**
会报错，因为byte类型再相加的时候，会自动转换成int类型，右边的int类型赋值给byte类型便会报错，加上强制类型转换可以通过编译。比如b = (byte)(b1+b2);

**2.b3 = (b4+b5);**
final修饰的为常量，所以编译前会检查两个常量的和是否越界。

**3.b3 = (b4+b1);**
编译报错，byte类型和常量相加的时候，因为不知道结果会不会溢出，于是会先转化为int类型再进行运算，所以在这条语句中 b + 1 实际上是int类型，所以不能复制给byte。

**4.b7 = 128;**
会报错，因为右边超过了byte的存储范围，强制类型转换 b = (byte)128；可以通过编译，但是会造成数据丢失。
只要赋予byte的值不超过byte的取值范围，系统都会自动帮你转换，效果和byte test = (byte) 127一样；
当赋予的值超过了byte类型的取值范围，那就要手动进行数据类型转换了，不然系统程序就报错了

**5.b++；**
不会报错，b++ 等价于 b = (byte)(b+1) ，因为底层会自动进行强制类型转换。

short类型和byte类型的运行结果一样。



## String,StringBuffer,StringBuilder区别?

|          | String         | StringBuffer           | StringBuilder          |
| -------- | -------------- | ---------------------- | ---------------------- |
| 执行速度 | 最差           | 其次                   | 最高                   |
| 线程安全 | 线程安全       | 线程安全               | 线程不安全             |
| 使用场景 | 少量字符串操作 | 多线程环境下的大量操作 | 单线程环境下的大量操作 |

* 对于String来说，是把数据存放在了常量池中，因为所有的String，默认都是以常量形式保存。底层使用 private final char value[] 数组。
* 对于StringBuffer来说更多的考虑到了多线程的情况，在进行字符串操作的时候，它使用了synchronize关键字，对方法进行了同步处理。 而StringBuilder是非线程安全的。
* StringBuffer和StringBuilder的初始化容量为16，当存满之后会扩容。  默认情况下，扩容为原来容量的2倍 + 2 

## Object 类的常见方法总结

Object 类是一个特殊的类，是所有类的父类。它主要提供了以下 11 个方法：

```java
public final native Class<?> getClass()//native方法，用于返回当前运行时对象的Class对象，使用了final关键字修饰，故不允许子类重写。

public native int hashCode() //native方法，用于返回对象的哈希码，主要使用在哈希表中，比如JDK中的HashMap。
public boolean equals(Object obj)//用于比较2个对象的内存地址是否相等，String类对该方法进行了重写用户比较字符串的值是否相等。

protected native Object clone() throws CloneNotSupportedException//naitive方法，用于创建并返回当前对象的一份拷贝。一般情况下，对于任何对象 x，表达式 x.clone() != x 为true，x.clone().getClass() == x.getClass() 为true。Object本身没有实现Cloneable接口，所以不重写clone方法并且进行调用的话会发生CloneNotSupportedException异常。

public String toString()//返回类的名字@实例的哈希码的16进制的字符串。建议Object所有的子类都重写这个方法。

public final native void notify()//native方法，并且不能重写。唤醒一个在此对象监视器上等待的线程(监视器相当于就是锁的概念)。如果有多个线程在等待只会任意唤醒一个。

public final native void notifyAll()//native方法，并且不能重写。跟notify一样，唯一的区别就是会唤醒在此对象监视器上等待的所有线程，而不是一个线程。

public final native void wait(long timeout) throws InterruptedException//native方法，并且不能重写。暂停线程的执行。注意：sleep方法没有释放锁，而wait方法释放了锁 。timeout是等待时间。

public final void wait(long timeout, int nanos) throws InterruptedException//多了nanos参数，这个参数表示额外时间（以毫微秒为单位，范围是 0-999999）。 所以超时的时间还需要加上nanos毫秒。

public final void wait() throws InterruptedException//跟之前的2个wait方法一样，只不过该方法一直等待，没有超时时间这个概念

protected void finalize() throws Throwable { }//实例被垃圾回收器回收的时候触发的操作
```



## Java中创建对象的五种方式

**1、new关键字**

这是我们最常见的**也是最简单**的创建对象的方式，通过这种方式我们还可以调用任意的构造器（无参的和有参的）

**2、Class.newInstance**

* 运用反射创建对象时最常用的方法。**Class类的newInstance**使用的是类的`public的无参构造器`。因此也就是说使用此方法创建对象的前提是**必须有public的无参构造器才行**

  ```java
  public class Main {
      public static void main(String[] args) throws Exception {
          Person person = Person.class.newInstance();
          System.out.println(person); // Person{name='null', age=null}，Person类必须有public的午餐构造器
      }
  }
  ```

**3、Constructor.newInstance**

* 本方法和Class类的`newInstance`方法很像，但是比它强大很多。 **java.lang.relect.Constructor类里也有一个newInstance方法可以创建对象。我们可以通过这个**`newInstance`**方法调用有参数（不再必须是无参）的和私有的构造函数（不再必须是public）**。

  ![image-20210911112745245](https://gitee.com/jobim/blogimage/raw/master/img/20210911112745.png)



4、Clone

* 无论何时我们调用一个对象的`clone`方法，JVM就会创建一个新的对象，将前面的对象的内容全部拷贝进去，用`clone`方法创建对象`并不会调用任何构造函数`。 要使用clone方法，我们**必须先实现Cloneable接口**并复写Object的clone方法（因为Object的这个方法是protected的，你若不复写，外部也调用不了呀）。

  ```java
  public class Person implements Cloneable {
  	...
  	// 访问权限写为public，并且返回值写为person
      @Override
      public Person clone() throws CloneNotSupportedException {
          return (Person) super.clone();
      }
      ...
  }
  
  public class Main {
  
      public static void main(String[] args) throws Exception {
          Person person = new Person("fsx", 18);
          Object clone = person.clone(); // 通过clone()创建对象，
          System.out.println(person == clone); //false
      }
  
  }
  ```

**5、反序列化**

* 序列化：将堆内存中的java 对象通过某种方式，存储到磁盘上或者传输给其他网络节点，也就是java对象转成二进制。

* 反序列化：与序列化相反，再将序列化之后的二进制串再转成数据结构或对象。

* 当我们序列化和反序列化一个对象，JVM会给我们创建一个单独的对象，在反序列化时，JVM创建对象并**不会调用任何构造函数**。为了反序列化一个对象，我们需要让我们的类实现`Serializable`接口。

  ```java
  ObjectInputStream in = new ObjectInputStream (new FileInputStream("data.obj")); //读取二进制文件
  Student stu3 = (Student)in.readObject(); //反序列化创建对象
  ```

  

  

## 对象创建的步骤

**1、判断对象对应的类是否加载、链接、初始化**

1. 虚拟机遇到一条new指令，首先去检查这个指令的参数能否在**运行时常量池**中定位到一个类的符号引用（就这个类的路径+名字），并且检查这个符号引用代表的类是否已经被加载，解析和初始化。（即判断类元信息是否存在）。
2. 如果该类没有加载，那么在双亲委派模式下，使用当前类加载器以ClassLoader + 包名 + 类名为key进行查找对应的.class文件，如果没有找到文件，则抛出ClassNotFoundException异常，如果找到，则进行类加载，并生成对应的Class对象。

**2、为对象分配内存**

1. 首先计算对象占用空间的大小，接着在堆中划分一块内存给新对象。如果实例成员变量是引用变量，仅分配引用变量空间即可，即4个字节大小
2. 如果内存规整：采用**指针碰撞**分配内存
   - 如果内存是规整的，那么虚拟机将采用的是指针碰撞法（Bump The Point）来为对象分配内存。
   
     > 意思是所有用过的内存在一边，空闲的内存放另外一边，中间放着一个指针作为分界点的指示器，分配内存就仅仅是把指针往空闲内存那边挪动一段与对象大小相等的距离罢了。
     >
     > 如果垃圾收集器选择的是Serial ，ParNew这种基于压缩算法的，虚拟机采用这种分配方式。一般使用带Compact（整理）过程的收集器时，使用指针碰撞。
     >
     > 标记压缩（整理）算法会整理内存碎片，堆内存一存对象，另一边为空闲区域
3. 如果内存不规整，**空闲列表方式**
   - 如果内存不是规整的，已使用的内存和未使用的内存相互交错，那么虚拟机将采用的是空闲列表来为对象分配内存。
   
     > 意思是虚拟机维护了一个列表，记录上哪些内存块是可用的，再分配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的内容。这种分配方式成为了 “空闲列表（Free List）”
     >
     > 选择哪种分配方式由Java堆是否规整所决定，而Java堆是否规整又由所采用的垃圾收集器是否带有压缩整理功能决定
     >
     > 标记清除算法清理过后的堆内存，就会存在很多内存碎片。

**3、处理并发问题**

* **象创建在虚拟机执行的过程中是非常频繁的行为，仅仅修改一个指针所指向的位置，在并发情况下不是线程安全的。因此也有两种解决方案**
  * **使用CAS并配上失败重试的方式保证更新操作的原子性**。
  * **（TLAB）给每一个线程在Java堆中预先分配线程私有分配缓冲区，哪个线程需要分配内存，只要在线程私有分配缓冲区中分配即可以**



**4、初始化分配到的空间**

- 内存分配完成后，虚拟机需要将分配到的**内存空间都初始化为零值**（不包括对象头），这一步操作保证了对象的实例字段在Java代码中可以不赋初始值就直接使用，程序能访问到这些字段的数据类型所对应的零值。

**5、设置对象的对象头**

* 将对象的所属类（即类的元数据信息）、对象的HashCode和对象的GC信息、锁信息等数据存储在对象的对象头中。这个过程的具体设置方式取决于JVM实现。

**6、执行init方法进行初始化**

1. 在Java程序的视角看来，初始化才正式开始。**初始化成员变量，执行实例化代码块，调用类的构造方法**，并把堆内对象的首地址赋值给引用变量
2. 因此一般来说（由字节码中跟随invokespecial指令所决定），new指令之后会接着就是执行init方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完成创建出来。





# 反射相关



**反射的机制：**

* 对于任意一个类，都能够知道这个类的所有属性和方法；
* 对于任意一个对象，都能够调用它的任意方法和属性；



**获取Class实例的几种方式：**

1. 已知具体的类，通过类的class属性获取，该方法最为安全可靠，程序性能最高 

		实例：Class clazz = String.class; 
2. 已知某个类的实例，调用该实例的getclass()方法获取Class对象 

		实例：Class clazz=person.getclass(); 
3. 已知一个类的全类名，且该类在类路径下，可通过Class类的静态方法forName()获取，可能抛出 ClassNotFoundException（比较常用） 

		实例：Class clazz = Class.forName(String classPath) 
4. 通过类加载器（了解）

		 ClassLoader cl = this.getclass().getClassLoader(); 
		 Class clazz = cl.loadClass("类的全类名");



# 异常相关







# 泛型相关



# Java序列化

<font color='blue'>**序列化机制：**</font>

* **允许将实现序列化的Java对象转换位字节序列，这些字节序列可以保存在磁盘上，或通过网络传输，以达到以后恢复成原来的对象。序列化机制使得对象可以脱离程序的运行而独立存在**



## 序列化的实现方式

* 如果需要将某个对象保存到磁盘上或者通过网络传输，那么这个类应该实现**Serializable**接口或者**Externalizable**接口之一。

### 实现Serializable接口

**序列化步骤：**

- **步骤一：创建一个ObjectOutputStream输出流；**

- **步骤二：调用ObjectOutputStream对象的writeObject输出可序列化对象。**

  ```java
  class Person implements Serializable {
      private String name;
      private int age;
  
      //我不提供无参构造器
      public Person(String name, int age) {
          System.out.println("有参构造方法");
          this.name = name;
          this.age = age;
      }
  
      @Override
      public String toString() {
          return "Person{" +
                  "name='" + name + '\'' +
                  ", age=" + age +
                  '}';
      }
  }
  
  public class WriteObject {
  
      public static void main(String[] args) {
          try (//创建一个ObjectOutputStream输出流
               ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("object.txt"))) {
              //将对象序列化到文件s
              Person person = new Person("9龙", 23);
              oos.writeObject(person);
          } catch (Exception e) {
              e.printStackTrace();
          }
      }
  
  }
  ```

**反序列化步骤：**

- **步骤一：创建一个ObjectInputStream输入流；**

- **步骤二：调用ObjectInputStream对象的readObject()得到序列化的对象**

  ```java
  public class ReadObject {
      public static void main(String[] args) {
          try (//创建一个ObjectInputStream输入流
               ObjectInputStream ois = new ObjectInputStream(new FileInputStream("object.txt"))) {
              Person brady = (Person) ois.readObject();
              System.out.println(brady);
          } catch (Exception e) {
              e.printStackTrace();
          }
      }
  }
  //输出结果
  //Person{name='9龙', age=23}
  ```

> **输出告诉我们，反序列化并不会调用构造方法。反序列的对象是由JVM自己生成的对象，不通过构造方法生成。**



<font color='blue'>**成员是引用的序列化：**</font>

* **如果一个可序列化的类的成员不是基本类型，也不是String类型，那这个引用类型也必须是可序列化的；否则，会导致此类不能序列化。**



<font color='blue'>**可选的自定义序列化：**</font>

* 有些时候，我们有这样的需求，某些属性不需要序列化。**使用`transient`或者`static`关键字选择不需要序列化的字段**



### 实现Externalizable接口

* 通过实现Externalizable接口，必须实现writeExternal、readExternal方法。

```java
public class ExPerson implements Externalizable {

    private String name;
    private int age;

    //注意，必须加上public 无参构造器
    public ExPerson() {
        System.out.println("调用无参构造");
    }

    public ExPerson(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        //将name反转后写入二进制流
        StringBuffer reverse = new StringBuffer(name).reverse();
        System.out.println(reverse.toString());
        out.writeObject(reverse);
        out.writeInt(age);
    }

    @Override
    public void readExternal(ObjectInput in) throws IOException, ClassNotFoundException {
        //将读取的字符串反转后赋值给name实例变量
        this.name = ((StringBuffer) in.readObject()).reverse().toString();
        System.out.println(name);
        this.age = in.readInt();
    }

    public static void main(String[] args) throws IOException, ClassNotFoundException {
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("ExPerson.txt"));
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("ExPerson.txt"));
        oos.writeObject(new ExPerson("brady", 23));
        System.out.println("====序列化完成====");
        ExPerson ep = (ExPerson) ois.readObject();
        System.out.println(ep);
    }
}
//输出结果
//ydarb
//====序列化完成====
//调用无参构造
//brady
//com.zb.one.ExPerson@6d03e736
```

**注意：Externalizable接口不同于Serializable接口，实现此接口必须实现接口中的两个方法实现自定义序列化，这是强制性的；特别之处是必须提供pulic的无参构造器，因为在反序列化的时候需要反射创建对象。**





<font color='blue' size='4'>**序列化版本号serialVersionUID的作用？**</font>

**序列化版本号可自由指定，如果不指定，JVM会根据类信息自己计算一个版本号，这样随着class的升级（属性的修改），就无法正确反序列化；不指定版本号另一个明显隐患是，不利于jvm间的移植，可能class文件没有更改，但不同jvm可能计算的规则不一样，这样也会导致无法反序列化。**





# 网络编程

## socket连接建立步骤

**客户端Socket的工作过程：**

1. 创建Socket：根据指定服务端的P地址或端口号构造Sσcket类对象。若服务器端响应，则建立客户端到服务器的通信线路。若连接失败，会出现异常。
2. 打开连接到 Socket的输入出流：使用getInputstream()方法获得输入流，使用getOutputStream()方法获得输出流，进行数据传输
3. 按照一定的协议对Socket进行读/写操作：通过输入流读取服务器放入线路的信息（但不能读取自己放入线路的信息），通过输出流将信息写入线程
4. 关闭 Socket：断开客户端到服务器的连接，释放线路

**服务器端Socket的工作过程**

1. 调用ServerSocket(int port)：创建一个服务器端套接字，并绑定到指定端口上。用于监听客户端的请求。
2. 调用accept()：监听连接请求，如果客户端请求连接，则接受连接，返回通信套接字对象。
3. 调用该Socket类对象的getOutputStream()和getInputStream()：获取输出流和输入流，开始网络数据的发送和接收。
4. 关闭 ServerSocket和Socket对象：客户端访问结束，关闭通信套接字。

![image-20211022100053412](https://gitee.com/jobim/blogimage/raw/master/img/20211022100053.png)

# JavaEE相关



## Cookie和Session

**Cookie和Session的区别？**

- **作用范围不同**，Cookie 保存在客户端（浏览器），Session 保存在服务器端。
- **存取方式的不同**，Cookie 只能保存 ASCII，Session 可以存任意数据类型，一般情况下我们可以在 Session 中保持一些常用变量信息，比如说 UserId 等。
- **有效期不同**，Cookie 可设置为长时间保持，比如我们经常使用的默认登录功能，Session 一般失效时间较短，客户端关闭或者 Session 超时都会失效。
- 隐私策略不同，Cookie 存储在客户端，比较容易遭到不法获取，早期有人将用户的登录名和密码存储在 Cookie 中导致信息被窃取；Session 存储在服务端，安全性相对 Cookie 要好一些。
- **存储大小不同**， 单个 Cookie 保存的数据不能超过 4K，Session 可存储数据远高于 Cookie。



**为什么需要 Cookie 和 Session？**

* 如今的网络通信方式采用的是http协议，而**http协议是无状态的协议**，因此一旦数据交换结束之后，客户端和服务器端的连接就断了。意味着你再发一条请求时，服务器是无法记住你的。因此必须引入一些机制来告诉服务端，本次操作用户是否登录，是哪个用户在执行的操作。



**session具体实现？**

1. 用户第一次请求服务器的时候，服务器根据用户提交的相关信息，创建对应的 Session 。**服务器在第一次获取 session 即调用 request.getSession() 的时候，服务器会创建一个 session 对象，并且存入服务器的 session 集合中以 sessionId 为标识键，也就是说根据 sessionId 即可取到对应 session 的引用。**

2. **同时也会创建一个`key为 JSESSIONID` 的 cookie 并且返回给浏览器，该cookie的`value为 sessionId`**

   ```java
   //当第一次调用request.getSession()时浏览器默认的操作
   Cookie cookie = new Cookie("JSESSIONID", sessionID);
   response.addCookie(cookie);
   ```

3. **客户端收到这个cookie后将其放到浏览器的缓存中**

4. **当用户第二次访问服务器的时候，请求会自动判断此域名下是否存在 Cookie 信息，如果存在自动将 Cookie 信息也发送给服务端，服务端会从 Cookie 中获取 SessionID，再根据 SessionID 查找对应的 Session 信息，获取用户的登陆信息。**

![image-20210930111316081](https://gitee.com/jobim/blogimage/raw/master/img/20210930111316.png)





**创建sessionId值的方法？**

* tomcat的session的id值生成的机制是一个随机数加时间加上jvm的id值，jvm的id值会根据服务器的硬件信息计算得来，因此不同jvm的id值都是唯一的







**1、equals 和 == 的区别？**

最直接，我们点开equals的源码

java

```java
    public boolean equals(Object obj) {
        return (this == obj);
    }
```

我们可以发现equals也是用的`==` 来比较的，但是为什么还要说它们之间的区别呢？因为equals的方法可以根据我们的需要重写。`==`如果比较的是两个值类型的话，那么就是比较它们之间是否相等，如果是引用类型的话，那么就是比较它们之间的地址了。

**2、为什么重写equals一定要重写hashcode？**

默认的hashcode方法是根据对象的内存地址经哈希算法得到的，如果不重写的话，那么在两个相同的对象在使用equals方法的时候就有可能不同。这在我们的map中的话，如果以对象为key的话，就会导致我们逻辑上的key相同却有着不同的值！

**3、Integer与int的==比较是怎么样的？**

首先看一下下面的比较

java

```java
    public static void main(String[] args) {
        Integer a = 3;
        int b = 3;
        Integer c = Integer.valueOf(3);
        Integer d = new Integer(3);
        System.out.println(a == b);   //输出true
        System.out.println(b == c);   //输出true
        System.out.println(b == d);   //输出true
        System.out.println(a == c);   //输出true
        System.out.println(c == d);   //输出false
    }
```

我们的Integer在int比较的时候，会自动拆箱，再做值比较，所以返回true；

Integer之间的比较的话，除了new Integer()之外，其他比较都是同一段地址，而new的新对象则不是，所以返回的false。

**4、接口和抽象类的区别？**

- 接口中的所有方法都是抽象的，而抽象类可以有抽象的和非抽象的
- 类可以实现很多个接口，但只能继承一个抽象类
- 类可以不实现抽象类和接口声明的所有方法，但是该类必须声明成抽象的
- 接口的成员方法默认是public，而抽象类的成员可以是private，protected，public
- JDK1.8开始，接口中可以包含default方法（可以进行实现），但是抽象类没有。

**5、Java的异常处理机制，Error和Exception的区别？**

二者都有共同的父类——Throwable！

Error：表示程序发生错误，是程序无法处理的，不可恢复的，如OutOfMemoryError

Exception: 表示程序可处理的异常，又分为CheckedException（受检异常）、UncheckedException(非受检异常)，受检异常发生在编译期，必须要使用try...catch 或者 throws捕获或者抛出异常，否则编译不通过（如IOException之类，多线程之类的）；非受检异常发生在运行期，具有不确定性，主要由程序的逻辑问题引起的，在程序设计的时候要认真考虑，尽量处理异常。（如NullPointException 参数值为null（空指针），IndexOutOfBoundsException 下标参数值越界）

**6、++和--操作是否为原子操作，为什么？**

不是原子性操作。原子性的意思是操作不可分割，但是我们的++和--确实可以分为三个步骤（读写改）

1. 从栈读取我们的值
2. 进行加1的操作
3. 将我们的值压回栈

所以再多线程情况下，就会导致我们的自增或者自减不准确！

**7、面向对象的三大特性是什么？请简单介绍一下！**

封装、继承、多态

**封装：** **所谓封装，也就是把客观事物封装成抽象的类，并且类可以把自己的数据和方法只让可信的类或者对象操作，对不可信的进行信息隐藏。**封装是面向对象的特征之一，是对象和类概念的主要特性。 简单的说，一个类就是一个封装了数据以及操作这些数据的代码的逻辑实体。在一个对象内部，某些代码或某些数据可以是私有的，不能被外界访问。通过这种方式，对象对内部数据提供了不同级别的保护，以防止程序中无关的部分意外的改变或错误的使用了对象的私有部分。

**继承：** **所谓继承是指可以让某个类型的对象获得另一个类型的对象的属性的方法。**它支持按级分类的概念。继承是指这样一种能力：它可以使用现有类的所有功能，并在无需重新编写原来的类的情况下对这些功能进行扩展。 通过继承创建的新类称为“子类”或“派生类”，被继承的类称为“基类”、“父类”或“超类”。继承的过程，就是从一般到特殊的过程。要实现继承，可以通过“继承”（Inheritance）和“组合”（Composition）来实现。继承概念的实现方式有二类：实现继承与接口继承。实现继承是指直接使用基类的属性和方法而无需额外编码的能力；接口继承是指仅使用属性和方法的名称、但是子类必须提供实现的能力；

**多态：** **所谓多态就是指一个类实例的相同方法在不同情形有不同表现形式。**多态机制使具有不同内部结构的对象可以共享相同的外部接口。这意味着，虽然针对不同对象的具体操作不同，但通过一个公共的类，它们（那些操作）可以通过相同的方式予以调用。

**8、Java中是如何具体实现多态的？**

（待补充）

**9、面向对象和面向过程的区别？**

面向对象方法中，把数据和数据操作放在一起，组成对象；对同类的对象抽 象出其共性组成类；类通过简单的接口与外界发生联系，对象和对象之间通过消 息进行通信。而面向过程方法是以过程为中心的开发方法，它自顶向下顺序进行， 程序结构按照功能划分成若干个基本模块，这些模块形成树状结构。

（过程）优点：性能比面向对象高，因为类调用时需要实例化，开销比较大，比较消耗源;比如嵌入式开发、Linux/Unix等一般采用面向过程开发，性能是最重要的因素。缺点：没有面向对象易维护、易复用、易扩展。

（对象）优点：易维护、易复用、易扩展，由于面向对象有封装、继承、多态性的特性，可以设计出低耦合的系统。缺点：性能比面向过程低。

**10、String/StringBuffer/StringBuilder的区别？**

**String：**不可变字符序列

**StringBuffer：**可变字符序列、效率低、线程安全（使用Synchronized修饰）

**StringBuilder：**可变字符序列、效率高、线程不安全

字符串直接相加本质也是转换成StringBuilder调用append，但是因为会产生大量的StringBuilder对象所以不如直接new一个StringBuilder来用效率高！

**11、什么是面向函数式编程？**

**12、谈谈static，final关键字？**

**final**

- 当用final修饰一个类时，表明这个类不能被继承。也就是说，如果一个类你永远不会让他被继承，就可以用final进行修饰。final类中的成员变量可以根据需要设为final，但是要注意final类中的所有成员方法都会被隐式地指定为final方法。
- 当final修饰一个变量的时，如果是基本数据类型的变量，则其数值一旦在初始化之后便不能更改；如果是引用类型的变量，则在对其初始化之后便不能再让其指向另一个对象。

**static**

- 被static修饰的变量属于类变量，可以通过**类名.变量名**直接引用，而不需要new出一个类
- 被static修饰的方法属于类方法，可以通过**类名.方法名**直接引用，而不需要new出一个类

被static修饰的变量、被static修饰的方法统一属于类的静态资源，是类实例之间共享的，换言之，一处变、处处变。

**引出问题**

这里引出一个问题，被final修饰的对象，变量存放在哪里，被static修饰的对象，变量存放在哪里，同时被两个关键字修饰存放在哪里？

这里给上一个[参考答案](https://www.li5jun.com/article/147.html)

**13、谈谈volatile、synchronized关键字？**

谈到这两个关键字需要了解到JMM和内存模型，可以看看我的另一篇博客。（[点击跳转](https://www.jianshu.com/p/c70ed52cb9b8)）

**14、谈谈深拷贝和浅拷贝？**

**浅拷贝：** 只复制当前对象的基本数据类型及引用变量，没有复制引用变量指向的实际对象。修改克隆对象可能影响原对象，不安全。

**深拷贝：** 完全拷贝基本数据类型和引用数据类型，安全。

**15、Java线程和系统线程的区别？**

简单总结，系统线程主要实现在三个方面：

- **实现在用户空间下**：操作系统对线程的存在一无所知，操作系统只能看到进程，而不能看到线程。所有的线程都是在用户空间实现。在操作系统看来，每一个进程只有一个线程。
- **实现在操作系统内核中：**内核线程就是直接由操作系统内核（Kernel）支持的线程，这种线程由内核来完成线程切换，内核通过操纵调度器（Scheduler）对线程进行调度，并负责将线程的任务映射到各个处理器上。通俗的讲就是，程序员直接使用操作系统中已经实现的线程，而线程的创建、销毁、调度和维护，都是靠操作系统（准确的说是内核）来实现，程序员只需要使用系统调用，而不需要自己设计线程的调度算法和线程对CPU资源的抢占使用。

Java线程在JDK1.2之前，是基于称为“绿色线程”（Green Threads）的用户线程实现的，而在JDK1.2中，线程模型替换为基于操作系统原生线程模型来实现。

[参考资料](https://www.jianshu.com/p/7c980955627e)

**16、进程和线程的区别？**

进程和线程的主要区别在于它们是不同的操作系统资源管理方式。

- 进程有独立的地址空间，一个进程崩溃后，在保护模式下不会对其他进程产生影响。
- 线程只是一个进程中的不同执行路径，有自己的堆栈和局部变量，但线程之间没有单独的地址空间。

一个线程死亡就等于整个进程死亡，所以多进程的程序要比多线程的程序健壮。但是，在进程切换时，耗费资源较大，效率较低。因此，对于一些要求同时进行并且又要共享某些变量的并发操作，只能用多线程，而不能用多进程。

**17、什么是同步与异步，阻塞与非阻塞？**

同步和异步是通信机制，阻塞和非阻塞是调用状态。

以IO为例：

同步 IO 是用户线程发起 IO 请求后需要等待或轮询内核 IO 操作完成后才能继续执行。异步 IO 是用户线程发起 IO 请求后可以继续执行，当内核 IO 操作完成后会通知用户线程，或调用用户线程注册的回调函数。

阻塞 IO 是 IO 操作需要彻底完成后才能返回用户空间 。非阻塞 IO 是 IO 操作调用后立即返回一个状态值，无需等 IO 操作彻底完成。

通俗概念可以参考这一篇理解[简单理解什么是同步阻塞/同步非阻塞，异步阻塞/异步非阻塞](https://blog.csdn.net/qq_36963372/article/details/83353017)

**18、各大排序的算法的复杂度和稳定性？**
[![img](https://img2020.cnblogs.com/blog/1993240/202008/1993240-20200818155320704-1152309296.png)](https://img2020.cnblogs.com/blog/1993240/202008/1993240-20200818155320704-1152309296.png)

**19、Java的基本数据类型**

JVM 没有 boolean 赋值的专用字节码指令，`boolean f = false` 就是使用 ICONST_0 即常数 0 赋值。单个 boolean 变量用 int 代替，boolean 数组会编码成 byte 数组。

| 数据类型 | 内存大小                               | 默认值   | 取值范围                    |
| -------- | -------------------------------------- | -------- | --------------------------- |
| byte     | 1 B                                    | (byte)0  | -128 ~ 127                  |
| short    | 2 B                                    | (short)0 | -215 ~ 215-1                |
| int      | 4B                                     | 0        | -231 ~ 231-1                |
| long     | 8B                                     | 0L       | -263 ~ 263-1                |
| float    | 4B                                     | 0.0F     | ±3.4E+38（有效位数 6~7 位） |
| double   | 8B                                     | 0.0D     | ±1.7E+308（有效位数 15 位） |
| char     | 英文 1B，中文 UTF-8 占 3B，GBK 占 2B。 | '\u0000' | '\u0000' ~ '\uFFFF'         |
| boolean  | 单个变量 4B / 数组 1B                  | false    | true、false                 |

**20、自动装箱/拆箱是什么？**

每个基本数据类型都对应一个包装类，除了 int 和 char 对应 Integer 和 Character 外，其余基本数据类型的包装类都是首字母大写即可。

**自动装箱：** 将基本数据类型包装为一个包装类对象，例如向一个泛型为 Integer 的集合添加 int 元素。

**自动拆箱：** 将一个包装类对象转换为一个基本数据类型，例如将一个包装类对象赋值给一个基本数据类型的变量。

比较两个包装类数值要用 `equals` ，而不能用 `==` 。



