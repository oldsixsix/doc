# 什么是字节码？采用字节码的好处？
在 Java 中，JVM 可以理解的代码就叫做字节码（即扩展名为 .class 的文件），它不面向任何特定的处理器，只面向虚拟机。Java 语言通过字节码的方式，在一定程度上解决了传统解释型语言执行效率低的问题，同时又保留了解释型语言可移植的特点。所以 Java 程序运行时比较高效，而且，由于字节码并不针对一种特定的机器，因此，Java 程序无须重新编译便可在多种不同操作系统的计算机上运行。
# 包装类
-  包装类是**为了解决基本数据类型无法面向对象编程所提供的**
- 包装类与基本数据类型的转换：自动装箱、拆箱
- 包装类拆箱就是基本数据类型，基本数据类型装箱就是包装类型
- 包装类的应用场景
- 创建集合类声明的泛型只能用包装类。
- 基本数据类型会有默认值，包装类默认值可以是null
- 包装类作为参数，允许参数为null

# String
- java8中，String内部使用char数组存储数据
-  value数组使用final，意味着value数组初始化之后就不能再引用其他数组了
- final修饰的变量只能进行一次赋值操作，且赋值后不可修改
-  
## 不可变性
- 可以缓存hash值，String可以用作hashMap的key
- 如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。
- **String 经常作为参数，String 不可变性可以保证参数不可变**。例如在作为网络连接参数的情况下如果 String 是可变的，那么在网络连接过程中，String 被改变，改变 String 的那一方以为现在连接的是其它主机，而实际情况却不一定是。
- **String 不可变性天生具备线程安全，可以在多个线程中安全地使用。**

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```
# 字符串常量池
- 字符串常量池的设计意图？
字符串作为最基础的数据类型，大量频繁创建字符串分配内存空间会极大的影响程序的性能，JVM为了提高性能和减少内存消耗。
 （1）每次创建字符串常量会首先在字符串常量池中判断是否存在该字符串
 （2）存在该字符串就返回其引用实例，不存在，实例化该字符串并放入池中，由于字符串是不可变的所以不用担心数据因为共享而发生冲突。
 （3）字符串常量池中存储着一个表用于维护每个字符串对象的引用。
- 字符串常量池在哪里？
字符串常量池则存在于方法区
- 如何操作字符串常量池？
java.lang.String.intern()使new创建的字符串对象指向字符串常量池中，如果不存在就创建一个，返回的是一个字符串常量池中的引用。
## String, StringBuffer and StringBuilder
**String**：不可变字符串；
底层是   private **final** char value[];

**StringBuffer**：可变字符串、效率低、线程安全；
 char[] value; （无final修饰可以改变）
 StringBuffer 的操作函数append，append 方法是由 synchronized 修饰的，是线程安全的。
 
**StringBuilder**：可变字符序列、效率高、线程不安全；
 char[] value; （无final修饰可以改变）
对于 StringBuilder 和 StringBuffer 的区别可以从下面的这张图片上看出，对于append()方法，缺少了synchronized 修饰，这使得 StringBuilder 不是一个线程安全。

## 总结
不可变性上，首先分析三个的底层结构，String的底层结构final修饰的字符数组，其余两个都是无final修饰的字符数组，String是不可变。
线程安全上，首先由于String的不可变所以它是线程安全的，StringBufffer的append追加方法是**synchronized** 修饰的同步方法，而StirngBuiler无synchronized 修饰，所以buffer是线程安全的。


# String Pool 字符串常量池
字符串常量池（String Pool）保存着所有字符串字面量（literal strings），这些字面量在编译时期就确定。不仅如此，还可以使用 String 的 intern() 方法在运行过程将字符串添加到 String Pool 中。
当一个字符串调用 intern() 方法时，如果 String Pool 中已经存在一个字符串和该字符串值相等（使用 equals() 方法进行确定），那么就会返回 String Pool 中字符串的引用；否则，就会在 String Pool 中添加一个新的字符串，并返回这个新字符串的引用。

# 值传递与引用传递
https://juejin.im/post/6844903943462453256

值传递与引用传递主要用在方法参数的传递上
1. 如果方法参数传递的是基本类型，则是值传递，传递的是基本类型字面量值的拷贝
2. 如果方法参数传递的是引用类型，则传递的是该变量所引用对象的堆内存地址的拷贝。

> java中方法参数传递方式是按值传递。 **如果参数是基本类型，传递的是基本类型的字面量值的拷贝**。 如果**参数是引用类型，传递的是该参量所引用的对象在堆中地址值的拷贝**。

## 基本数据类型作为参数传递
1. 基本类型作为参数传递--传递的是值的拷贝，无论如何改变这个拷贝，原来的值是不会改变的。

```java
public static void main(String[] args) {
    //基本类型作为参数传递
    int num = 2;
    System.out.println("before change , num = " + num);
    changeData(num);
    System.out.println("after change , num = " + num);
}
public static void changeData(int num) {
    num = 10;
}

before change , num = 2
after change , num = 2

```
## 对象作为参数传递

```java
public static void main(String[] args) {    
    //对象作为参数传递
    A a = new A("hello");
    System.out.println("before change , a.str = " + a.str);
    changeData(a);
    System.out.println("after change , a.str = " + a.str);
}

public static void changeData(A a) {
    a.str = "hi";
}

class A {
    public String str;
    public A(String str) {
        this.str = str;
    }
}

before change , a.str = hello
after change , a.str = hi
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020073122402498.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
main方法中将a的副本传入方法中，这个副本仍然是堆内存中对象的引用，拷贝的a仍然保存了堆内存对象的地址，在方法中修改的仍然指向了堆内存中的对象。


```java
public class JavaDemo {

    public static void main(String[] args) {
        
        String str = new String("ada");
        char[] ch = { 'a', 'b', 'c' };

        change(str,ch);

        System.out.print(str +" and ");
        System.out.print(ch);
    }

//复制了一份str的拷贝执行堆内存中的"ada"
//进入change方法后， str = "ada 111";
//执行的是把"ada 111"这个字符串对象的引用地址赋给str的拷贝与str无关，
//相当于str的拷贝不再指向堆内存中的"ada"了，而是指向"ada 111"
//但是main方法中的str没有改变
	
    public static void change(String str, char ch[]) {
        str = "ada 111";
        ch[0] = 'd';
    }
}

ada and dbc
```

# final
## 声明数据
声明数据为常量，可以是编译时常量，也可以是在运行时被初始化后不能被改变的常量。
 - 对于基本类型，final 使数值不变； 
 - 对于引用类型，final 使引用不变，也就不能引用其它对象，但是被引用的对象本身是可以修改的。也就是引用地址不能修改

## 声明方法
final修饰的方法不能被子类重写

## 声明类
声明的类不允许被继承

# static
## 静态变量
类的所有实例共享静态变量，可以直接通过类名来访问他，静态变量在内存中只存在一份，非静态变量属于类的实例，有多少个实例就有多少个非静态成员变量。
## 静态方法
静态方法在类加载的时候就存在了，它不依赖于任何实例。所以静态方法必须有实现，也就是说它不能是抽象方法。
静态方法在类中只能访问属于类的内存空间，不能访问类的具体实例
**只能访问所属类的静态字段和静态方法，方法中不能有 this 和 super 关键字，因此这两个关键字与具体对象关联。**

## 静态代码块
在类第一次初始化时运行一次，后面无论多少次初始化都不运行，
也就是说静态代码块只运行一次。
## 静态内部类
- 首先介绍非静态内部类，如果要创建非静态内部类的话，需要先实例化它的外部类，然后再创建其非静态的内部类。
- 而静态外部类可以直接创建不需要创建外部类
- 同样静态内部类只能访问外部类所属的静态变量和方法

## 静态成员初始化顺序
**静态变量和静态语句块优先于实例变量和普通语句块**，**静态变量和静态语句块的初始化顺序取决于它们在代码中的顺序。**

> 父类（静态变量、静态语句块）
>  子类（静态变量、静态语句块） 
>  父类（实例变量、普通语句块） 
>  父类（构造函数） 
>  子类（实例变量、普通语句块）
> 子类（构造函数）
# Object通用方法
Java中所有的类都继承自java.lang.Object类，Object类中一共有11个方法：
**Java中为原生类型(boolean,byte,char,short,int,long,float,double)和void都创建了一个预先定义好的类型，可以通过包装类的TYPE静态属性获取。上述的Integer类中TYPE和int.class是等价的。**

# 数据类型
- 基本数据类型保存的就是字面量值，存放在栈内存中
- 引用类型: 类类型、接口类型、数组保存的是引用值，引用值存储的是对象在堆内存中存储的地址。
- 基本数据类型在声明是系统就给它分配空间，无论是否赋值，声明的时候虚拟机就会分配 4字节的内存区域。引用数据类型不同，它声明时只给变量分配了引用空间，而不分配数据空间。
- String 类型时声明的时候没有分配数据空间，只有 4byte 的引用大小，在栈区，而在堆内存区域没有任何分配。


# 深拷贝与浅拷贝
## 浅拷贝
对基本数据类型进行了值传递（字面量的复制），对引用数据类型进行引用传递的拷贝，没有创建一个新的对象称为浅拷贝。
## 深拷贝
对基本数据类型进行值传递，对引用数据类型创建一个新的对象并且复制其对象的内容，称为深拷贝。

## 总结
深浅拷贝对于基本数据类型都是字面量的复制，但是对于引用类型来说深拷贝创建了一个新的对象复制了对象的内容，而浅拷贝只是复制了对象的引用而已。

## clone方法
对于clone方法来说
- 如果clone的对象中只有基本数据类型的话，那么clone出来的就是对象的深拷贝
- 如果clone的对象含有引用类型，那么就是一次浅拷贝，内部的引用对象仍然是引用传递，传递的是地址。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020080109184331.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)

# == 和equals
- 对于基本类型只能用 == 判断两个字面值是否相等
- 对于引用类型，==判断两个变量是否指向同一块堆内存地址，而equals判断的是引用对象指向内存地址的存储的内容是否相等。

eqauls实现过程
1. 判断两个对象是否是同一个对象的引用，如果是直接返回
2. 检查所属类对象是否相同，不是直接返回
3. 将object进行转型，判断每个关键域是否相等

equals 表示等价 == 表示相等严格意义上的同一块内存
# 如何重写equals方法？
首先重写equals方法需要满足一下几个要求：
a.自反性：对于任何非空的x,x.equals(x)都应该返回true
b.对称性：对于任何引用x和y，当且仅当x.equals(y)返回true时，y.equals(x)也应该返回true
c.传递性：对于任何引用x,y,z，如果x.equals(y)返回true，y.equals(z)返回true，那么x.equals(z)也应该返回true
d.一致性：如果x和y的引用没有发生变化，那么反复调用x.equals(y)的结果应该相同
e.对于任何非空的引用x,x.equals(null)应该返回false
其次具体重写equals方法的步骤
1. 判断两个对象是否是一个对象
2. 判断待检测对象是否为空
3. 判断两个对象所属类是否相同
4. 对其他类型对象进行类的强制转换以便进行下一步内容的比较
5. 比较转换后同类型对象中属性值不为空情况下是否完全一样
```java
public class A   
{   
       public boolean equals(Object otherObject)
  {
       //测试两个对象是否是同一个对象，是的话返回true
       if(this==otherObject) return true;
       //测试检测的对象是否为空，是就返回false
       if(otherObject==null) return false;
       //测试两个对象所属的类是否相同，否则返回false
       if(getClass()!=otherObject.getClass())  return false;
       //对otherObject进行类型转换以便和类A的对象进行比较
       A other=(A)otherObject;
       //对于值可能为null的属性，检测时应使用Object的equals方法，不为null的可以直接使用==检测
       return Object.equals(类A对象的属性A，other的属性A）&&类A对象的属性B==other的属性B……;
   }    
}
```

# hashCode()
hashCode() 返回哈希值，而 equals() 是用来判断两个对象是否等价。**等价的两个对象散列值一定相同，但是散列值相同的两个对象不一定等价，这是因为计算哈希值具有随机性，两个值不同的对象可能计算出相同的哈希值。**

- 值不同的对象也可能计算相同的散列值，值相同的对象散列值一定相同。
- **HashSet 和 HashMap 等集合类使用了 hashCode() 方法来计算对象应该存储的位置**，因此要将对象添加到这些集合类中，需要让对应的类实现 hashCode() 方法。


# 抽象类与接口
## 概念
抽象类对事物进行抽象，捕获公共特性和行为
接口核心就是对行为进行抽象
- 抽象类是用于捕捉一类事务的通用特性的包括属性及其行为
- 使用接口对行为进行抽象，接口的核心是定义行为，表示其实现类可以做什么，至于实现类主体是谁，如何实现接口并不关心。
## 区别
**语法上**
只能继承一个类，但是可以实现多个接口
抽象类和接口都不能被实例化
接口是抽象类的眼神，可以看成是一个完全抽象的类
接口默认的字段都是public static final 


## 设计上
抽象类提供了里氏替换原则，子类对象可以替换掉所有的父类对象 is A 关系
接口实现的是一种 LIKE A 关系


# 反射
反射在程序运行状态中，对任意一个类(指的是.class文件)，动态的获取这个类的所有的属性和方法，以及动态的调用类对象的属性和方法。比如在JDK的动态代理就用到了反射在运行中动态的获取类对象的方法并对方法的实现进行增强。
（getProxyClass(ClassLoader loader, Class<?>...interfaces)）
newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)

# 注解
spring中配置bean需要在 spring.xml 文件配置具体的信息。我们使用了注解以后，可以直接在 类的源代码上，增加注解…bean 就被配置到 Spring容器中管理 了。也就是，注解可以给类、方法上注入信息。
# JAVA8新特性
1. Lamda表达式
Lambda 表达式，也可称为闭包，它是推动 Java 8 发布的最重要新特性。
**Lambda 允许把函数作为一个方法的参数**（函数作为参数传递进方法中）。
使用 **Lambda 表达式可以使代码变的更加简洁紧凑**。
# 集合
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200801115551712.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
## 说说 List,Set,Map 三者的区别？

> List(对付顺序的好帮手)： 存储的元素是**有序的、可重复的。**
>  Set(注重独一无二的性质): 存储的元素是**无序的、不可重复的**。
>  Map(用Key 来搜索的专家): 使用键值对（kye-value）存储，类似于数学上的函数 y=f(x)，“x”代表 key，"y"代表value，**Key 是无序的、不可重复的**，value 是无序的、可重复的，每个键最多映射到一个值。

## LIST（有序集合）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200801115511664.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)

> Arraylist： Object[]数组
>  Vector：Object[]数组
>  LinkedList： **双向链表**(JDK1.6 之前为循环链表，JDK1.7 取消了循环) 
### Arraylist 和 Vector 的区别?
底层都是Object[] 但是Vector是线程安全的
- ArrayList 是 List 的主要实现类，底层使用 Object[ ]存储，适用于频繁的查找工作，线程不安全 ；
- Vector 是 List 的古老实现类，底层使用 Object[ ]存储，线程安全的。
### Arraylist 与 LinkedList ？
1. 都是线程不安全，底层：一个是数组一个是双向链表
2. 数组支持快速随机访问，链表不支持
3. 内存：Arraylist需要预留一定的容量，linklist需要保存当前节点的前驱后继以及数据
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200801120108596.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
### ArrayList的扩容机制
https://github.com/Snailclimb/JavaGuide/blob/master/docs/java/collection/ArrayList-Grow.md
1. 有三种方式对应三种构造方法对ArrayList进行初始化
默认构造函数，创建初始容量为空的数组，第一次添加的时候初始容量为10
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200801121457341.png)
带初始容量参数的构造函数（用户自己指定容量）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200801121619142.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)

传入一个集合利用迭代器进行构造
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020080112164963.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
## Set集合

### 比较 HashSet、LinkedHashSet 和 TreeSet 三者的异同
HashSet 是 Set 接口的主要实现类 ，HashSet 的底层是 HashMap，线程不安全的，可以存储 null 值；

LinkedHashSet 是 HashSet 的子类，能够按照添加的顺序遍历；

TreeSet 底层使用红黑树，能够按照添加元素的顺序进行遍历，排序的方式有自然排序和定制排序。


## Map接口
https://blog.csdn.net/weixin_41105242/article/details/106972635
### HashMap 和 Hashtable 的区别
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020080112242542.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
## HashMap
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200801151545616.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
HashMap数据结构：1.8之前，用数组+链表的方式组成，数组是hashMap的主体，链表则是为了解决hash冲突。
当两个对象调用的hashCode方法计算的哈希值一致导致计算的在数组中存储的索引值相等的情况，而采用拉链法解决冲突。
jdk1.8后主要增加了红黑树用于解决hash冲突，当链表长度大于阈值（默认是8）且数组长度大于64时，此时该索引位置上的所有数据都会使用红黑树进行存储。
当阈值大于8但是数组长度小于64时，此时并不会将链表变为红黑树，而是选择对数组进行扩容。
这样做的目的是因为数组比较小，尽量避开红黑树结构，这种情况下变为红黑树结构，反而会降低效率，因为红黑树需要逬行左旋，右旋，变色这些操作来保持平衡。同时数组长度小于64时，搜索时间相对要快些。所以结上所述为了提高性能和减少搜索时间，底层阈值大于8并且数组长度大于64时，链表才转换为红黑树，具体可以参考 treeifyBin() 方法。

小结
特点：
1.存储无序的

2.键和值位置都可以是null，但是键位置只能是一个null

3.键位置是唯一的，底层的数据结构控制的

4.jdk1.8前数据结构是**链表+数组**，jdk1.8之后是**链表+数组+红黑树**

5.**阈值（边界值）>8并且数组长度大于64**，才将链表转换为红黑树，变为红黑树的目的是为了高效的查询。

6.每次对数组扩容是都是x2变为原来的两倍，并且将原有的数据复制过来重新进行hash再存储，这一过程会比较耗时


问题：1.8为什么引入红黑树？这样结构的话不是更麻烦了吗，为何阈值大于8且数组长度大于64换成红黑树？
jdk1.8以前HashMap的实现是数组+链表，**即使哈希函数取得再好，也很难达到元素百分百均匀分布**。**当HashMap中有大量的元素都存放到同一个桶中时，这个桶下有一条长长的链表，**这个时候HashMap就相当于一个单链表，**假如单链表有n个元素，遍历的时间复杂度就是O(n)，完全失去了它的优势。**

红黑树解决的是：当链表比较长的时候，提高查询效率的方法 


添加数据到HashMap中的过程
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200801160007695.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
threshold（临界值）= capacity（容量）* loadFactor（负载因子）。这个值是当前已占用数组长度的最大值。size超过这个值就重新resize（扩容），扩容后的HashMap容量是之前容量的两倍。
16*0.75=12 默认情况下数组长度超过12就会扩容

## hashMap面试题
1. HashMap中hash函数是怎么实现的？还有哪些hash函数的实现方式？

> 答：底层对于hashMap的key值的hashCode结合数组长度进行hash操作，操作有无符号右移、按位异或、按位与来计算出存储的索引。
> hash函数还有平方取中法，伪随机数法和取余数法。这三种效率都比较低。。

2. 当两个对象的hashcode相等时会怎么样？

> 答：会产生哈希碰撞。若key值内容相同则替换旧的value，否则就连接到链表后面，如果链表长度超过阈值8，且数组超过临界值会先进行扩容，如果数组长度超过64，且链表长度超过8，就转换为红黑树存储。

3. HashMap的扩容机制

> 首先数组容量是有限的，当达到临界值的时候就会进行扩容，也就是resize
> 临界值=数组当前的长度 x 负载因子  默认是 16 x  0.75 = 12
> 扩容分为两步，首先创建一个长度是原来数组两倍的空数组，然后把原来数组中的所有数据重新hash到新的数组中
> 注：数组长度变化了hash值也会随之改变， Hash的公式---> index = HashCode（Key） & （Length - 1） 

4. 为什么hashMap底层的数组长度必须是2的n次幂

> 这个和hash算法的公式有关，为了极可能的保证hash算法结果的均匀分布，如果是2的n次幂的话，length-1的二进制就全是1，那么index的结果就等同于hashcode的后几位的值了，只要保证得到的hashCode分布均匀那么index就会分布均匀。

5. 为什么我们重写equals方法的时候需要重写hashcode方法
如果为重写的话，object中实现的equal是比较的两个对象的内存地址，而我们hashMap中equals在链表中找到具体的key
重写hashcode就是为了保证不同的对象返回不同的hash值，相同的对象返回相同的hash值
> 在未重写equals方法我们是继承了object的equals方法，**那里的 equals是比较两个对象的内存地址**，显然我们new了2个对象内存地址肯定不一样
> 对于值对象，== 比较的是两个对象的值
> 对于引用对象， == 比较的是两个对象的地址
大家是否还记得我说的HashMap是通过key的hashCode去寻找index的，那index一样就形成链表了，也就是说”帅丙“和”丙帅“的index都可能是
6. HashMap线程不安全如何解决
线程安全的有HashTable、ConcurrentHashMap、SynchronizedMap，性能最好的是**ConcurrentHashMap**，一般都用它来解决。

**Hashtable线程安全但效率低下**
Hashtable容器使用**synchronized**来保证线程安全，但在线程竞争激烈的情况下Hashtable的效率非常低下。因为当一个线程访问Hashtable的同步方法时，其他线程访问Hashtable的同步方法时，可能会进入阻塞或轮询状态。如线程1使用put进行添加元素，线程2不但不能使用put方法添加元素，并且也不能使用get方法来获取元素，所以竞争越激烈效率越低。

**ConcurrentHashMap并发机制**
1.7中采用分段锁的方式来保证线程安全，每一把锁用于锁住数组中一部分数据，这样就可以解决多线程访问容器中不同数据段数据不需要竞争锁的问题，有效提高了并发效率
1.8中采用了CAS+synchronized的方式保证线程安全，数据结构采用：数组+链表+红黑树。CAS(比较并交换的原子操作)
V O N 内存值 旧值 新值  取内存地址中的值与旧值比较，如果相同，则表示无线程占用将N（新值）写入进去。】

7. 红黑树原理是什么？

> 红黑树是一个含有红黑节点的平衡二叉树，每个节点不是红色就是黑色，根节点和叶子节点都是黑色，且每个红色结点的两个子结点一定都是黑色。需要通过左旋、右旋、上色来将链表转换成红黑树，主要用于提高链表中的查找效率，牺牲了插入和删除的效率，每次插入和删除都需要再进行调整恢复红黑树。


# java内存模型
java将内存分为工作内存与主内存，其中主内存是线程间共享的，工作内存是线程独占。
当多个线程访问同一个对象或者变量的时候，由于每个线程都需要将对象、变量拷贝到工作内存中，然后在工作内存中对变量进行修改最后写入主内存，但是修改的时候是在工作内存中进行的，工作内存线程私有不与其他线程共享，当线程修改变量值的时候对其他线程不可见，为了保证线程的可见性，java提供了volatile关键字。volatile关键字修饰变量，**就是告知线程对该变量的访问必须从主内存中获取。而对它的改变必须同步刷新到主内存中。这样就能保证线程对变量访问的可见性。**关于volatile的使用，参看如下例子：

# java线程间通信
## 同步机制
通过synchronized 方法对线程运行加锁，保证线程间对公共读写变量的同步读写
## 等待/通知机制

> Java多线程的等待/通知机制是基于Object类的wait()方法和notify(), notifyAll()方法来实现的。
> wait()方法让持有锁的线程释放锁，
> notify持有锁的线程通知等待的线程不用等待了，但如果需要让线程得到锁，必须调用wait方法释放持有的锁。
## 信号量
`volatile`关键字的自己实现的信号量通信。


# HTTP协议
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200803115457596.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
## 三次握手
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200803115719956.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200803142035642.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200803142134672.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70)

# Java语言的特点
- 平台无关性，java语言可以一次编译处处运行，字节码文件有JVM虚拟机上执行有很好的跨平台性。
- 支持分布式应用和多线程
- 提供了对Web应用开发的支持
- 具有较好的安全性和健壮性。Java的强类型机制、垃圾回收器、异常处理和安全检查机制使得用Java语言编写的程序有很好的健壮性。
- 去除了C++语言中难以理解、容易混淆的特性，例如头文件、指针、结构、单元、运算符重载、虚拟基础类、多重继承等，使得程序更加严谨、简洁。


# 异常
## 什么是异常？
程序异常产生的不正常的情况称为异常，Throwable类是java语言中所有错误或者异常的超类
java异常的体系结构
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200901173330837.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3MTI2MTc1,size_16,color_FFFFFF,t_70#pic_center)
1. Error： 程序不应该捕捉的错误，应该交由JVM来处理。
2. Exception：程序中应该要捕获的错误。里面又分为：运行时异常和非运行时异常
（1）RuntimeException：运行时异常，也叫未检查异常，是Exception的子类，但不需捕捉的异常超类，但是实际发生异常时，还是会导致程序停止运行的的，只是编译时没有报错而已。比如除数为零，数组空指针等等，这些都是在运行之后才会报错。此类异常，可以处理也可以不处理，并且可以避免。

（2）在Exception的所有子类中 除了RuntimeException类和它的子类，**其他类都叫做非运行时异常**，或者叫已检查异常，通常被定义为Checked类**，是必须要处理可能出现的异常，否则编译就报错了**。Checked类主要包含：IO类和SQL类的异常情况，这些在使用时经常要先处理异常（使用throws或try catch捕获）。

java几种常见的异常：
运行时异常：
1，java.lang.ArrayIndexOutOfBoundsException 数组索引越界异常。当对数组的索引值为负数或大于等于数组大小时抛出。
2，ArithmeticException 算术错误情形，如以零作除数，算术条件异常。
3 java.lang.SecurityException 安全性异常
4，IllegalArgumentException 方法接收到非法参数，非法参数异常！
5，java.lang.ArrayStoreException 数组中包含不兼容的值抛出的异常
6，java.lang.NegativeArraySizeException 数组长度为负异常
7 java.lang.ClassNotFoundException 找不到类异常。当应用试图根据字符串形式的类名构造类，而在遍历CLASSPAH之后找不到对应名称的class文件时，抛出该异常。
8 java.lang.NullPointerException 空指针异常。当应用试图在要求使用对象的地方使用了null时，抛出该异常。譬如：调用null对象的实例方法、访问null对象的属性、计算null对象的长度、使用throw语句抛出null等等。
9，java.lang.NumberFormatException（数字格式转换异常）
10，java.lang.ClassCastException(强制类型转换异常)

IOException
1， IOException 操作输入流和输出流时可能出现的异常
2， EOFException 文件已结束异常
3， FileNotFoundException 文件未找到异常

异常的处理：
异常的处理分为消极的处理（自己处理不了，就往调用它的地方上抛throws，异常没有解决，只是抛出）和积极处理（异常捕获，捕捉异常通过try-catch语句或者try-catch-finally语句实现）

# 了解过动态代理么，怎么实现动态代理？
首先实现动态代理的方式通常有JDK动态代理机制和CGLIB动态代理机制这两个的实现方式有一些区别
## JDK动态代理机制
首先JDK动态代理机制中需要代理类实现InvocationHandler接口并重写invoke方法，同时被代理对象也需要实现接口，解释一下为什么被代理类一定要实现接口，原因是JDK动态代理在创建代理对象的时候是调用newProxyInstance方法，该方法通过反射获取需要传入类的加载器，以及被代理类的接口实现以及实现InvocationHandler接口的对象。通过代理对象调用方法的时候实际会调用到InvocationHandler 中重写的invoke方法，在invoke方法中可以动态增强被代理类的方法。

## CGLIB的动态代理机制
JDK动态代理机制的一个缺点就是只能代理实现了接口的类，CGLIB动态代理机制就弥补了这一缺点，通过继承的方式实现了代理，例如Spring中的AOP模块，如果目标对象实现了接口，则默认使用JDK动态代理，如果没有实现接口则使用CGLIBE动态代理。
在CGLIBE动态代理机制中`MethodInterceptor 接口`和 `Enhancer 类`是核心。
1. 首先需要创建代理类实现MethodInterceptor接口重写intercept，在intercept方法实现动态代理增强被代理对象的方法
2.  Enhancer类的create方法来创建代理类。该代理类继承了被代理类，当代理类调用方法，实际调用的是 MethodInterceptor 中的 intercept 方法。

# Object类常用方法
-    public final native Class<?> getClass(); 返回此Object的运行时类
-    public native int hashCode();  
- 	 public boolean equals(Object obj)
-    protected native Object clone() throws CloneNotSupportedException;
-    public String toString()
-    public final native void notify();
-    public final native void notifyAll();
-    public final native void wait(long timeout) throws InterruptedException;
-    protected void finalize() throws Throwable { }

```java
private static native void registerNatives();
/*对象初始化时自动调用此方法*/
static {
    registerNatives();
}
/*返回此Object的运行时类*/
public final native Class<?> getClass();
/*
hashCode的常规协定是：
1.在java应用程序执行期间,在对同一对象多次调用hashCode()方法时,必须一致地返回相同的整数,前提是将对象进行equals比较时所用的信息没有被修改。从某一应用程序的一次执行到同一应用程序的另一次执行,该整数无需保持一致。
2.如果根据equals(object)方法,两个对象是相等的,那么对这两个对象中的每个对象调用hashCode方法都必须生成相同的整数结果。
3.如果根据equals(java.lang.Object)方法,两个对象不相等,那么对这两个对象中的任一对象上调用hashCode()方法不要求一定生成不同的整数结果。但是,程序员应该意识到,为不相等的对象生成不同整数结果可以提高哈希表的性能。
*/
public native int hashCode();
/*比较对象的内存地址,跟String.equals方法不同,它比较的只是对象的值*/
public boolean equals(Object obj) {
    return (this == obj);
}
/*本地clone方法,用于对象的复制*/
protected native Object clone() throws CloneNotSupportedException;
/*
返回该对象的字符串表示,非常重要的方法
getClass().getName();获取字节码文件的对应全路径名例如java.lang.Object
Integer.toHexString(hashCode());将哈希值转成16进制数格式的字符串。
*/
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
/*唤醒在此对象监视器上等待的单个线程*/
public final native void notify();
/*唤醒在此对象监视器上等待的所有线程*/
public final native void notifyAll();
/*
在其他线程调用此对象的notify()方法或notifyAll()方法前,导致当前线程等待。换句话说,此方法的行为就好像它仅执行wait(0)调用一样。
当前线程必须拥有此对象监视器。该线程发布对此监视器的所有权并等待,直到其他线程通过调用notify方法或notifyAll方法通知在此对象的监视器上等待的线程醒来,然后该线程将等到重新获得对监视器的所有权后才能继续执行。
*/
public final native void wait(long timeout) throws InterruptedException;
/*在其他线程调用此对象的notify()方法或notifyAll()方法,或者超过指定的时间量前,导致当前线程等待*/
public final void wait(long timeout, int nanos) throws InterruptedException {
    if (timeout < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (nanos < 0 || nanos > 999999) {
        throw new IllegalArgumentException(
                            "nanosecond timeout value out of range");
    }

    if (nanos > 0) {
        timeout++;
    }

    wait(timeout);
}
public final void wait() throws InterruptedException {
    wait(0);
}

/*当垃圾回收期确定不存在对该对象的更多引用时,由对象的垃圾回收器调用此方法。*/
protected void finalize() throws Throwable { }
```

