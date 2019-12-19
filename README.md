慕课-Java并发编程精讲



**实现Runnable接口好在哪里？**
继承Thread类是不推荐的，因为它有以下的一些缺点：
1. 从代码架构角度：具体的任务（run方法）应该和“创建和运行线程的机制（Thread类）”解耦，
用runnable对象可以实现解耦。
2. 使用继承Thread的方式的话，那么每次想新建一个任务，只能新建一个独立的线程，而这样做
的损耗会比较大（比如重头开始创建一个线程、执行完毕以后再销毁等。如果线程的实际工作内
容，也就是run()函数里只是简单的打印一行文字的话，那么可能线程的实际工作内容还不如损
耗来的大）。如果使用Runnable和线程池，就可以大大减小这样的损耗。
3. 继承Thread类以后，由于Java语言不支持双继承，这样就无法再继承其他的类，限制了可扩展
性。
通常我们优先选择方法1。





**两种方法的本质对比**
方法一和方法二，也就是“实现Runnable接口并传入Thread类”和“继承Thread类然后
重写run()”在实现多线程的本质上，并没有区别，都是最终调用了start()方法来新建
线程。这两个方法的最主要区别在于run()方法的内容来源：
    @Override
    public void run() {
         if (target != null) {
            target.run();
         }
    }
方法一：最终调用target.run();
方法二：run()整个都被重写


**有多少种实现多线程的方式？**
答题思路，有以下5点：
1. 从不同的角度看，会有不同的答案。
2. 典型答案是两种，分别是实现Runnable接口和继承Thread类，然后具体展开说；
3. 但是，我们看原理，其实Thread类实现了Runnable接口，并且看Thread类的run方法，会发现
其实那两种本质都是一样的，run方法的代码如下：
    @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
方法一和方法二，也就是“继承Thread类然后重写run()”和“实现Runnable接口并传入Thread类”在实
 现多线程的本质上，并没有区别，都是最终调用了start()方法来新建线程。这两个方法的最主要区别
 在于run()方法的内容来源：
 方法一：最终调用target.run();
 方法二：run()整个都被重写
 4. 然后具体展开说其他方式；
 还有其他的实现线程的方法，例如线程池等，它们也能新建线程，但是细看源码，从没有逃出过
 本质，也就是实现Runnable接口和继承Thread类。
 5. 结论:我们只能通过新建Thread类这一种方式来创建线程，但是类里面的run方法有两种方式来实
 现，第一种是重写run方法，第二种实现Runnable接口的run方法，然后再把该runnable实例传
 给Thread类。除此之外，从表面上看线程池、定时器等工具类也可以创建线程，但是它们的本
 质都逃不出刚才所说的范围。
 以上这种描述比直接回答一种、两种、多种都更准确。


**启动线程的正确和错误的方式**
start()和run()的比较
执行run()是主线程执行
用start()是新建的线程执行



**start方法的执行流程是什么？**
1. 检查线程状态，只有NEW状态下的线程才能继续，否则会抛出IllegalThreadStateException（在
运行中或者已结束的线程，都不能再次启动，详见CantStartTwice10类）
2. 被加入线程组
3. 调用start0()方法启动线程
注意点：
start方法是被synchronized修饰的方法，可以保证线程安全；
由JVM创建的main方法线程和system组线程，并不会通过start来启动。


**start调用native start0()再调用run方法**

**1、一个线程两次调用start()会出现什么情况？**
会抛出线程状态异常。start()方法在执行后（先检查线程状态、后执行）时候会先检查所有执行的线程的状态，如果线程的状态为0（即为全新线程）才会执行后面的添加入线程组的操作，然后调用native start0方法，
在start0里面调用run()方法
所以如果直接调用run()方法就是一个普通方法，必须调用start()才会启动一个线程，执行线程的各个生命周期

**停止线程**
    原理介绍：使用interrupt来通知线程，而不是强制。Java中无法强制停止，是否停止决定权在于线程
    
    
**Java中停止线程的原则是什么？**
在Java中，最好的停止线程的方式是使用中断interrupt，但是这仅仅是会通知到被终止的线程“你该
停止运行了”，被终止的线程自身拥有决定权（决定是否、以及何时停止），这依赖于请求停止方和
被停止方都遵守一种约定好的编码规范。
任务和线程的启动很容易。在大多数时候,我们都会让它们运行直到结束,或者让它们自行停止。然而,
有时候我们希望提前结束任务或线程,或许是因为用户取消了操作,或者服务需要被快速关闭，或者是
运行超时或出错了。
要使任务和线程能安全、快速、可靠地停止下来,并不是一件容易的事。Java没有提供任何机制来安
全地终止线程。但它提供了中断( Interruption),这是一种协作机制,能够使一个线程终止另一个线程
的当前工作。
这种协作式的方法是必要的,我们很少希望某个任务、线程或服务立即停止,因为这种立即停止会使共
享的数据结构处于不一致的状态。相反,在编写任务和服务时可以使用一种协作的方式:当需要停止时,
它们首先会清除当前正在执行的工作,然后再结束。这提供了更好的灵活性,因为任务本身的代码比发
出取消请求的代码更清楚如何执行清除工作。
生命周期结束(End-of-Lifecycle)的问题会使任务、服务以及程序的设计和实现等过程变得复杂,而这
个在程序设计中非常重要的要素却经常被忽略。一个在行为良好的软件与勉强运的软件之间的最主
要区别就是,行为良好的软件能很完善地处理失败、关闭和取消等过程。
本章将给出各种实现取消和中断的机制,以及如何编写任务和服务,使它们能对取消请求做出响应


**这是因为，interrupt是一个信号，这个信号并不是实时检测的，也不是每行代码都能检测的，在本例中，只有程序运行到sleep时，
才能检测到线程被中断了，然后会抛出异常；而代码在计算其他内容时，比如执行if (num % 100 == 0)时，是无法感知到中断已经发生了的。
所以问题不在于休眠时间的长短，而是在于“每次循环都有休眠，所以终究会运行到休眠的sleep代码，在sleep时才会抛出异常”。**


**如何停止线程？**
A. 用volatile的boolean作为标记来停止
B. 用stop()方法让线程停止
C. 用interrupt来请求线程停止
解答：
应该选C。
1. 原理：用interrupt来请求线程停止而不是强制，好处是安全。
2. 想停止线程，要请求方、被停止方、子方法被调用方相互配合才行：
a) 作为被停止方：每次循环中或者适时检查中断信号，并且在可能抛出InterrupedException的
地方处理该中断信号；
b) 请求方：发出中断信号；
c) 子方法调用方（被线程调用的方法的作者）要注意：优先在方法层面抛出
InterrupedException，或者检查到中断信号时，再次设置中断状态；
3. 最后再说错误的方法：stop/suspend已废弃，volatile的boolean无法处理长时间阻塞的情况


**无法响应中断时如何停止线程？**
A. 用interrupt方法来请求停止线程
B. 不可中断的阻塞无法处理
C. 根据不同的类调用不同的方法
解答：
应该选C。
如果线程阻塞是由于调用了 wait()，sleep() 或 join() 方法，你可以中断线程，通过抛出
InterruptedException 异常来唤醒该线程。
但是对于不能响应InterruptedException的阻塞，很遗憾，并没有一个通用的解决方案。
但是我们可以利用特定的其它的可以响应中断的方法，比如ReentrantLock.lockInterruptibly()，比如
关闭套接字使线程立即返回等方法来达到目的。
答案有很多种，因为有很多原因会造成线程阻塞，所以针对不同情况，唤起的方法也不同。
总结就是说如果不支持响应中断，就要用特定方法来唤起，没有万能药。

**无法响应不可中断的阻塞
我们可以通过中断来唤醒阻塞，但是无法响应不可中断的阻塞，没有办法**

**线程的6种状态**
New
Runnable(可运行 待执行和执行中状态，包括没有CPU时间片的等待期间)
Blocked(只要synchronize修饰才会进入此状态)
Waiting(不带时间参数)
Timed Waiting(带时间参数)
Terminated

sleep:阻塞，不会释放锁
wait:阻塞，会释放锁
notify:唤醒线程
notifyAll:唤醒所有线程

**wait、notify、notifyAll 属于Object类的，必须使用在sync()中

从Object.wait()状态刚被唤醒时，通常不会立刻抢到那就会从Waiting先进入Blocked状态，抢到锁后再转换状态（官方文档）
如果发生异常，可以直接跳到Terminated状态，不必再遵循路径，比如可以从waiting直接到Terminated **


**JDK中sleep()、wait()等会抛中断异常的方法在抛出异常之前会清除线程的中断标识**

**sleep()**
sleep方法可以让线程进入Waiting状态，并且不占用CPU资源，但是不释放锁，直到规定时间后再执行，休眠期间如果被中断，会抛出异常并清除中断状态。


**wait和sleep方法的异同？**
你知道wait和sleep方法的异同吗？
解答：
相同
1. Wait和sleep方法都可以使线程阻塞，对应线程状态是Waiting或Time_Waiting。
2. wait和sleep方法都可以响应中断Thread.interrupt()。
不同
1. wait方法的执行必须在同步方法中进行，而sleep则不需要(原因：线程更安全，避免死锁和永久等待)。
2. 在同步方法里执行sleep方法时，不会释放monitor锁，但是wait方法会释放monitor锁。
3. sleep方法短暂休眠之后会主动退出阻塞，而没有指定时间的 wait方法则需要被其他线程中断后才能退出阻塞(原因：因为wait可能没有时间参数)。
4. wait()和notify(),notifyAll()是Object类的方法，sleep()和yield()是Thread类的方法(Java对象设计的每个类都有一把锁，对象的头中都有几位保留monitor的预留)


其实join源码中线程run()结束后会自动执行notifyAll()操作

join()期间，新线程会让主线程处于waiting状态

yield释放时间片，状态还是Runnable，不会释放锁，等待下次时间片执行
JVM不保证执行，最好不要使用yield()
yield()和sleep()区别：是否随时可能被再次调度

捕获子线程异常：
1：不推荐，在子线程中run方法中try catch
2: 使用自己写的UncaughtExceptionHandler


解决发布逸出的问题:
逸出： 1、方法返回一个private对象   解决方法：每次返回一个副本
      2、还未初始化（构造函数还没完全执行完毕）就把对象提供给外界  解决方法：工厂模式
      
      
JVM内存结构 VS Java内存模型 VS Java对象模型

JMM最主要的内容：重排序、可见性、原子性
重排序好处：提高处理速度
重排序的3种情况：编译器优化、CPU指令重排、内存的“重排序”

编译器：JVM、JIT
CPU：CPU总之可能会重拍序
内存：线程A修改线程B却看不到，引发可见性问题


happens-before是解决可见性问题的
happens-before：
    1、单线程规则
    2、锁操作(synchronized和Lock)
    3、volatile
    4、线程启动
    5、join
    6、传递性
    7、中断
    8、构造方法
    9、工具类的happens-before原则：
        1、线程安全的容器get一定能看到put等操作
        2、countDownLatch()
        3、Semaphore(信号量)
        4、Future
        5、线程池
        6、CyclicBarrier
        
        
volatile的作用：1、解决可见性  2、禁止重排序
volatile不适用场景：
    1、a++的场合
适用场景：

最好是单纯的赋值操作，不能取决于之前的状态

synchronized 保证了原子性和可见性


单例模式作用：1、节省内存和计算 2、保证结果正确 3、方便管理
单例模式的适用场景： 1、无状态的工具类 2、全局信息类

什么是死锁？
    1、发生在并发中
    2、互不相让

死锁的4个必要条件，缺一不可：
1、互斥条件
2、请求和保持条件
3、不剥夺条件
4、循环等待条件

修复死锁的策略：
鸵鸟策略：不推荐
避免策略：避免锁的获取顺序
检测和恢复策略


死锁检测与恢复：
进程终止（逐个终止线程）
资源抢占（线程回退几步）


避免死锁的几个建议：
1、设置超时时间 （tryLock）
2、多使用并发类工具
3、尽量降低锁的使用粒度：用不同的锁而不是一个锁
4、如果能使用同步代码块，就不要使用同步方法
