# 一篇文章掌握整个JVM，JVM超详细解析

![一篇文章掌握整个JVM，JVM超详细解析！！！](/Users/mark/typora_workspace/NoteBook/images/9576988ae8094a36a77674218e882dbd~tplv-k3u1fbpfcp-zoom-crop-mark:1304:1304:1304:734.image)

@[TOC](https://link.juejin.cn/?target=JVM)

`不懂JVM看完这一篇文章你就会非常懂了，文章很长，非常详细！！！` ![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/fdfb95d0cbcc4c7abafde3e9ddd53996~tplv-k3u1fbpfcp-watermark.image)

## 先想想一些问题

### 1 我们开发人员编写的Java代码是怎么让电脑认识的

首先先了解电脑是二进制的系统，他只认识 01010101

比如我们经常要编写 HelloWord.java 电脑是怎么认识运行的 HelloWord.java是我们程序员编写的，我们人可以认识，但是电脑不认识 

**Java文件编译的过程** 因此就需要编译：

1. 程序员编写的.java文件
2. 由javac编译成字节码文件.class：（为什么编译成class文件，因为JVM只认识.class文件）
3. 在由JVM编译成电脑认识的文件 （对于电脑系统来说 文件代表一切）

（这是一个大概的观念 抽象画的概念） ![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/d285e01e3324408bb38867d322c395b9~tplv-k3u1fbpfcp-watermark.image)

### 2 为什么说java是跨平台语言

这个夸平台是中间语言（JVM）实现的夸平台 java有JVM从软件层面屏蔽了底层硬件、指令层面的细节让他兼容各种系统

难道 C 和 C++ 不能夸平台吗 其实也可以 C和C++需要在编译器层面去兼容不同操作系统的不同层面，写过C和C++的就知道不同操作系统的有些代码是不一样 

### 3 Jdk和Jre和JVM的区别

看Java官方的图片，Jdk中包括了Jre，Jre中包括了JVM

Jvm在倒数第二层 由他可以在（最后一层的）各种平台上运行

Jre大部分都是 C 和 C++ 语言编写的，他是我们在编译java时所需要的基础的类库

Jdk还包括了一些Jre之外的东西 ，就是这些东西帮我们编译Java代码的， 还有就是监控Jvm的一些工具

![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/b76e8e384b79444aa77220a59b670749~tplv-k3u1fbpfcp-watermark.image)

### 4 为什么要学习JVM

**为什么要学习Jvm，学习Jvm可以干什么**

首先先想：为什么Java可以霸占企业级开发那么多年 因为：内存管理

```
我们在java开发中何时考虑过内存管理 
不像c和c++还要考虑什么时候释放资源
我们java只需要考虑业务实现就行了
复制代码
```



**那就有些人可能又会要说了，Jvm都做完了这些操作，为什么我们还要学习，学习个屁啊**

```
假如：内存出现问题了，出现了内存溢出 ，内存泄漏问题怎么办
复制代码
```

这就好像一个人一样，我一般情况吃什么从来不用考虑进入了身体那一个部位，可是总有一天，假如吃了不该吃的也是要进医院的



## 深入学习JVM

**注释**：JVM就是Java虚拟机，Java虚拟机就是JVM

### 1 JVM运行时数据区

什么是运行时数据区（就是我们java运行时的东西是放在那里的） ![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/d2fe2c617e4a49aaae3283fcd2e6c5ad~tplv-k3u1fbpfcp-watermark.image)

### 2 解析JVM运行时数据区

#### 2.1 方法区（Method Area）

1. 方法区是所有线程共享的内存区域，它用于存储已被Java虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。
2. 它有个别命叫Non-Heap（非堆）。当方法区无法满足内存分配需求时，抛出OutOfMemoryError异常。

#### 2.2 Java堆（Java Heap）

1. java堆是java虚拟机所管理的内存中最大的一块，是被所有线程共享的一块内存区域，在虚拟机启动时创建。此内存区域的唯一目的就是存放对象实例。
2. 在Java虚拟机规范中的描述是：所有的对象实例以及数组都要在堆上分配。
3. java堆是垃圾收集器管理的主要区域，因此也被成为“GC堆”。
4. 从内存回收角度来看java堆可分为：新生代和老生代。
5. 从内存分配的角度看，线程共享的Java堆中可能划分出多个线程私有的分配缓冲区。
6. 无论怎么划分，都与存放内容无关，无论哪个区域，存储的都是对象实例，进一步的划分都是为了更好的回收内存，或者更快的分配内存。
7. 根据Java虚拟机规范的规定，java堆可以处于物理上不连续的内存空间中。当前主流的虚拟机都是可扩展的（通过 -Xmx 和 -Xms 控制）。如果堆中没有内存可以完成实例分配，并且堆也无法再扩展时，将会抛出OutOfMemoryError异常。

#### 2.3 程序计数器（Program Counter Register）

1. 程序计数器是一块较小的内存空间，它可以看作是：保存当前线程所正在执行的字节码指令的地址(行号)
2. 由于Java虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的，一个处理器都只会执行一条线程中的指令。因此，为了线程切换后能恢复到正确的执行位置，每条线程都有一个独立的程序计数器，各个线程之间计数器互不影响，独立存储。称之为“线程私有”的内存。程序计数器内存区域是虚拟机中唯一没有规定OutOfMemoryError情况的区域。

总结：也可以把它叫做线程计数器

**例子**：在java中最小的执行单位是线程，线程是要执行指令的，执行的指令最终操作的就是我们的电脑，就是 CPU。在CPU上面去运行，有个非常不稳定的因素，叫做调度策略，这个调度策略是时基于时间片的，也就是当前的这一纳秒是分配给那个指令的。

**假如**：线程A在看直播 ![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/48c7d7792bf643879a8361daee0e427b~tplv-k3u1fbpfcp-watermark.image)突然，线程B来了一个视频电话，就会抢夺线程A的时间片，就会打断了线程A，线程A就会挂起 ![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/5b2a79aedfa5474db52c46c6e48c9358~tplv-k3u1fbpfcp-watermark.image)然后，视频电话结束，这时线程A究竟该干什么？ （线程是最小的执行单位，他不具备记忆功能，他只负责去干，那这个记忆就由：**程序计数器来记录**）

![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/7ef9a52be7904fee824f06e5a553c3d4~tplv-k3u1fbpfcp-watermark.image)

#### 2.4 Java虚拟机栈（Java Virtual Machine Stacks）

1. java虚拟机是线程私有的，它的生命周期和线程相同。
2. 虚拟机栈描述的是Java方法执行的内存模型：每个方法在执行的同时都会创建一个栈帧（Stack Frame）用于存储局部变量表、操作数栈、动态链接、方法出口等信息。

**解释**：每虚拟机栈中是有单位的，单位就是**栈帧**，一个方法一个**栈帧**。一个**栈帧**中他又要存储，局部变量，操作数栈，动态链接，出口等。

![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/afb5cde14a674960b54640e5d13119b5~tplv-k3u1fbpfcp-watermark.image) **解析栈帧：**

1. 局部变量表：是用来存储我们临时8个基本数据类型、对象引用地址、returnAddress类型。（returnAddress中保存的是return后要执行的字节码的指令地址。）
2. 操作数栈：操作数栈就是用来操作的，例如代码中有个 i = 6*6，他在一开始的时候就会进行操作，读取我们的代码，进行计算后再放入局部变量表中去
3. 动态链接：假如我方法中，有个 service.add()方法，要链接到别的方法中去，这就是动态链接，存储链接的地方。
4. 出口：出口是什呢，出口正常的话就是return 不正常的话就是抛出异常落



**思考：**. **一个方法调用另一个方法，会创建很多栈帧吗？** 答：会创建。如果一个栈中有动态链接调用别的方法，就会去创建新的栈帧，栈中是由顺序的，一个栈帧调用另一个栈帧，另一个栈帧就会排在调用者下面

**栈指向堆是什么意思？** 栈指向堆是什么意思，就是栈中要使用成员变量怎么办，栈中不会存储成员变量，只会存储一个应用地址，堆中的数据等下讲

**递归的调用自己会创建很多栈帧吗？** 递归的话也会创建多个栈帧，就是一直排下去

#### 2.5 本地方法栈（Native Method Stack）

1. 本地方法栈很好理解，他很栈很像，只不过方法上带了 native 关键字的栈字
2. 它是虚拟机栈为虚拟机执行Java方法（也就是字节码）的服务
3. native关键字的方法是看不到的，必须要去oracle官网去下载才可以看的到，而且native关键字修饰的大部分源码都是C和C++的代码。
4. 同理可得，本地方法栈中就是C和C++的代码



### 3 Java内存结构

![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/1e7e64860ec5405fa051ead59bc719bd~tplv-k3u1fbpfcp-watermark.image) 上面已经讲了运行时数据区，这里就差几个小组件了

#### 3.1 直接内存（Direct Memory）

1. 直接内存不是虚拟机运行时数据区的一部分，也不是java虚拟机规范中定义的内存区域。但是既然是内存，肯定还是受本机总内存（包括RAM以及SWAP区或者分页文件）大小以及处理器寻址空间的限制。
2. 在JDK1.4 中新加入了NIO(New Input/Output)类，引入了一种基于通道(Channel)与缓冲区（Buffer）的I/O 方式，它可以使用native 函数库直接分配堆外内存，然后通脱一个存储在Java堆中的DirectByteBuffer 对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在Java堆和Native（本地）堆中来回复制数据。

**直接内存与堆内存的区别：** 直接内存申请空间耗费很高的性能，堆内存申请空间耗费比较低 直接内存的IO读写的性能要优于堆内存，在多次读写操作的情况相差非常明显

**代码示例：（报错修改time 值）**

```java
package com.lijie;

import java.nio.ByteBuffer;

/**
 * 直接内存 与 堆内存的比较
 */
public class ByteBufferCompare {

    public static void main(String[] args) {
        allocateCompare();   //分配比较
        operateCompare();    //读写比较
    }

    /**
     * 直接内存 和 堆内存的 分配空间比较
     */
    public static void allocateCompare() {
        int time = 10000000;    //操作次数
        long st = System.currentTimeMillis();
        for (int i = 0; i < time; i++) {

            ByteBuffer buffer = ByteBuffer.allocate(2);      //非直接内存分配申请
        }
        long et = System.currentTimeMillis();
        System.out.println("在进行" + time + "次分配操作时，堆内存：分配耗时:" + (et - st) + "ms");
        long st_heap = System.currentTimeMillis();
        for (int i = 0; i < time; i++) {
            ByteBuffer buffer = ByteBuffer.allocateDirect(2); //直接内存分配申请
        }
        long et_direct = System.currentTimeMillis();
        System.out.println("在进行" + time + "次分配操作时，直接内存：分配耗时:" + (et_direct - st_heap) + "ms");
    }

    /**
     * 直接内存 和 堆内存的 读写性能比较
     */
    public static void operateCompare() {
        //如果报错修改这里，把数字改小一点
        int time = 1000000000;
        ByteBuffer buffer = ByteBuffer.allocate(2 * time);
        long st = System.currentTimeMillis();
        for (int i = 0; i < time; i++) {
            buffer.putChar('a');
        }
        buffer.flip();
        for (int i = 0; i < time; i++) {
            buffer.getChar();
        }
        long et = System.currentTimeMillis();
        System.out.println("在进行" + time + "次读写操作时，堆内存：读写耗时：" + (et - st) + "ms");
        ByteBuffer buffer_d = ByteBuffer.allocateDirect(2 * time);
        long st_direct = System.currentTimeMillis();
        for (int i = 0; i < time; i++) {
            buffer_d.putChar('a');
        }
        buffer_d.flip();
        for (int i = 0; i < time; i++) {
            buffer_d.getChar();
        }
        long et_direct = System.currentTimeMillis();
        System.out.println("在进行" + time + "次读写操作时，直接内存：读写耗时:" + (et_direct - st_direct) + "ms");
    }
}
复制代码
```

测试结果：

```
在进行10000000次分配操作时，堆内存：分配耗时:98ms
在进行10000000次分配操作时，直接内存：分配耗时:8895ms
在进行1000000000次读写操作时，堆内存：读写耗时：5666ms
在进行1000000000次读写操作时，直接内存：读写耗时:884ms
复制代码
```

代码来源：「猕猴桃0303」 链接为：[blog.csdn.net/leaf_0303/a…](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fleaf_0303%2Farticle%2Fdetails%2F78961936) 

#### 3.2 JVM字节码执行引擎

虚拟机核心的组件就是执行引擎，它负责执行虚拟机的字节码，一般户先进行编译成机器码后执行。

“虚拟机”是一个相对于“物理机”的概念，虚拟机的字节码是不能直接在物理机上运行的，需要JVM字节码执行引擎编译成机器码后才可在物理机上执行。 

#### 3.3 垃圾收集系统

程序在运行过程中，会产生大量的内存垃圾（一些没有引用指向的内存对象都属于内存垃圾，因为这些对象已经无法访问，程序用不了它们了，对程序而言它们已经死亡），为了确保程序运行时的性能，java虚拟机在程序运行的过程中不断地进行自动的垃圾回收（GC）。

垃圾收集系统是Java的核心，也是不可少的，Java有一套自己进行垃圾清理的机制，开发人员无需手工清理 

### 4 JVM的垃圾回收机制

垃圾回收机制简称GC

GC主要用于Java堆的管理。Java 中的堆是 JVM 所管理的最大的一块内存空间，主要用于存放各种类的实例对象。

#### 4.1 什么是垃圾回收机制

程序在运行过程中，会产生大量的内存垃圾（一些没有引用指向的内存对象都属于内存垃圾，因为这些对象已经无法访问，程序用不了它们了，对程序而言它们已经死亡），为了确保程序运行时的性能，java虚拟机在程序运行的过程中不断地进行自动的垃圾回收（GC）。

GC是不定时去堆内存中清理不可达对象。不可达的对象并不会马上就会直接回收， 垃圾收集器在一个Java程序中的执行是自动的，不能强制执行清楚那个对象，即使程序员能明确地判断出有一块内存已经无用了，是应该回收的，程序员也不能强制垃圾收集器回收该内存块。程序员唯一能做的就是通过调用System.gc 方法来"建议"执行垃圾收集器，但是他是否执行，什么时候执行却都是不可知的。这也是垃圾收集器的最主要的缺点。当然相对于它给程序员带来的巨大方便性而言，这个缺点是瑕不掩瑜的。

手动执行GC:

```
System.gc(); // 手动回收垃圾
复制代码
```



#### 4.2 finalize方法作用

1. finalize()方法是在每次执行GC操作之前时会调用的方法，可以用它做必要的清理工作。
2. 它是在Object类中定义的，因此所有的类都继承了它。子类覆盖finalize()方法以整理系统资源或者执行其他清理工作。finalize()方法是在垃圾收集器删除对象之前对这个对象调用的。

**代码示例**

```java
package com.lijie;

public class Test {
	public static void main(String[] args) {
		Test test = new Test();
		test = null;
		System.gc(); // 手动回收垃圾
	}

	@Override
	protected void finalize() throws Throwable {
		// gc回收垃圾之前调用
		System.out.println("gc回收垃圾之前调用的方法");
	}
}
复制代码
```



#### 4.3 新生代、老年代、永久代(方法区)的区别

1. Java 中的堆是 JVM 所管理的最大的一块内存空间，主要用于存放各种类的实例对象。
2. 在 Java 中，堆被划分成两个不同的区域：新生代 ( Young )、老年代 ( Old )。

**先不要管为什么要分代，后面有例子**

1. 老年代就一个区域。新生代 ( Young ) 又被划分为三个区域：Eden、From Survivor、To Survivor。
2. 这样划分的目的是为了使 JVM 能够更好的管理堆内存中的对象，包括内存的分配以及回收。
3. 默认的，新生代 ( Young ) 与老年代 ( Old ) 的比例的值为 1:2 ( 该值可以通过参数 –XX:NewRatio 来指定 )，即：新生代 ( Young ) = 1/3 的堆空间大小。老年代 ( Old ) = 2/3 的堆空间大小。
4. 其中，新生代 ( Young ) 被细分为 Eden 和 **两个 Survivor 区域**，这两个 Survivor 区域分别被命名为 From Survivor 和 ToSurvivor ，以示区分。
5. 默认的，Edem ： From Survivor ： To Survivor = 8 : 1 : 1 ( 可以通过参数 –XX:SurvivorRatio 来设定 )，即： Eden = 8/10 的新生代空间大小，From Survivor = To Survivor = 1/10 的新生代空间大小。
6. JVM 每次只会使用 Eden 和其中的一块 Survivor 区域来为对象服务，所以无论什么时候，总是有一块 Survivor 区域是空闲着的。
7. 因此，新生代实际可用的内存空间为 9/10 ( 即90% )的新生代空间。
8. 永久代就是JVM的方法区。在这里都是放着一些被虚拟机加载的类信息，静态变量，常量等数据。这个区中的东西比老年代和新生代更不容易回收。

##### 4.3.1 为什么要这样分代：

其实主要原因就是可以根据各个年代的特点进行对象分区存储，更便于回收，采用最适当的收集算法：

1. 新生代中，每次垃圾收集时都发现大批对象死去，只有少量对象存活，便采用了复制算法，只需要付出少量存活对象的复制成本就可以完成收集。
2. 而老年代中因为对象存活率高、没有额外空间对它进行分配担保，就必须采用“标记-清理”或者“标记-整理”算法。

新生代又分为Eden和Survivor （From与To，这里简称一个区）两个区。加上老年代就这三个区。数据会首先分配到Eden区当中（当然也有特殊情况，如果是大对象那么会直接放入到老年代（大对象是指需要大量连续内存空间的java对象）。当Eden没有足够空间的时候就会触发jvm发起一次Minor GC，。如果对象经过一次Minor-GC还存活，并且又能被Survivor空间接受，那么将被移动到Survivor空间当中。并将其年龄设为1，对象在Survivor每熬过一次Minor GC，年龄就加1，当年龄达到一定的程度（默认为15）时，就会被晋升到老年代中了，当然晋升老年代的年龄是可以设置的。

##### 4.3.2 Minor GC、Major GC、Full GC区别及触发条件

1. Minor GC是新生代GC，指的是发生在新生代的垃圾收集动作。由于java对象大都是朝生夕死的，所以Minor GC非常频繁，一般回收速度也比较快。
2. Major GC是老年代GC，指的是发生在老年代的GC，通常执行Major GC会连着Minor GC一起执行。Major GC的速度要比Minor GC慢的多。
3. Full GC是清理整个堆空间，包括年轻代和老年代

**Minor GC 触发条件一般为：**

1. eden区满时，触发MinorGC。即申请一个对象时，发现eden区不够用，则触发一次MinorGC。
2.  新创建的对象大小 > Eden所剩空间

**Major GC和Full GC 触发条件一般为：** Major GC通常是跟full GC是等价的

1. 每次晋升到老年代的对象平均大小>老年代剩余空间
2. MinorGC后存活的对象超过了老年代剩余空间
3. 永久代空间不足
4. 执行System.gc()
5. CMS GC异常
6. 堆内存分配很大的对象



#### 4.4 如何判断对象是否存活

##### 4.4.1 引用计数法

1. 引用计数法就是如果一个对象没有被任何引用指向，则可视之为垃圾。这种方法的缺点就是不能检测到环的存在。
2. 首先需要声明，至少主流的Java虚拟机里面都没有选用引用计数算法来管理内存。
3. 什么是引用计数法：每个对象在创建的时候，就给这个对象绑定一个计数器。每当有一个引用指向该对象时，计数器加一；每当有一个指向它的引用被删除时，计数器减一。这样，当没有引用指向该对象时，计数器为0就代表该对象死亡

**引用计数法的优点：**

- 引用计数算法的实现简单，判定效率也很高，在大部分情况下它都是一个不错的算法，

**引用计数法的缺点：**

- 主流的Java虚拟机里面没有选用引用计数算法来管理内存，其中最主要的原因是它很难解决对象之间相互循环引用的问题。
- 例如：

```java
package com.lijie;

public class Test {
	public Object object = null;
	public static void main(String[] args) {
		Test a = new Test();
		Test b = new Test();
		/**
		 * 循环引用，此时引用计数器法失效
		 */
		a.object = b;
		b.object = a;

		a = null;
		b = null;
	}
}
复制代码
```

**引用计数法的应用场景：**

- 建议不要用

##### 4.4.2 可达性分析法

1. 该种方法是从GC Roots开始向下搜索，搜索所走过的路径为引用链。当一个对象到GC Roots没用任何引用链时，则证明此对象是不可用的，表示可以回收。

![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/2b525da4f2d04c91ba3de5d10163d071~tplv-k3u1fbpfcp-watermark.image)

1. 上图上图中Object1、Object2、Object3、Object4、Object5到GC Roots是可达的，表示它们是有引用的对象，是存活的对象不可以进行回收
2. Object6、Object7、Object8虽然是互相关联的，但是它们到GC Roots是不可达的，所以他们是可以进行回收的对象。

**那些可以作为GC Roots 的对象：**

```
1、虚拟机栈（栈帧中的本地变量表）中引用的对象；
2、方法区中类静态属于引用的对象；
3、方法区中常量引用的对象；
4、本地方法栈中JNI（即一般说的Native方法）引用的对象。
等
复制代码
```

**可达性算法的优点：**

- 解决相互循环引用问题。

**可达性算法的优点：**

- 目前和引用计数法比没得缺点

**可达性算法的应用场景：**

- 这是目前主流的虚拟机都是采用的算法



#### 4.5 垃圾回收机制策略（也称为GC的算法）

##### 4.5.1 引用计数算法（Reference counting）

每个对象在创建的时候，就给这个对象绑定一个计数器。每当有一个引用指向该对象时，计数器加一；每当有一个指向它的引用被删除时，计数器减一。这样，当没有引用指向该对象时，计数器为0就代表该对象死亡，这时就应该对这个对象进行垃圾回收操作。

**引用计数法的优点：**

- 引用计数算法的实现简单，判定效率也很高。

**引用计数法的缺点：**

- 主流的Java虚拟机里面没有选用引用计数算法来管理内存，其中最主要的原因是它很难解决对象之间相互循环引用的问题。
- 例如：

```java
package com.lijie;

public class Test {
	public Object object = null;
	public static void main(String[] args) {
		Test a = new Test();
		Test b = new Test();
		/**
		 * 循环引用，此时引用计数器法失效
		 */
		a.object = b;
		b.object = a;

		a = null;
		b = null;
	}
}
复制代码
```

**引用计数法的应用场景：**

- 建议不要用



##### 4.5.2 标记–清除算法（Mark-Sweep）

为每个对象存储一个标记位，记录对象的状态（活着或是死亡）。 分为两个阶段，一个是标记阶段，这个阶段内，为每个对象更新标记位，检查对象是否死亡；第二个阶段是清除阶段，该阶段对死亡的对象进行清除，执行 GC 操作。

**标记清除算法的优点：**

- 是可以解决循环引用的问题
- 必要时才回收(内存不足时)

**标记清除算法的缺点：**

- 回收时，应用需要挂起，也就是stop the world。
- 标记和清除的效率不高，尤其是要扫描的对象比较多的时候
- 会造成内存碎片(会导致明明有内存空间,但是由于不连续,申请稍微大一些的对象无法做到),

**标记清除算法的应用场景：**

- 该算法一般应用于老年代,因为老年代的对象生命周期比较长。



##### 4.5.3 标记–整理算法

标记清除算法和标记压缩算法非常相同，但是标记压缩算法在标记清除算法之上解决内存碎片化（有些人叫"标记整理算法"为"标记压缩算法"）

标记-整理法是标记-清除法的一个改进版。同样，在标记阶段，该算法也将所有对象标记为存活和死亡两种状态；不同的是，在第二个阶段，该算法并没有直接对死亡的对象进行清理，而是将所有存活的对象整理一下，放到另一处空间，然后把剩下的所有对象全部清除。这样就达到了标记-整理的目的。

**标记–整理算法优点：**

- 解决标记清除算法出现的内存碎片问题，

**标记–整理算法缺点：**

- 压缩阶段，由于移动了可用对象，需要去更新引用。

**标记–整理算法应用场景：**

- 该算法一般应用于老年代,因为老年代的对象生命周期比较长。



##### 4.5.4 复制算法

该算法将内存平均分成两部分，然后每次只使用其中的一部分，当这部分内存满的时候，将内存中所有存活的对象复制到另一个内存中，然后将之前的内存清空，只使用这部分内存，循环下去。

这个算法与标记-整理算法的区别在于，该算法不是在同一个区域复制，而是将所有存活的对象复制到另一个区域内。

**复制算法的优点：**

- 在存活对象不多的情况下，性能高，能解决内存碎片和java垃圾回收算法之-标记清除 中导致的引用更新问题。

**复制算法的缺点：**:

- 会造成一部分的内存浪费。不过可以根据实际情况，将内存块大小比例适当调整；如果存活对象的数量比较大，复制算法的性能会变得很差。

**复制算法的应用场景：**

- 复制算法一般是使用在新生代中，因为新生代中的对象一般都是朝生夕死的，存活对象的数量并不多，这样使用复制算法进行拷贝时效率比较高。
- jvm将Heap（堆）内存划分为新生代与老年代。又将新生代划分为Eden与2块Survivor Space(幸存者区) ，然后在Eden –>Survivor Space 与To Survivor之间实行复制算法。
- 不过jvm在应用复制算法时，并不是把内存按照1:1来划分的，这样太浪费内存空间了。一般的jvm都是8:1。也即是说,Eden区:From区:To区域的比例是始终有90%的空间是可以用来创建对象的,而剩下的10%用来存放回收后存活的对象。



##### 4.5.5 分代算法（主要的算法就是上面四种，这个是附加的）

这种算法，根据对象的存活周期的不同将内存划分成几块，新生代和老年代，这样就可以根据各个年代的特点采用最适当的收集算法。可以用抓重点的思路来理解这个算法。 新生代对象朝生夕死,对象数量多，只要重点扫描这个区域，那么就可以大大提高垃圾收集的效率。另外老年代对象存储久，无需经常扫描老年代，避免扫描导致的开销。

**新生代**

- 在新生代，每次垃圾收集器都发现有大批对象死去，只有少量存活，采用复制算法，只需要付出少量存活对象的复制成本就可以完成收集。

**老年代**

- 而老年代中因为对象存活率高、没有额外空间对它进行分配担保，就必须“标记清除法或者标记整理算法进行回收。



### 5 垃圾收集器

#### 5.1 什么是垃圾收集器？

- 垃圾收集器是垃圾回收算法（引用计数法、标记清楚法、标记整理法、复制算法）的具体实现，不同垃圾收集器、不同版本的JVM所提供的垃圾收集器可能会有很在差别。
- **我这以JDK8为准：**

![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/1da5d2be80b14253b777e3dd8a3ebf19~tplv-k3u1fbpfcp-watermark.image)

**图中展示了7种不同分代的收集器：** Serial、ParNew、Parallel Scavenge、CMS、Serial Old、Parallel Old、G1

**而它们所处区域，则表明其是属于新生代还是老年代的收集器：**

- 新生代收集器：Serial、ParNew、Parallel Scavenge
- 老年代收集器：CMS、Serial Old、Parallel Old
- 整堆收集器：G1

**两个收集器间有连线，表明它们可以搭配使用：**

```
Serial / Serial Old
Serial / CMS
ParNew / Serial Old
ParNew / CMS
Parallel Scavenge / Serial Old
Parallel Scavenge / Parallel Old
G1
复制代码
```

#### 5.2 垃圾回收器详解

| 垃圾回收器        | 工作区域        | 回收算法             | 工作线程 | 用户线程并行 | 描述                                                         |
| ----------------- | --------------- | -------------------- | -------- | ------------ | ------------------------------------------------------------ |
| Serial            | 新生带          | 复制算法             | 单线程   | 否           | Client模式下默认新生代收集器。简单高效                       |
| ParNew            | 新生带          | 复制算法             | 多线程   | 否           | Serial的多线程版本，Server模式下首选， 可搭配CMS的新生代收集器 |
| Parallel Scavenge | 新生带          | 复制算法             | 多线程   | 否           | 目标是达到可控制的吞吐量                                     |
| Serial Old        | 老年带          | 标记-整理            | 单线程   | 否           | Serial老年代版本，给Client模式下的虚拟机使用                 |
| Parallel Old      | 老年带          | 标记-整理            | 多线程   | 否           | Parallel Scavenge老年代版本，吞吐量优先                      |
| CMS               | 老年带          | 标记-清楚            | 多线程   | 是           | 追求最短回收停顿时间                                         |
| G1                | 新生带 + 老年带 | 标记-整理 + 复制算法 | 多线程   | 是           | JDK1.9默认垃圾收集器                                         |

##### 5.2.1 Serial

- Serial 收集器：新生代。发展历史最悠久的收集器。它是一个单线程收集器，它只会使用一个 CPU 或者线程去完成垃圾收集工作，而且在它进行垃圾收集时，必须暂停其他所有的工作线程，直到它收集结束。

**特点：**

1. 新生代收集器，使用复制算法收集新生代垃圾。
2. 单线程的收集器，GC工作时，其它所有线程都将停止工作。
3. 简单高效，适合单 CPU 环境。单线程没有线程交互的开销，因此拥有最高的单线程收集效率。

**使用方式：**

```
//如何设置JVM参数底下会讲解：这里只是列举一部分参数：

设置垃圾收集器："-XX:+UseSerialGC"  --添加该参数来显式的使用改垃圾收集器；
复制代码
```



##### 5.2.2 ParNew

- ParNew 收集器：新生代。Serial 的多线程版本，即同时启动多个线程去进行垃圾收集。

**特点：**

1. 新生代收集器。ParNew垃圾收集器是Serial收集器的多线程版本，采用复制算法。
2. 除了多线程外，其余的行为、特点和Serial收集器一样。
3. 只有它能与 CMS 收集器配合使用。
4. 但在单个CPU环境中，不比Serail收集器好，多线程使用它比较好。

**使用方式：**

```
//如何设置JVM参数底下会讲解：这里只是列举一部分参数：

设置垃圾收集器："-XX:+UseParNewGC"  --强制指定使用ParNew；    
设置垃圾收集器： "-XX:+UseConcMarkSweepGC"  --指定使用CMS后，会默认使用ParNew作为新生代收集器；
设置垃圾收集器参数："-XX:ParallelGCThreads"  --指定垃圾收集的线程数量，ParNew默认开启的收集线程与CPU的数量相同；
复制代码
```



##### 5.2.3 Parallel Scavenge

- Parallel Scavenge 收集器：新生代。和 ParNew 的关注点不一样，该收集器更关注吞吐量，尽快地完成计算任务。

**特点：**

1. 新生代收集器。
2. 采用复制算法。
3. 多线程收集。
4. 与ParNew 不同的是：高吞吐量为目标，（减少垃圾收集时间，让用户代码获得更长的运行时间）

**使用方式：**

```
//如何设置JVM参数底下会讲解：这里只是列举一部分参数：

设置垃圾收集器："-XX:+UseParallelGC"  --添加该参数来显式的使用改垃圾收集器；
设置垃圾收集器参数："-XX:MaxGCPauseMillis"  --控制垃圾回收时最大的停顿时间(单位ms)
设置垃圾收集器参数："-XX:GCTimeRatio"  --控制程序运行的吞吐量大小吞吐量大小=代码执行时间/(代码执行时间+gc回收的时间)
设置垃圾收集器参数："-XX:UseAdaptiveSizePolicy"  --内存调优交给虚拟机管理
复制代码
```



##### 5.2.4 Serial Old

- Serial Old 收集器：Serial 的老年代版本，使用标记 - 整理算法。

**特点：**

1. 老年代收集器， 采用"标记-整理"算法。
2. 单线程收集。

**使用方式：**

```
//如何设置JVM参数底下会讲解：这里只是列举一部分参数：

在JDK1.5及之前，与Parallel Scavenge收集器搭配使用，
在JDK1.6后有Parallel Old收集器可搭配。
现在的作为CMS收集器的后备预案，在并发收集发生Concurrent Mode Failure时使用
复制代码
```



##### 5.2.5 Parallnel old

- Parallnel old 收集器，多线程：Parallel 的老年代版本，使用标记 - 整理算法。

**特点：**

1. 针对老年代。
2. 采用"标记-整理"算法。
3. 多线程收集。
4. 但在单个CPU环境中，不比Serial Old收集器好，多线程使用它比较好。

**使用方式：**

```
//如何设置JVM参数底下会讲解：这里只是列举一部分参数：

 设置垃圾收集器："-XX:+UseParallelOldGC"：指定使用Parallel Old收集器；
复制代码
```



##### 5.2.6 CMS

- CMS 收集器：老年代。是一种以获取最短回收停顿时间为目标的收集器，适用于互联网站或者 B/S 系统的服务端上。

**特点：**

1. 针对老年代，采用标记-清楚法清除垃圾；
2. 基于"标记-清除"算法(不进行压缩操作，产生内存碎片)；
3. 以获取最短回收停顿时间为目标；
4. 并发收集、低停顿；
5. CMS收集器有3个明显的缺点：1.对CPU资源非常敏感、2.无法处理浮动垃圾，可能出现"Concurrent Mode Failure"失败、3.产生大量内存碎片
6. 垃圾收集线程与用户线程（基本上）可以同时工作

**使用方式：**

```
//如何设置JVM参数底下会讲解：这里只是列举一部分参数：

设置垃圾收集器："-XX:+UseConcMarkSweepGC"：指定使用CMS收集器；
复制代码
```



##### 5.2.7 G1

- G1 收集器：分代收集器。当今收集器技术发展最前沿成果之一，是一款面向服务端应用的垃圾收集器。G1可以说是CMS的终极改进版，解决了CMS内存碎片、更多的内存空间登问题。虽然流程与CMS比较相似，但底层的原理已是完全不同。

**特点：**

1. 能充分利用多CPU、多核环境下的硬件优势；
2. 可以并行来缩短(Stop The World)停顿时间；
3. 也可以并发让垃圾收集与用户程序同时进行；
4. 分代收集，收集范围包括新生代和老年代
5. 能独立管理整个GC堆（新生代和老年代），而不需要与其他收集器搭配；
6. 能够采用不同方式处理不同时期的对象；
7. 应用场景可以面向服务端应用，针对具有大内存、多处理器的机器；
8. 采用标记-整理 + 复制算法来回收垃圾

**使用方式：**

```
//如何设置JVM参数底下会讲解：这里只是列举一部分参数：

设置垃圾收集器："-XX:+UseG1GC"：指定使用G1收集器；
设置垃圾收集器参数："-XX:InitiatingHeapOccupancyPercent"：当整个Java堆的占用率达到参数值时，开始并发标记阶段；默认为45；
设置垃圾收集器参数："-XX:MaxGCPauseMillis"：为G1设置暂停时间目标，默认值为200毫秒；
设置垃圾收集器参数："-XX:G1HeapRegionSize"：设置每个Region大小，范围1MB到32MB；目标是在最小Java堆时可以拥有约2048个Region
复制代码
```



### 6 JVM参数配置

#### 6.1 JVM内存参数简述

```
#常用的设置
-Xms：初始堆大小，JVM 启动的时候，给定堆空间大小。 

-Xmx：最大堆大小，JVM 运行过程中，如果初始堆空间不足的时候，最大可以扩展到多少。 

-Xmn：设置堆中年轻代大小。整个堆大小=年轻代大小+年老代大小+持久代大小。 

-XX:NewSize=n 设置年轻代初始化大小大小 

-XX:MaxNewSize=n 设置年轻代最大值

-XX:NewRatio=n 设置年轻代和年老代的比值。如: -XX:NewRatio=3，表示年轻代与年老代比值为 1：3，年轻代占整个年轻代+年老代和的 1/4 

-XX:SurvivorRatio=n 年轻代中 Eden 区与两个 Survivor 区的比值。注意 Survivor 区有两个。8表示两个Survivor :eden=2:8 ,即一个Survivor占年轻代的1/10，默认就为8

-Xss：设置每个线程的堆栈大小。JDK5后每个线程 Java 栈大小为 1M，以前每个线程堆栈大小为 256K。

-XX:ThreadStackSize=n 线程堆栈大小

-XX:PermSize=n 设置持久代初始值	

-XX:MaxPermSize=n 设置持久代大小
 
-XX:MaxTenuringThreshold=n 设置年轻带垃圾对象最大年龄。如果设置为 0 的话，则年轻代对象不经过 Survivor 区，直接进入年老代。

#下面是一些不常用的

-XX:LargePageSizeInBytes=n 设置堆内存的内存页大小

-XX:+UseFastAccessorMethods 优化原始类型的getter方法性能

-XX:+DisableExplicitGC 禁止在运行期显式地调用System.gc()，默认启用	

-XX:+AggressiveOpts 是否启用JVM开发团队最新的调优成果。例如编译优化，偏向锁，并行年老代收集等，jdk6纸之后默认启动

-XX:+UseBiasedLocking 是否启用偏向锁，JDK6默认启用	

-Xnoclassgc 是否禁用垃圾回收

-XX:+UseThreadPriorities 使用本地线程的优先级，默认启用	

等等等......
复制代码
```

#### 6.2 JVM的GC收集器设置

```
-XX:+UseSerialGC:设置串行收集器，年轻带收集器 

 -XX:+UseParNewGC:设置年轻代为并行收集。可与 CMS 收集同时使用。JDK5.0 以上，JVM 会根据系统配置自行设置，所以无需再设置此值。

-XX:+UseParallelGC:设置并行收集器，目标是目标是达到可控制的吞吐量

-XX:+UseParallelOldGC:设置并行年老代收集器，JDK6.0 支持对年老代并行收集。 

-XX:+UseConcMarkSweepGC:设置年老代并发收集器

-XX:+UseG1GC:设置 G1 收集器，JDK1.9默认垃圾收集器
复制代码
```



#### 6.3 JVM参数在哪设置

##### 6.3.1 IDEA在哪里设置JVM参数

1、单个项目的应用 ![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/ad7374316e334e83aea57d1afd57edeb~tplv-k3u1fbpfcp-watermark.image)![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/3833fa45ecf146e2a0a02780bfc1445a~tplv-k3u1fbpfcp-watermark.image)2、全局的配置

1. 找到IDEA安装目录中的bin目录
2. 找到idea.exe.vmoptions文件
3. 打开该文件编辑并保存。

![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/86c65f6e308448a0af68efdfe7785e3a~tplv-k3u1fbpfcp-watermark.image)

##### 6.3.2 Eclipse在哪里设置JVM参数

**1、配置单个项目**

点击绿色图标右边的小箭头 ![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/aa6eb42ff1bd45f9832a92c97668b05d~tplv-k3u1fbpfcp-watermark.image) 在点击：Run Configurations ->VM arguments ![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/22d2ad1e254f4b5289aea8fb78cc4185~tplv-k3u1fbpfcp-watermark.image)

**2、配置全局JVM参数** 修改Eclipse的配置文件，在eclipse安装目录下的：eclipse.ini文件 ![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/ce63a87de8034cd29d05914b56a76fb0~tplv-k3u1fbpfcp-watermark.image)

##### 6.3.3 war(Tomcat)包在哪里设置JVM参数

**war肯定是部署在Tomcat上的，那就是修改Tomcat的JVM参数**

**1、在Windows下就是在文件/bin/catalina.bat，** 增加如下设置：JAVA_OPTS（JAVA_OPTS，就是用来设置 JVM 相关运行参数的变量）

```
set "JAVA_OPTS=-Xms512M -Xmx1024M ...等等等 JVM参数"
复制代码
```

![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/c6bada5c1bd14ff9a011ef7433dd1bdb~tplv-k3u1fbpfcp-watermark.image)

**2、Linux要在tomcat 的bin 下的catalina.sh 文件里添加**

```
注意：位置要在cygwin=false前
JAVA_OPTS="-Xms512M -Xmx1024M ...等等等 JVM参数"
复制代码
```

![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/c9da415a0d464c45b0f1625f37705e70~tplv-k3u1fbpfcp-watermark.image)

##### 6.3.4 Jar包在哪里设置JVM参数

Jar包简单，一般都是SpringBoot项目打成Jar包来运行

```
#运行时java -jar是直接插入JVM命令就好了
java -Xms1024m -Xmx1024m ...等等等 JVM参数 -jar springboot_app.jar & 
复制代码
```



#### 6.4 调优总结

1. 在实际工作中，我们可以直接将初始的堆大小与最大堆大小相等， 这样的好处是可以减少程序运行时垃圾回收次数，从而提高效率。
2. 初始堆值和最大堆内存内存越大，吞吐量就越高， 但是也要根据自己电脑(服务器)的实际内存来比较。
3. 最好使用并行收集器,因为并行收集器速度比串行吞吐量高，速度快。 当然，服务器一定要是多线程的
4. 设置堆内存新生代的比例和老年代的比例最好为1:2或者1:3。 默认的就是1:2
5. 减少GC对老年代的回收。设置生代带垃圾对象最大年龄，进量不要有大量连续内存空间的java对象，因为会直接到老年代，内存不够就会执行GC

**注释**：其实最主要的还是服务器要好，你硬件都跟不上，软件再好都没用 **注释**：老年代GC很慢，新生代没啥事 **注释**：默认的JVM堆大小好像是电脑实际内存的四分之一左右，

```java
package com.lijie;

public class Test {
    public static void main(String[] args) {
        System.out.print("最大内存");
        System.out.println(Runtime.getRuntime().maxMemory() / 1024.0 / 1024 + "M");
    }
}
复制代码
```

我的电脑是8G的运行内存 ![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/0d7e9beadd574d0192db655bb61f5f08~tplv-k3u1fbpfcp-watermark.image) 

### 7 类加载器

#### 7.1 类加载的机制及过程

程序主动使用某个类时，如果该类还未被加载到内存中，则JVM会通过加载、连接、初始化3个步骤来对该类进行初始化。如果没有意外，JVM将会连续完成3个步骤，所以有时也把这个3个步骤统称为类加载或类初始化。

**Jvm执行class文件** ![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/d9f4b0facdb9424c9d0b6daa0eba8d01~tplv-k3u1fbpfcp-watermark.image)

##### 1、加载

- 加载指的是将类的class文件读入到内存，并将这些静态数据转换成方法区中的运行时数据结构，并在堆中生成一个代表这个类的java.lang.Class对象，作为方法区类数据的访问入口，这个过程需要类加载器参与。
- Java类加载器由JVM提供，是所有程序运行的基础，JVM提供的这些类加载器通常被称为系统类加载器。除此之外，开发者可以通过继承ClassLoader基类来创建自己的类加载器。
- 类加载器，可以从不同来源加载类的二进制数据，比如：本地Class文件、Jar包Class文件、网络Class文件等等等。
- 类加载的最终产物就是位于堆中的Class对象（注意不是目标类对象），该对象封装了类在方法区中的数据结构，并且向用户提供了访问方法区数据结构的接口，即Java反射的接口

##### 2、连接过程

- 当类被加载之后，系统为之生成一个对应的Class对象，接着将会进入连接阶段，连接阶段负责把类的二进制数据合并到JRE中（意思就是将java类的二进制代码合并到JVM的运行状态之中）。类连接又可分为如下3个阶段。

1. 验证：确保加载的类信息符合JVM规范，没有安全方面的问题。主要验证是否符合Class文件格式规范，并且是否能被当前的虚拟机加载处理。
2. 准备：正式为类变量（static变量）分配内存并设置类变量初始值的阶段，这些内存都将在方法区中进行分配
3. 解析：虚拟机常量池的符号引用替换为字节引用过程

##### 3、初始化

- 初始化阶段是执行类构造器`<clinit>`() 方法的过程。类构造器`<clinit>`()方法是由编译器自动收藏类中的`所有类变量的赋值动作和静态语句块(static块)中的语句合并产生，代码从上往下执行。`
- 当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化
- 虚拟机会保证一个类的`<clinit>`() 方法在多线程环境中被正确加锁和同步

**总结就是**：初始化是为类的静态变量赋予正确的初始值 

#### 7.2 类加载器的介绍

1. 启动（Bootstrap）类加载器
2. 扩展（Extension）类加载器
3. 系统类加载器
4. 自定义加载器

![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/b1e680fa05f1464e81d2a2a94fb16597~tplv-k3u1fbpfcp-watermark.image)

##### 1 根类加载器（bootstrap class loader）

它用来加载 Java 的核心类，是用原生代码来实现的，并不继承自 java.lang.ClassLoader（负责加载$JAVA_HOME中jre/lib/rt.jar里所有的class，由C++实现，不是ClassLoader子类）。由于引导类加载器涉及到虚拟机本地实现细节，开发者无法直接获取到启动类加载器的引用，所以不允许直接通过引用进行操作。

##### 2 扩展类加载器（extensions class loader）

扩展类加载器是指Sun公司(已被Oracle收购)实现的sun.misc.Launcher$ExtClassLoader类，由Java语言实现的，是Launcher的静态内部类，它负责加载<JAVA_HOME>/lib/ext目录下或者由系统变量-Djava.ext.dir指定位路径中的类库，开发者可以直接使用标准扩展类加载器。

##### 3 系统类加载器（system class loader）

被称为系统（也称为应用）类加载器，它负责在JVM启动时加载来自Java命令的-classpath选项、java.class.path系统属性，或者CLASSPATH换将变量所指定的JAR包和类路径。程序可以通过ClassLoader的静态方法getSystemClassLoader()来获取系统类加载器。如果没有特别指定，则用户自定义的类加载器都以此类加载器作为父加载器。由Java语言实现，父类加载器为ExtClassLoader。（Java虚拟机采用的是双亲委派模式即把请求交由父类处理，它一种任务委派模式，）

类加载器加载Class大致要经过如下8个步骤：

1. 检测此Class是否载入过，即在缓冲区中是否有此Class，如果有直接进入第8步，否则进入第2步。
2. 如果没有父类加载器，则要么Parent是根类加载器，要么本身就是根类加载器，则跳到第4步，如果父类加载器存在，则进入第3步。
3. 请求使用父类加载器去载入目标类，如果载入成功则跳至第8步，否则接着执行第5步。
4. 请求使用根类加载器去载入目标类，如果载入成功则跳至第8步，否则跳至第7步。
5. 当前类加载器尝试寻找Class文件，如果找到则执行第6步，如果找不到则执行第7步。
6. 从文件中载入Class，成功后跳至第8步。
7. 抛出ClassNotFountException异常。
8. 返回对应的java.lang.Class对象



#### 7.3 理解双亲委派模式

双亲委派机制，其工作原理的是，如果一个类加载器收到了类加载请求，它并不会自己先去加载，而是把这个请求委托给父类的加载器去执行，如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归，请求最终将到达顶层的启动类加载器，如果父类加载器可以完成类加载任务，就成功返回，倘若父类加载器无法完成此加载任务，子加载器才会尝试自己去加载，这就是双亲委派模式，即每个儿子都很懒，每次有活就丢给父亲去干，直到父亲说这件事我也干不了时，儿子自己才想办法去完成。 ![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/d023742afb8a454d8746147675c8b315~tplv-k3u1fbpfcp-watermark.image) 双亲委派机制的优势：采用双亲委派模式的是好处是Java类随着它的类加载器一起具备了一种带有优先级的层次关系，通过这种层级关可以避免类的重复加载，当父亲已经加载了该类时，就没有必要子ClassLoader再加载一次。其次是考虑到安全因素，java核心api中定义类型不会被随意替换，假设通过网络传递一个名为java.lang.Integer的类，通过双亲委托模式传递到启动类加载器，而启动类加载器在核心Java API发现这个名字的类，发现该类已被加载，并不会重新加载网络传递的过来的java.lang.Integer，而直接返回已加载过的Integer.class，这样便可以防止核心API库被随意篡改。 

#### 7.4 类加载器间的关系

我们进一步了解类加载器间的关系(并非指继承关系)，主要可以分为以下4点 启动类加载器，由C++实现，没有父类。 拓展类加载器(ExtClassLoader)，由Java语言实现，父类加载器为null 系统类加载器(AppClassLoader)，由Java语言实现，父类加载器为ExtClassLoader 自定义类加载器，父类加载器肯定为AppClassLoader。 

### 8 JVM可视化工具

##### 8.1为什么要可视化工具

开发大型 Java 应用程序的过程中难免遇到内存泄露、性能瓶颈等问题，比如文件、网络、数据库的连接未释放，未优化的算法等。随着应用程序的持续运行，可能会造成整个系统运行效率下降，严重的则会造成系统崩溃。为了找出程序中隐藏的这些问题，在项目开发后期往往会使用性能分析工具来对应用程序的性能进行分析和优化。

#### 8.2 visualVm

VisualVM 是一款免费的，集成了多个 JDK 命令行工具的可视化工具，它能为您提供强大的分析能力，对 Java 应用程序做性能分析和调优。这些功能包括生成和分析海量数据、跟踪内存泄漏、监控垃圾回收器、执行内存和 CPU 分析，同时，它能自动选择更快更轻量级的技术尽量减少性能分析对应用程序造成的影响，提高性能分析的精度。

他作为Oracle JDK 的一部分，位于 JDK 根目录的 bin 文件夹下。VisualVM 自身要在 JDK6 以上的版本上运行，但是它能够监控 JDK1.4 以上版本的应用程序

##### 8.2.1 打开visualVm

位于 JDK 根目录的 bin 文件夹下的jvisualvm.exe **注**：我的JDK11没有，不知道为什么，我jdk8就找的到此工具 ![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/df93997cc2614674a67ba22cab1d51c3~tplv-k3u1fbpfcp-watermark.image)![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/96115da023c649b494af8d90e151eb69~tplv-k3u1fbpfcp-watermark.image)

##### 8.2.2 本地测试项目JVM运行状态

我这本地有了好几个进程，这是我IDea工具的 ![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/4f70752b3c1b4757896e8734bb2c170c~tplv-k3u1fbpfcp-watermark.image) 我运行一个SpringBoot项目 ![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/389bcb4715c44da7a19002f0acfe3a12~tplv-k3u1fbpfcp-watermark.image)此时就开始监控了 ![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/c861a32ab4d045b5a7531573f9dc5bb2~tplv-k3u1fbpfcp-watermark.image)

##### 8.2.3 测试服务器项目JVM运行状态

省略：：： 

#### 8.3 jconsole

从Java 5开始 引入了 JConsole。JConsole 是一个内置 Java 性能分析器，可以从命令行或在 GUI shell 中运行。您可以轻松地使用 JConsole（或者，它更高端的 “近亲” VisualVM ）来监控 Java 应用程序性能和跟踪 Java 中的代码。

##### 8.3.1 启动JConsole

点击jdk/bin 目录下面的jconsole.exe 即可启动,然后会自动自动搜索本机运行的所有虚拟机进程。选择其中一个进程可开始进行监控 ![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/8239262b1e2e4352900d050fbdd0ddab~tplv-k3u1fbpfcp-watermark.image)![在这里插入图片描述](/Users/mark/typora_workspace/NoteBook/images/12e83b2bd5534d5f81c5efbede64438b~tplv-k3u1fbpfcp-watermark.image)

##### 8.3.2 远程连接项目也很简单，和 visualVm基本一致，可以自己研究一下