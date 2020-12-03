### 类的加载时机
1. 当虚拟机启动时，初始化用户指定的主类，就是启动执行的 main 方法所在的类;
2. 当遇到用以新建目标类实例的 new 指令时，初始化 new 指令的目标类，就是 new 一个类的时候要初始化;
3. 当遇到调用静态方法的指令时，初始化该静态方法所在的类;
4. 当遇到访问静态字段的指令时，初始化该静态字段所在的类;
5. 子类的初始化会触发父类的初始化;
6. 如果一个接口定义了 default 方法，那么直接实现或者间接实现该接口的类的初始化， 会触发该接口的初始化;
7. 使用反射 API 对某个类进行反射调用时，初始化这个类，其实跟前面一样，反射调用 要么是已经有实例了，要么是静态方法，都需要初始化;
8. 当初次调用 MethodHandle 实例时，初始化该 MethodHandle 指向的方法所在的 类。



### 不会初始化

1. 通过子类引用父类的静态字段，只会触发父类的初始化，而不会触发子类的初始化。
2. 定义对象数组，不会触发该类的初始化。
3. 常量在编译期间会存入调用类的常量池中，本质上并没有直接引用定义常量的类，不 会触发定义常量所在的类。
4. 通过类名获取 Class 对象，不会触发类的初始化，Hello.class 不会让 Hello 类初始 化。
5. 通过 Class.forName 加载指定类时，如果指定参数 initialize 为 false 时，也不会触 发类初始化，其实这个参数是告诉虚拟机，是否要对类进行初始化。Class.forName (“jvm.Hello”)默认会加载 Hello 类。
6. 通过 ClassLoader 默认的 loadClass 方法，也不会触发初始化动作(加载了，但是 不初始化)。


### 三类加载器

1. 启动类加载器(BootstrapClassLoader) 
    负责加载JRE的核心类库，如jre目标下的rt.jar,charsets.jar等
2. 扩展类加载器(ExtClassLoader)
    负责加载JRE扩展目录ext中JAR类包
3. 应用类加载器(AppClassLoader)
    负责加载ClassPath路径下的类包




### 加载器特点
1. 双亲委托 
ClassLoader使用的是双亲委托模型来搜索类的，每个ClassLoader实例都有一个父类加载器的引用
这个过程是由上至下依次检查的，首先由最顶层的类加载器Bootstrap ClassLoader试图加载，如果没加载到，则把任务转交给Extension ClassLoader试图加载，如果也没加载到，则转交给App ClassLoader 进行加载，如果它也没有加载得到的话，则返回给委托的发起者，由它到指定的文件系统或网络等URL中加载该类。如果它们都没有加载到这个类时，则抛出ClassNotFoundException异常

 因为这样可以避免重复加载，当父亲已经加载了该类的时候，就没有必要子ClassLoader再加载一次。考虑到安全因素，我们试想一下，如果不使用这种委托模式，那我们就可以随时使用自定义的String来动态替代java核心api中定义的类型，这样会存在非常大的安全隐患，而双亲委托的方式，就可以避免这种情况，因为String已经在启动时就被引导类加载器（Bootstrcp ClassLoader）加载，所以用户自定义的ClassLoader永远也无法加载一个自己写的String，除非你改变JDK中ClassLoader搜索类的默认算法。
 
 优点  
1. 可以保证java核心类库的安全，即保证由引导类加载器加载的类不能被用户随便替换，用户不能自己随便定义一个二进制名也为
java.lang.String 的类来替换java核心类库的java.lang.String类，否则会抛出ClassCastException。


2. 使得一个类的不同版本可以共存在jvm中，带来了极大的灵活性，OSGi技术的实现就是得益于此。

缺点：  

 而根据一个类的定义加载器是这个类中引用的其它类的初始加载器可知，java核心类库中定义的类是不能使用系统类加载器定义的类。而java提供了很多服务提供者接口（Service Provider Interface SPI),许可第三方来实现这些类的接口。第三方开发的类通常是由应用类加载器在类路径下（classpath）来找到并且定义的。引导类加载器是无法找到 SPI 的实现类的，因为它只加载 Java 的核心库。它也不能代理给系统类加载器，因为它是系统类加载器的祖先类加载器。也就是说，类加载器的双亲委托模型无法解决这个问题，这是双亲委托模型的缺点。



2. 负责依赖 


3. 缓存加载


### JVM 命令行工具
1. jps/jinfo 查看 java 进程
  jps/jps -lmv
2. jstat 查看 JVM 内部 gc 相关信息   
   stat -gc 1763 1000 100 
   每隔1000 ms，1000次  
   stat -gcutil 1763 1000 1000（使用率）  
3. jmap 查看 heap 或类占用空间统计 
  jmap -histo 1763   
  jmap -heap 1763  
4. jstack 查看线程信息  
  jstack -l 1763  
5. jcmd 执行 JVM 相关分析命令(整合命令)  
  jcmd pid VM.version  
  jcmd pid VM.flags  
  jcmd pid VM.command_line  
  jcmd pid VM.system_properties   
  jcmd pid Thread.print  
  jcmd pid GC.class_histogram  
  jcmd pid GC.heap_info     
6. jrunscript/jjs 执行 js 命令  
  当curl命令用:   
  jrunscript -e "cat('http://www.baidu.com')" 执行js脚本片段  
  jrunscript -e "print('hello,kk.jvm'+1)" 执行js文件  
  jrunscript -l js -f /XXX/XXX/test.js  
7. JVM 图形化工具--jconsole  
   JVM 图形化工具--jvisualvm  
   JVM 图形化工具--jmc  
   
### JVM
Eden so s1 8:1:1  
由如下参数控制提升阈值 -XX:+MaxTenuringThreshold=15  
mark-and-sweep algorithm.  
   The algorithm traverses all object references, starting with the GC roots, and marks every object found as alive.    
   All of the heap memory that is not occupied by marked objects is reclaimed. It is simply marked as free, essentially swept free of unused objects.  
可以作为 GC Roots 的对象  
1. 当前正在执行的方法里的局部变量和输入参数  
2. 活动线程(Active threads)  
3. 所有类的静态字段(static field) 
4. JNI 引用

停止-复制(mark-copy)  
标记-清除(Mark-Sweep)  
标记-整理(Mark-Compact)  
分代收集算法(Generational Collection)  
效率：复制算法>标记/整理算法>标记/清除算法（此处的效率只是简单的对比时间复杂度，实际情况不一定如此  
内存整齐度：复制算法=标记/整理算法>标记/清除算法。  
内存利用率：标记/整理算法=标记/清除算法>复制算法。 

1. Parallel GC  
-XX:+UseParallelGC
2. Mostly Concurrent Mark and Sweep Garbage Collector  
-XX:+UseConcMarkSweepGC
3. G1 GC

### NIO
端口：进程  
ip：计算机  

## 阻塞式 IO

一般通过在 while(true) 循环中服务 端会调用 accept() 方法等待接收客户 端的连接的方式监听请求，请求一旦 接收到一个连接请求，就可以建立通 信套接字在这个通信套接字上进行读 写操作，此时不能再接收其他客户端 连接请求，只能等待同当前连接的客 户端的操作执行完成， 不过可以通过 多线程来支持多个客户端的连接

 Runnable#run()没有返回值   
 Callable#call()方法有返回值  
 
 创建固定线程池的经验  
 
 1. 如果是CPU密集型，则线程池大小设置成N或N+1  
 2. 如果是IO密集型，则线程池大小设置为2N或2N+2  
 
 ### Spring AOP
 
AOP-面向切面编程  
Spring 早期版本的核心功能，管理对象生命周期与对象装配  
为了实现管理和装配，一个自然而然的想法就是，加一个中间层代理(字节码增强)来实现所有对象 的托管  
IoC-控制反转  
也称为 DI(Dependency Injection)依赖注入  
对象装配思路的改进  
从对象 A 直接引用和操作对象 B，变成对象 A 里指需要依赖一个接口 IB，系统启动和装配阶段，把 IB 接口的实例对象注入到对象 A，这样 A 就不需要依赖一个 IB 接口的具体实现，也就是类 B  
从而可以实现在不修改代码的情况，修改配置文件，即可以运行时替换成注入 IB 接口另一实现类 C 的一个对象实例  


实现AOP的技术，主要分为两大类：一是采用动态代理技术，利用截取消息的方式，对该消息进行装饰，以取代原有对象行为的执行；  
二是采用静态织入的方式，引入特定的语法创建“方面”，从而使得编译器可以在编译期间织入有关“方面”的代码。  


## Class 文件级加载
Java 源代码 --> java 编译器 --> .class文件(二进制文件) -> JVM 虚拟机读取字节码文件，取出二进制数据，加载到内存中  
https://blog.csdn.net/zhangjg_blog/article/details/21486985
class文件是一种8位字节的二进制流文件， 各个数据项按顺序紧密的从前向后排列， 相邻的项之间没有间隙， 这样可以使得class文件非常紧凑， 体积轻巧， 可以被JVM快速的加载至内存， 并且占据较少的内存空间。 我们的Java源文件， 在被编译之后， 每个类（或者接口）都单独占据一个class文件， 并且类中的所有信息都会在class文件中有相应的描述， 由于class文件很灵活， 它甚至比Java源文件有着更强的描述能力。

class文件中的信息是一项一项排列的， 每项数据都有它的固定长度， 有的占一个字节， 有的占两个字节， 还有的占四个字节或8个字节， 数据项的不同长度分别用u1, u2, u4, u8表示， 分别表示一种数据项在class文件中占据一个字节， 两个字节， 4个字节和8个字节。 可以把u1, u2, u3, u4看做class文件数据项的“类型” 。

