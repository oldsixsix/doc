# 介绍一下Java运行时数据区各个区域功能
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200826221213597.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200923093001710.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70#pic_center)

 -  Java运行时数据区主要包含6个部分分别是：堆、方法区、虚拟机栈、程序计数器、本地方法栈、运行时常量池。
 - 堆：java堆被所有线程所共享，其唯一的用途就是存放对象实例，几乎所有的对象实例都在堆中分配内存。
 - 方法区：也是被各个线程所共享的区域，它用于存储已被虚拟机加载的类信息、常量、静态变量
 - 运行时常量池：JDK1.7 及之后版本的 JVM 已经将运行时常量池从方法区中移了出来，在 Java 堆（Heap）中开辟了一块区域存放运行时常量池。这块区域在 1.7 之前原来是方法区的一部分，运行时常量池存储的东西较为复杂，主要分为字面量和符号引用。字面量存放的字面量主要包括 常量（final 修饰的），比如：final int x = 1、静态变量（static 修饰的），还有一些其他的字面量。
符号引用：符号引用主要包括：类的完全限定名、字段名称和描述符、方法名称和描述符，包括很多符号，比如：() 也可以看做符号引用。
- 虚拟机栈也是线程私有的，所以它的生命周期与程序计数器相同。虚拟机栈描述的是 Java 方法执行的内存模型。
- 程序计数器是一块很小的区域，它存储的是当前线程正在执行的字节码的地址，程序计数器是唯一的一块在 Java 虚拟机规范中没有规定任何 OutOfMemoryError 的区域。由于它是线程私有的，所以它的生命周期随着线程的创建而创建，随着线程的结束而死亡 。
- 本地方法栈与虚拟机栈类似，虚拟机栈是执行 Java 方法开辟的内存空间，而本地方法栈是执行 Native 方法开辟的内存空间。


# 什么情况下会发生栈内存溢出？
思路： 描述栈定义，再描述为什么会溢出，再说明一下相关配置参数，OK的话可以给面试官手写是一个栈溢出的demo。
我的答案：
- 栈是线程私有的，他的生命周期与线程相同，每个方法在执行的时候都会创建一个栈帧，用来存储局部变量表，操作数栈，动态链接，方法出口等信息。局部变量表又包含基本数据类型，对象引用类型
- 如果线程请求的栈深度大于虚拟机所允许的最大深度，将抛出StackOverflowError异常**，方法递归调用本身就会产生这种结果。**
- 如果Java虚拟机栈可以动态扩展，并且扩展的动作已经尝试过，但是无法申请到足够的内存去完成扩展，或者在新建立线程的时候没有足够的内存去创建对应的虚拟机栈，那么Java虚拟机将抛出一个OutOfMemory 异常。(线程启动过多)
- 参数 -Xss 去调整JVM栈的大小


# JVM内存为什么要分代。新生代中为什么要分为Eden和Survivor?
- 内存分代是根据对象存活周期的不同将内存划分为几块，然后每一块根据此块内对象的存活时间特点不同，选择不同的垃圾回收算法。
- 年轻代：绝大多数刚创建的对象会被分配在Eden区，其中的绝大多数对象很快就会消亡。Eden区是连续的内存空间，因此在其上分配内存极快；当Eden区满的时候，执行Minor GC。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200826224206809.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70#pic_center)
将年轻代内存块分为一块较大的eden空间和两块较小的survivor空间，每次进行minorGC时将eden和其中一块survivor中还存活的对象全部一次性复制到另一块上，最后清理掉eden和那一块survivor，如果另一块survivor没有足够的内存空间存放上次新生代收集下来的存活对象时，这些对象将被转移到老年代。
## 为什么要分为Eden和Survivor?为什么要设置两个Survivor区？
如果没有Survivor，Eden区每进行一次Minor GC，存活的对象就会被送到老年代。老年代很快被填满，触发Major GC.老年代的内存空间远大于新生代，进行一次Full GC消耗的时间比Minor GC长得多,所以需要分为Eden和Survivor。
Survivor的存在意义，就是减少被送到老年代的对象，进而减少Full GC的发生，Survivor的预筛选保证，只有经历16次Minor GC还能在新生代中存活的对象，才会被送到老年代。
**设置两个Survivor区最大的好处就是解决了碎片化**，刚刚新建的对象在Eden中，经历一次Minor GC，Eden中的存活对象就会被移动到第一块survivor space S0，Eden被清空；等Eden区再满了，就再触发一次Minor GC，Eden和S0中的存活对象又会被复制送入第二块survivor space S1（这个过程非常重要，因为这种复制算法保证了S1中来自S0和Eden两部分的存活对象占用连续的内存空间，避免了碎片化的发生）


## 整个过程
清理过程
1.        创建新对象，大多数放在Eden区
2.        Eden满了（或达到一定比例），触发Minor GC,   把有用的复制到Survivor1, 同时清空Eden区。
3.        Eden区再次满了，出发Minor GC, 把Eden和Survivor1中有用的，复制到Survivor2, 同时清空Eden，Survivor1。
4.        Eden区第三次满了，出发Minor GC, 把Eden和Survivor2中有用的，复制到Survivor1, 同时清空Eden，Survivor2。
        形成循环，Survoivor1和Survivor中来回清空、复制，过程中有一个Survivor处于空的状态用于下次复制的。如果存活对象过多，幸存者区放不下会将其放入老年代中

5.        重复多次（默认15），没有被Survivor清理的对象，复制到Old（Tenuerd）区.
这样循环多次，每次存活下来的对象age+1，当对象的age超过阈值15还没有释放，则说明对象存活时间较长，则将对象赋值到老年代中存储，
6.        当Old达到一定比例，触发FULL GC，清理老年代。FULL GC采用的是标记整理算法来完成老年代的垃圾回收。
7.        当Old满了，触发Full GC。注意，Full GC清理代价大，系统资源消耗高。

# 讲讲类的加载机制
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200905104700484.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70#pic_center)


1. 类的加载指什么？
首先类的加载是指将类的.class文件的二进制数据读入到内存中，将其放在运行时数据区的方法区内，然后在堆区创建一个java.lang.Class对象，用来封装类在方法区内的数据结构。类加载的最终结果是产生位于堆区的Class对象，Class对象封装了类在方法区的数据结构，并提供了访问接口。
2. 类的生命周期以及类的加载过程
加载 → 验证 → 准备 → 解析 → 初始化  → 使用 → 卸载
类的加载过程包括 加载 → 验证 → 准备 → 解析 → 初始化 这五个阶段，，加载、验证、准备和初始化这四个阶段发生的顺序是确定的，而解析阶段则不一定，它在某些情况下可以在初始化阶段之后开始，这是为了支持Java语言的运行时绑定（也成为动态绑定或晚期绑定）。

## 加载
查找并加载类的二进制数据
 加载时类加载过程的第一个阶段，在加载阶段，虚拟机需要完成以下三件事情：
 1、类加载器通过一个类的全限定名来获取其定义的二进制字节流。
 2、将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
 3、在Java堆中生成一个代表这个类的java.lang.Class对象，作为对方法区中这些数据的访问入口。
 
 ## 连接（ 验证 → 准备  → 解析 ）
验证：确保被加载的类的正确性
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200905102719330.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70#pic_center)
准备：为类的静态变量分配内存，并将其初始化为默认值
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200905102813369.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70#pic_center)
解析：把类中的符号引用转换为直接引用
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200905103229302.png#pic_center)
## 初始化
初始化，为类的静态变量赋予正确的初始值，JVM负责对类进行初始化，主要对类变量进行初始化。在Java中对类变量进行初始值设定有两种方式：
 ①声明类变量是指定初始值
 ②使用静态代码块为类变量指定初始值
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200905103440181.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70#pic_center)

## 结束生命周期
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200905103613320.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70#pic_center)
# 类加载器
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200905110322687.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200905110345656.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70#pic_center)
## JVM类加载机制
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200905110403529.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70#pic_center)

  # GC
  ## 那些内存需要回收
  程序计数器、虚拟机栈、本地方法栈3个区域随线程而生、随线程而灭，因为方法结束或者线程结束时，内存自然就跟随着回收了，就不需要过多考虑回收的问题。
  由于java运行时内存区域中，堆和方法区只有在程序运行过程中才会知道需要创建那些对象，分配内存和回收内存都是动态，所以需要进行回收。
  ##  如何回收
  在堆里面存放着Java世界中几乎所有的对象实例，垃圾收集器会判定对象是否”存活”（与引用有关），不“存活”的对象便会在适当的时候通过垃圾收集器回收。
  ### 如何判断对象是否是垃圾
  1. 引用计数法
	给对象添加一个引用计数器，每当有一个地方引用它的时候，计数器的值就加1，当引用失效的时候，计数器的值就减1.
	当计数器为0的对象就是不能再被使用的了，这个时候就会判定这些对象为垃圾通知垃圾收集器进行回收。
	缺点：存在相互引用的问题，这样两个垃圾对象互相引用，无法释放内存。Java虚拟机中没有选择这种方法来管理内存。
  2. 可达性分析算法
	通过一系列称为“GC Roots”的根对象作为起始点，从这些节点开始向下搜索，所走过的路径称为引用链（Reference Chain），当一个					对象到GC Roots没有任何引用链项链时，则证明此对象是不可达的，则将此对象标记为垃圾
	那些可以作为GC ROOT对象
1.虚拟机栈（栈帧中的局部变量表）中的引用对象
2.方法区中类静态属性（静态对象）引用的对象
3.方法区中常量（final 修饰的成员对象）引用的对象。
4.本地方法栈中JNI（Native）引用的对象。

对象引用的强弱
**强引用** ： 类似Object obj=new Object()这类的引用，只要强引用还在，垃圾收集器永远不会回收掉被引用的对象。
**软引用** ： 软引用是用来描述一些有用但非必须的对象，在系统将要发生内存溢出（OOM）异常之前，将会把这些对象列进回收范围之中进行二次回收。如果这次回收还没有足够的内存，才会抛出内存溢出（OOM）异常。JDK1.2之后提供了SoftReference类来实现软引用。
**弱引用** ： 弱引用也是用来描述非必需对象的，但是它的引用比软引用更弱一些，被弱引用关联的对象只能生存到下一次垃圾收集发生之前。JDK1.2之后提供了WeakReference类来实现软引用。
**虚引用** ： 虚引用也成为幽灵引用或者幻影引用，它是最弱的一种引用关系。无法通过虚引用来取得一个对象的实例。为一个对象设置虚引用的唯一目的就是能够在这个对象被回收的时候收到一个系统通知。

 ### 选择合适的算法进行垃圾回收
1. 标记清除算法
2. 复制算法
3. 标记整理算法
4. 分代收集算法
  
# 双亲委派模型
双亲委派模型的工作流程是：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把请求委托给父加载器去完成，依次向上，因此，所有的类加载请求最终都应该被传递到顶层的启动类加载器中，只有当父加载器在它的搜索范围中没有找到所需的类时，即无法完成该加载，子加载器才会尝试自己去加载该类。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200905110508111.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70#pic_center)
作用：
1、防止重复加载同一个.class。通过委托去向上面问一问，加载过了，就不用再加载一遍。保证数据安全。
2、保证核心.class不能被篡改。通过委托方式，不会去篡改核心.clas，即使篡改也不会去加载，即使加载也不会是同一个.class对象了。不同的加载器加载同一个.class也不是同一个Class对象。这样保证了Class执行安全。


![在这里插入图片描述](https://img-blog.csdnimg.cn/20201001225815125.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70#pic_center)


