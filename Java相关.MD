
##多线程、并发及线程的基础问题
###1. Java 中能创建 Volatile 数组吗？
java是支持多线程的，为了解决线程并发的问题，引入了同步快**synchronized** 和 **Volatile** 关键字，   
但是Volatile是线程不安全的，比如一个变量加了关键字Volatile,在单线程时将受到保护，但是多线程同时访问时，Volatile就   
不能起到保护的作用了。   
这是因为，JVM在运行时，有一块内存区是JVM虚拟机栈，每一个线程运行时都有一个线程栈，线程栈保存了线程运行时变量的信息值，   
当线程要访问某个对象值的时候，它的流程是这样的， 先根据对象的引用--->找到堆内存中变量的值---> 找到后把堆内存中变量的   
具体值load到线程内存中---> 线程建立一个副本---> 线程内部改变值 ---> 线程退出前把改变后的值写入到堆内存中。   
![](http://i.imgur.com/cfLAN1F.png)   
   
###2. volatile 能使得一个非原子操作变成原子操作吗？
对一个 volatile 型的 long 或 double 变量的读写是原子   

###3. 你是如何调用 wait（）方法的？使用 if 块还是循环？为什么？   
wait() 方法应该在循环调用，因为当线程获取到 CPU 开始执行的时候，其他条件可能还没有满足，所以在处理前，   
循环检测条件是否满足会更好。下面是一段标准的使用 wait 和 notify 方法的代码：   
<pre>
<code>
// The standard idiom for using the wait method
synchronized (obj) {
   while (condition does not hold)
      obj.wait(); // (Releases lock, and reacquires on wakeup)   
      ...      
      // Perform action appropriate to condition
}
</code>
</pre>   

###4. 什么是线程局部变量   
线程局部变量是局限于线程内部的变量，属于线程自身所有，不在多个线程间共享。Java 提供 ThreadLocal    
类来支持线程局部变量，是一种实现线程安全的方式。但是在管理环境下（如 web 服务器）使用线程局部变量的时候要特别小心，   
在这种情况下，工作线程的生命周期比任何应用变量的生命周期都要长。任何线程局部变量一旦在工作完成后没有释放，   
Java 应用就存在内存泄露的风险   

###5. Java 中 sleep 方法和 wait 方法的区别    
虽然两者都是用来暂停当前运行的线程，但是 sleep() 实际上只是短暂停顿，因为它不会释放锁，而 wait()   
意味着条件等待，这就是为什么该方法要释放锁，因为只有这样，其他等待的线程才能在满足条件时获取到该锁。   
   
###6.什么是不可变对象（immutable object）？Java 中怎么创建一个不可变对象？   
不可变对象，顾名思义就是创建后不可以改变的对象，典型的例子就是Java中的String类。   
<pre><code>
String s = "ABC"; 
s.toLowerCase(); 
</code></pre>   
如上s.toLowerCase()并没有改变“ABC“的值，而是创建了一个新的String类“abc”，然后将新的实例的指向变量s。   
要写出这样的类，需要遵循以下几个原则：
1）immutable对象的状态在创建之后就不能发生改变，任何对它的改变都应该产生一个新的对象。
2）Immutable类的所有的属性都应该是final的。
3）对象必须被正确的创建，比如：对象引用在对象创建过程中不能泄露(leak)。
4）对象应该是final的，以此来限制子类继承父类，以避免子类改变了父类的immutable特性。
5）如果类中包含mutable类对象，那么返回给客户端的时候，返回该对象的一个拷贝，而不是该对象本身（该条可以归为第一条中的一个特例）
<pre><code>
public final class Contacts {

    private final String name;
    private final String mobile;

    public Contacts(String name, String mobile) {
        this.name = name;
        this.mobile = mobile;
    }
  
    public String getName(){
        return name;
    }
  
    public String getMobile(){
        return mobile;
    }
}
</code></pre>  

##数据类型和 Java 基础面试问题   
###7. java中int与byte，以及long与byte之间的转换   
<pre><code>
public class Utilities {  
    public static byte[] int2Bytes(int num) {  
        byte[] byteNum = new byte[4];  
        for (int ix = 0; ix < 4; ++ix) {  
            int offset = 32 - (ix + 1) * 8;  
            byteNum[ix] = (byte) ((num >> offset) & 0xff);  
        }  
        return byteNum;  
    }  
  
    public static int bytes2Int(byte[] byteNum) {  
        int num = 0;  
        for (int ix = 0; ix < 4; ++ix) {  
            num <<= 8;  
            num |= (byteNum[ix] & 0xff);  
        }  
        return num;  
    }  
  
    public static byte int2OneByte(int num) {  
        return (byte) (num & 0x000000ff);  
    }  
  
    public static int oneByte2Int(byte byteNum) {
            //针对正数的int  
        return byteNum > 0 ? byteNum : (128 + (128 + byteNum));  
    }  
  
    public static byte[] long2Bytes(long num) {  
	    byte[] byteNum = new byte[8];  
	    for (int ix = 0; ix < 8; ++ix) {  
	        int offset = 64 - (ix + 1) * 8;  
	        byteNum[ix] = (byte) ((num >> offset) & 0xff);  
	    }  
	    return byteNum;  
	}  
  
	public static long bytes2Long(byte[] byteNum) {  
	    long num = 0;  
	    for (int ix = 0; ix < 8; ++ix) {  
	        num <<= 8;  
	        num |= (byteNum[ix] & 0xff);  
	    }  
	    return num;  
	}  
</code></pre>   

###8. 存在两个类，B 继承 A，C 继承 B，我们能将 B 转换为 C 么？如 C = (C) B；  
下面的这段代码会报错哦
<pre><code>
@Test
public void testa(){
    B b = new B();
    C c = (C) b;
    System.out.println(b);
    System.out.println(c);
}
</code></pre>   

###9. Java 中 ++ 操作符是线程安全的吗   
不是线程安全的操作。它涉及到多个指令，如读取变量值，增加，然后存储回内存，这个过程可能会出现多个线程交差。   

###10. a = a + b 与 a += b 的区别   
+= 隐式的将加操作的结果类型强制转换为持有结果的类型。如果两这个整型相加，如 byte、short 或者 int，   
首先会将它们提升到 int 类型，然后在执行加法操作。如果加法操作的结果比 a 的最大值要大，   
则 a+b 会出现编译错误，但是 a += b 没问题，如下：   
<pre><code>
byte a = 127;
byte b = 127;
b = a + b; // error : cannot convert from int to byte
b += a; // ok   
</code></pre> 
（译者注：这个地方应该表述的有误，其实无论 a+b 的值为多少，编译器都会报错，因为 a+b 操作会将 a、b 提升为 int 类型，   
所以将 int 类型赋值给 byte 就会编译出错）   

###11.3*0.1 == 0.3 将会返回什么？true 还是 false？
false 因为有些浮点数不能完全精确的表示出来。 

###12. 64 位 JVM 中，int 的长度
Java 中，int 类型变量的长度是一个固定值，与平台无关，都是 32 位。意思就是说，在 32 位 和 64 位 的Java    
虚拟机中，int 类型的长度是相同的

###13. Java 堆空间及 GC
启动 Java 进程的时候，会为它分配内存。内存的一部分用于创建堆空间，当程序中创建对象的时候，就从对空间中分配内存。   
GC 是 JVM 内部的一个进程，回收无效对象的内存用于将来的分配   
栈：一种先进先出的数据结构   
堆：用二叉树形式组成的数据结构   
栈区：由编译器自动分配释放 ，存放函数的参数值，局部变量的值   
堆区：一般由程序员分配释放 ， 若程序员不释放，程序结束时可能由OS回收 。注意它与数据结构中的堆是两回事，分配方式倒是类似于链表

###14.“a==b”和”a.equals(b)”有什么区别
如果 a 和 b 都是对象，则 a==b 是比较两个对象的引用，只有当 a 和 b 指向的是堆中的同一个对象才会返回 true，   
而 a.equals(b) 是进行逻辑比较，所以通常需要重写该方法来提供逻辑一致性的比较。例如，String 类重写 equals() 方法   

###15. List、Set、Map 和 Queue    
![](http://i.imgur.com/ihWWcfm.png)  
List、Set、Queue都是事先了Collection接口，而Map就是单独的一个接口。
在Java容器中一共定义了2种集合, 顶层接口分别是**Collection**和**Map**。但是这2个接口都不能直接被实现使用，  
分别代表两种不同类型的容器。
Collection代表的是单个元素对象的序列   
public interface Collection<E> extends Iterable<E>   
Map代表的是“键值对”对象的集合   
public interface Map<K,V>   
List是存储的元素容器是有个有序的可以索引到元素的容器，并且里面的元素可以重复。   
Set里面和List最大的区别是Set里面的元素对象不可重复   
ArrrayList 底层的数据结构是数组，支持随机访问，而 LinkedList 的底层数据结构书链表，不支持随机访问。   
使用下标访问一个元素，ArrayList 的时间复杂度是 O(1)，而 LinkedList 是 O(n)   
a) Hashtable 是 JDK 1 遗留下来的类，而 HashMap 是后来增加的。   
b）Hashtable 是同步的，比较慢，但 HashMap 没有同步策略，所以会更快。   
c）Hashtable 不允许有个空的 key，但是 HashMap 允许出现一个 null key   
