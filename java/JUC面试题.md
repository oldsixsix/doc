# 说说你怎么使用synchronized的
synchronized关键字有三种主要的使用方式：
- 对对象实例加锁
- 对类的静态方法加锁，锁的是类对象
- 对代码块加锁
synchronized应用的一个场景，双重校验实现单例模式

```java
public class Singleton{
//用volatile修饰单例对象保证修改后立刻对其他线程可见
private volatile static Singleton instance;
//私有化构造方法，保证外部不可创建其对象
private Singleton(){}
//如果直接对方法加synchronized锁，效率非常低
public static Singleton getSingletonInstance(){
	//首先都进入方法
	if(instance==null)
	{
// 采用类锁一次只能通过一个，一个类只有一个class对象，所以所有的异步方法都会在这个锁这里再进行同步判断
   synchronized (SingletonDemo3.class){
                //如果能进来，表示第一个进入这个锁的
                if (instance==null)
                {
                //这里初始化后 volatile立刻使得该对象对其他线程可见
                    instance=new SingletonDemo3();
                }
            }
	}
}

}
```
# 说一说自己对于 synchronized 关键字的了解
1. synchronized关键字可以保证被它修饰的方法或者代码块在任意时刻只能有一个线程执行，
2. 底层实现原理
（1）修饰同步代码块
修饰同步代码块是通过使用monitorenter 和 monitorexit 指令实现的，其中 monitorenter 指令指向同步代码块的开始位置，monitorexit 指令则指向同步代码块的结束位置。
当执行 monitorenter 指令时，**线程试图获取锁也就是获取 monitor监视器对象**(**monitor对象存在于每个Java对象的对象头中，**) **的持有权**。当计数器为0则可以成功获取，获取后将锁计数器设为1也就是加1。
**在执行 monitorexit 指令后，将锁计数器设为0，表明锁被释放。****如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止。**
=
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201011150825743.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70#pic_center)

（2）synchronized 修饰方法的的情况
synchronized 修饰的方法并没有 monitorenter 指令和 monitorexit 指令，取得代之的确实是 ACC_SYNCHRONIZED 标识，该标识指明了该方法是一个同步方法，JVM 通过该 ACC_SYNCHRONIZED 访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用。


# synchronized 和Lock锁区别?
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200906155703650.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70#pic_center)
（1）synchronized是JVM层面实现的，java提供的关键字，Lock是JDK在Concurrent包下提供的类，API层面的锁。
（2）synchronized不需要手动释放锁，底层会自动释放，Lock则需要手动释放锁，否则有可能导致死锁。
synchronized在发生异常时候会自动释放占有的锁，因此不会出现死锁；而lock发生异常时候，不会主动释放占有的锁，必须手动unlock来释放锁，可能引起死锁的发生。（所以最好将同步代码块用try catch包起来，finally中写入unlock，避免死锁的发生。）
（3）lock等待锁过程中可以用interrupt来中断等待，而synchronized只能等待锁的释放，不能响应中断；
（4）Lock可以通过trylock来知道有没有获取锁的状态，而synchronized不能；
（5）synchronized可重入、不可中断、非公平，lock锁可重入可判断可公平。

# 谈谈 synchronized和ReentrantLock 的区别？
1. 两者都是可重入锁
自己可以再次获取自己的内部锁，比如一个线程获得了某个对象的锁，此时这个对象锁还没有释放，当其再次想要获取这个对象的锁的时候还是可以获取的，如果不可重入锁的话，就会造成死锁。**同一个线程每次获取锁，锁的计数器都自增1，所以要等到锁的计数器下降为0时才能释放锁。**
2. synchronized 依赖于 JVM 而 ReentrantLock 依赖于 API
相比synchronized，ReentrantLock增加了一些高级功能。主要来说主要有三点：①等待可中断；②可实现公平锁；
- ReentrantLock提供了一种能够中断等待锁的线程的机制，通过lock.lockInterruptibly()来实现这个机制。也就是说正在等待的线程可以选择放弃等待，改为处理其他事情。
- ReentrantLock可以指定是公平锁还是非公平锁。而synchronized只能是非公平锁。所谓的公平锁就是先等待的线程先获得锁。
3. ReentrantLock 比 synchronized 增加了一些高级功能

# java内存模型的可见性、原子性和有序性？
通常我们无法确保执行读操作的线程能适时地看到其他线程写入的值，为了确保多个线程之间对内存写入操作的可见性，必须使用同步机制。
- **可见性**：是指线程之间的可见性，一个线程修改的状态对另一个线程是可见的，也就是一个线程修改的结果，另一个线程马上能看到。比如：用volatile修饰的变量，就会具有可见性。**volatile修饰的变量不允许线程内部缓存和重排序，即直接修改内存**。所以对其他线程是可见的。但是这里需要注意一个问题，**volatile只能让被他修饰内容具有可见性，但不能保证它具有原子性**。比如 volatile int a = 0；之后有一个操作 a++；这个变量a具有可见性，但是a++ 依然是一个非原子操作，也就是这个操作同样存在线程安全问题。

- **原子性**：具有原子性就是指一个操作或者某几个操作只能在一个线程中执行完之后，另一个线程才能开始执行该操作，也就是说这些操作不可以分割在几个线程中交替执行。
比如 i++: i = i + 1 这个操作不是一个原子性操作：可以分解成三个原子性操作
1. 读取变量i的值
2. 将变量i的值加1
3. 将结果写入i变量之中
由于线程是基于处理器分配的时间片执行的，在这个过程中，这三个步骤可以让多个线程交叉执行。
**在 Java 中 synchronized 和在 lock、unlock 中操作保证原子性。**

- **有序性**：Java内存模型中，允许编辑器和处理器对指令进行重排序来提高运行性能，但是重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性。java语言通过提供 volatile 和 synchronized 两个关键字来保证线程之间操作的有序性，首先volatile 是因为其本身包含“禁止指令重排序”的语义，synchronized 是由“一个变量在同一个时刻只允许一条线程对其进行 lock 操作”这条规则获得的，此规则决定了持有同一个对象锁的两个同步块只能串行执行。

# 为什么会有线程安全问题？
https://blog.csdn.net/qingxinziran007/article/details/108028919
首先线程安全的本质是：在**多线程环境下对共享变量的读写操作导致操作结果与预测结果不一致的问题**，线程安全问题都是**由全局变量及静态变量引起的**。

线程安全问题就是由于java内存模型的可见性，有序性，原子性这三个方面导致的
首先解释一下可见性：
1. 多核cpu缓存导致的（**不可见性**），多线程在多个cpu中执行，每个cpu都有自己的缓存，线程的执行首先是cpu从内存中缓存一个变量副本到cpu中，然后对这个变量副本进行操作，操作完了之后写回CPU。这样每个线程都是操作的变量副本，线程之间的操作就不具备可见性了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200906170757620.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70#pic_center)
2. 操作系统的线程切换导致的（原子性）问题
在jave语言中有很多非原子性操作，比如++i至少需要执行三条CPU指令
指令1：首先，需要把变量 count 从内存加载到 CPU 的寄存器；
指令 2：之后，在寄存器中执行 +1 操作；
指令 3：最后，将结果写入内存（缓存机制导致可能写入的是 CPU 缓存而不是内存）。
线程A和线程B同时执行count++,结果应该是count=2;
操作系统做任务切换，可以发生在任何一条 CPU 指令执行完，是的，是 CPU 指令，而不是高级语言里的一条语句。对于上面的三条指令来说，我们假设 count=0，如果线程 A 在指令 1 执行完后做线程切换，线程 A 和线程 B 按照下图的序列执行，那么我们会发现两个线程都执行了 count+=1 的操作，但是得到的结果不是我们期望的 2，而是 1。
![在这里插入图片描述](https://img-blog.csdnimg.cn/202009061710190.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70#pic_center)
3. 编译器优化导致的有序性问题
对指令进行重排序来提高运行性能，但是重排序过程不会影响到单线程程序的执行，却会影响到多线程并发执行的正确性。

> synchronized  可以保证代码片段的原子性。
>  volatile 关键字可以保证共享变量的可见性。
>   volatile 关键字可以禁止指令进行重排序优化。

# 说说 synchronized 关键字和 volatile 关键字的区别？
（1）volatile关键字是线程同步的轻量级实现，所以volatile性能肯定比synchronized关键字要好。但是volatile关键字只能用于变量而synchronized关键字可以修饰方法以及代码块。
（2）synchronized关键字在JavaSE1.6之后进行了主要包括为了减少获得锁和释放锁带来的性能消耗而引入的偏向锁和轻量级锁以及其它各种优化之后执行效率有了显著提升，实际开发中使用 synchronized 关键字的场景还是更多一些。
（3）volatile关键字能保证数据的可见性，但不能保证数据的原子性。synchronized关键字两者都能保证。
（4）volatile关键字主要用于解决变量在多个线程之间的可见性，而 synchronized关键字解决的是多个线程之间访问资源的同步性。

# ThreadLocal简介
ThreadLocal类主要解决的就是让每个线程拥有属于自己的值，只有本线程可以获取该数据，而对于其他线程则获取不到。
实际上每个线程持有一个ThreadLocalMap对象。每一个新的线程Thread都会实例化一个ThreadLocalMap并赋值给成员变量threadLocals，使用时若已经存在threadLocals则直接使用已经存在的对象。
首先每个线程对象Thread中都会有一个ThreadLocal成员变量默认为空，当我们需要使用它的时候ThreadLocal中的get方法获取的是当前线程中的ThreadLocalMap，threadLocal实际将数据存储在ThreadLocalMap，其中key是ThreadLocal对象，value是ThreadLocal存储的对象，ThreadlocalMap本质上也是一个hashmap。

## ThreadLocal 内存泄露问题
通过查看源我们可以发现ThreadLocalMap 中**使用的 key 为 ThreadLocal 的弱引用**,而 value 是强引用。如果ThreadLocal 没有被外部强引用的情况下，在垃圾回收的时候，key 会被清理掉，而 value 不会被清理掉，key就为null，如果我们不采取任何措施的话，value就一直是强引用无法被GC回收，这个时候就可能产生内存泄露，**ThreadLocalMap实现中已经考虑了这种情况，在调用 set()、get()、remove() 方法的时候，会清理掉 key 为 null 的记录。**使用完 ThreadLocal方法后 最好手动调用remove()方法。

```java
      static class Entry extends WeakReference<ThreadLocal<?>> {
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
```
# 产生死锁的条件以及多线程中如何避免死锁？
## 首先**死锁指的是**多线程因为竞争资源而造成的一种互相等待的状态，
**死锁产生有四个条件**，
（1）互斥条件：进程要求对所分配的资源（如打印机）进行排他性控制，即在一段时间内某 资源仅为一个进程所占有。此时若有其他进程请求该资源，则请求进程只能等待。
（2）不剥夺条件：进程所获得的资源在未使用完毕之前，不能被其他进程强行夺走，即只能 由获得该资源的进程自己来释放（只能是主动释放)。
（3）请求和保持条件：进程已经保持了至少一个资源，但又提出了新的资源请求，而该资源 已被其他进程占有，此时请求进程被阻塞，但对自己已获得的资源保持不放。
（4）循环等待条件：存在一种进程资源的循环等待链，链中每一个进程已获得的资源同时被 链中下一个进程所请求。即存在一个处于等待状态的进程集合{Pl, P2, ..., pn}，其中Pi等 待的资源被P(i+1)占有（i=0, 1, ..., n-1)，Pn等待的资源被P0占有，如图2-15所示。
## 多线程中通常有三种方式避免死锁：
（1）调整加锁的顺序
当多个线程需要相同的一些锁，但是按照不同的顺序加锁，死锁就很容易发生。
如果能确保所有的线程都是按照相同的顺序获得锁，那么死锁就不会发生。
（2）设置加锁超时
在线程尝试获取锁的时候设置一个超时时间，如果超过了设定时间则该线程放弃对该锁的请求。
（3）线性资源分配法



# 通过管道进行线程通信
1. 管道流（pipeStream）是一种特殊的流，用于在不同线程间直接传送数据。
2. 一个线程发送数据到输出管道，另一个线程从输入管道中读数据。通过将输入管道和输出管道使用connect方法进行连接，实现不同线程间的通信，而无须借助于类似公共变量的读写实现通信。

# 乐观锁与悲观锁

# CAS-乐观锁
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201011201542159.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70#pic_center)
## CAS的问题
1. ABA问题
因为CAS会检查旧值有没有变化，这里存在这样一个有意思的问题。比如一个旧值A变为了成B，然后再变成A，刚好在做CAS时检查发现旧值并没有变化依然为A，但是实际上的确发生了变化。解决方案可以沿袭数据库中常用的乐观锁方式，添加一个版本号可以解决。原来的变化路径A->B->A就变成了1A->2B->3C。java这么优秀的语言，当然在java 1.5后的atomic包中提供了AtomicStampedReference来解决ABA问题，解决思路就是这样的。

2. 自旋时间过长

使用CAS时非阻塞同步，也就是说不会将线程挂起，会自旋（无非就是一个死循环）进行下一次尝试，如果这里自旋时间过长对性能是很大的消耗。如果JVM能支持处理器提供的pause指令，那么在效率上会有一定的提升。

3. 只能保证一个共享变量的原子操作

当对一个共享变量执行操作时CAS能保证其原子性，如果对多个共享变量进行操作,CAS就不能保证其原子性。有一个解决方案是利用对象整合多个共享变量，即一个类中的成员变量就是这几个共享变量。然后将这个对象做CAS操作就可以保证其原子性。atomic中提供了AtomicReference来保证引用对象之间的原子性。

# 锁的优化
https://www.jianshu.com/p/d53bf830fa09
## 偏向锁
场景：应用于同一个线程多次获取该锁，没有其他线程竞争该锁的情况下，使用偏向锁，对于这个锁来说相当于一个单线程场景，持有偏向锁的线程整个同步以及CAS操作都不需要做，降低锁的代价提高性能。

偏向锁的获取过程：
当锁对象第一次被该线程获取的时候，**虚拟机将会把对象头中的标志位设为偏向锁，使用CAS操作把获取到的锁的线程ID记录到Mark Word中，**如果CAS操作成功，持有偏向锁的线程以后每次进入这个锁相关的同步块时，都不再进行任何获取锁以及同步操作
当有另一个线程去尝试获取这个锁时，偏向模式宣告结束。根据锁对象目前是否处于被锁定的状态，撤销偏向后恢复到未锁定或轻量级锁定的状态，后续的同步操作就如上面介绍的轻量级锁那样执行
## 轻量级锁
场景：存在多个线程竞争同一个锁，但是竞争的程度很轻，可以认为两个线程对于同一个锁的操作大部分的时候都会错开，或者稍微等待一下（另一个自旋一小会就可以获取到锁）

线程在执行同步块之前，判断对象头是否是无锁状态，如果是JVM会先在当前线程的栈桢中创建用于存储锁记录的空间，并将对象头中的Mark Word复制到锁记录中，然后线程尝试**使用CAS将对象头中的Mark Word更新为指向锁记录的指针**。如果成功，当前线程获得锁，如果失败，表示其他线程竞争锁，当前线程便尝试使用自旋来获取锁。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201011212405404.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70#pic_center)
## 重量级锁
sychronized就是重量级锁。Synchronized是通过对象内部的一个叫做监视器锁（monitor）来实现的。但是监视器锁本质又是依赖于底层的操作系统的Mutex Lock来实现的。而操作系统实现线程之间的切换这就需要从用户态转换到核心态，这个成本非常高，状态之间的转换需要相对比较长的时间，这就是为什么Synchronized效率低的原因。JDK为了sychronized的优化，引入了轻量级锁和偏向锁。一个依据：“对于绝大部分的锁，在整个同步周期内都是不存在竞争的。”这是轻量级锁和偏向锁的依据。

## 偏向锁，轻量级锁及重量级锁
偏向所锁，轻量级锁都是乐观锁，重量级锁是悲观锁。
一个对象刚开始实例化的时候，没有任何线程来访问它的时候。它是可偏向的，意味着，它现在认为只可能有一个线程来访问它，所以当第一个
线程来访问它的时候，它会偏向这个线程，此时，对象持有偏向锁。偏向第一个线程，这个线程在修改对象头成为偏向锁的时候使用CAS操作，并将
对象头中的ThreadID改成自己的ID，之后再次访问这个对象时，只需要对比ID，不需要再使用CAS在进行操作。
一旦有第二个线程访问这个对象，因为偏向锁不会主动释放，所以第二个线程可以看到对象时偏向状态，这时表明在这个对象上已经存在竞争了，检查原来持有该对象锁的线程是否依然存活，如果挂了，则可以将对象变为无锁状态，然后重新偏向新的线程，如果原来的线程依然存活，则马上执行那个线程的操作栈，检查该对象的使用情况，如果仍然需要持有偏向锁，则偏向锁升级为轻量级锁，（偏向锁就是这个时候升级为轻量级锁的）。如果不存在使用了，则可以将对象回复成无锁状态，然后重新偏向。
轻量级锁认为竞争存在，但是竞争的程度很轻，一般两个线程对于同一个锁的操作都会错开，或者说稍微等待一下（自旋），另一个线程就会释放锁。 但是当自旋超过一定的次数，或者一个线程在持有锁，一个在自旋，又有第三个来访时，轻量级锁膨胀为重量级锁，重量级锁使除了拥有锁的线程以外的线程都阻塞，防止CPU空转。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201011212651553.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70#pic_center)

