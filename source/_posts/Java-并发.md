---
title: Java 并发
date: 2017-04-12 18:40:22
categories: Java
mathjax: false
tags: [Thinking in Java]
---
<!--more-->
## 写在前面
多线程和并发可以是专门的一本书了，这里简单学习了下，参考阅读《Thinking in Java》“并发”部分。

*新类库中的构件在实际应用中应该挺重要的，我看到最后觉得太累了，反而还没看。和新IO简直是一模一样的情况啊。*

## 并发的多面性
简单地说，Java的多线程机制就是在一个进程内(系统分配的资源固定)，将CPU的时间切片再分给不同的线程。这样使得控制权的转移对编程人员来说是透明的（或者说是黑盒）。对于多CPU来说，并发操作无疑是高效的。
学习并发，也有利于理解构架分布式系统时候用到的消息机制。
**另外，有函数型编程可以将并发任务彼此隔离。这点需要之后学习过程中接着体会，**

## 基本的线程机制
### 定义任务
```Java
//Thread的start()自动调用Runnable的run()
public class Multitask {
	public static void	main(String[] argus) {
		for (int i = 0; i < 3; i++) {
			LiftOff liftOff = new LiftOff();
			Thread t = new Thread(liftOff);
			t.start();
		}
		System.out.println("main thread still runs");
	}
}
class LiftOff implements Runnable{
	protected int countDown = 10;
	private static int taskCount = 0;
	private final int id = taskCount++;
	public LiftOff() { }
	public LiftOff(int countDown) {
		this.countDown = countDown;
	} 
	public String status(){
		return "#"+id+"("+(countDown>0?countDown:"Liftoff!")+").";
	}
	@Override
	public void run() {
		System.out.print("thread"+id+"runs:");
		while(countDown-->0){
			System.out.print(status());
			Thread.yield();//表示暂停当前线程，执行其他线程(包括自身线程) 
			//加入yield()使得每次打印后都会切换线程，不加则由cpu觉得切换的时间
		}
	}
}
/*
main thread still runs
thread1runs:thread0runs:thread2runs:#1(9).#0(9).#2(9).#1(8).#2(8).#0(8).#2(7).
#1(7).#2(6).#0(7).#2(5).#1(6).#0(6).#2(4).#1(5).#0(5).#2(3).#1(4).#0(4).#2(2).
#1(3).#2(1).#0(3).#1(2).#2(Liftoff!).#0(2).#1(1).#0(1).#1(Liftoff!).#0(Liftoff!).
*/
```
以上主线程和t线程间的调度是由线程调度器自动控制的，如果在多CPU机器上，线程调度器会分发线程给不同处理器。而线程调度器的调度机制是非确定性的，因此输出先后(执行先后)无法保证。

### 使用Executor
在JavaSE5(也就是jdk1.5)的`java.util.concurrent.*`中执行器(`Executor`)将管理Thread对象。将上述的main()中代码更改如下：
```Java
public static void	main(String[] argus) {
	ExecutorService exec = Executors.newCachedThreadPool();
	for (int i = 0; i < 3; i++) {
		LiftOff liftOff = new LiftOff();
		exec.execute(liftOff);
	}
	exec.shutdown();
	System.out.println("main thread still runs");
}
```
`CachedThreadPool`会为每个`Runnable`创建线程，而线程由`Executor`管理，其中`shutdown()`方法是防止`Executor`有新任务提交。
另外还有

* `FixedThreadPool`固定了线程池的数量，
* 和`SingleThreadExecutor`(相当于`FixedThreadPool`为1的时候)保证其他进程不会被并发调用。如果向`SingleThreadExecutor`提交多个任务，这些任务将会排队即序列化任务。
将上述的main()中代码更改如下：
```Java
public static void	main(String[] argus) {
	ExecutorService exec = Executors.newSingleThreadExecutor();
	for (int i = 0; i < 3; i++) {
		LiftOff liftOff = new LiftOff();
		exec.execute(liftOff);
	}
	exec.shutdown();
	System.out.println("main thread still runs");
}
/*
thread0runs:main thread still runs
#0(9).#0(8).#0(7).#0(6).#0(5).#0(4).#0(3).#0(2).#0(1).
#0(Liftoff!).thread1runs:#1(9).#1(8).#1(7).#1(6).#1(5).
#1(4).#1(3).#1(2).#1(1).#1(Liftoff!).thread2runs:#2(9).
#2(8).#2(7).#2(6).#2(5).#2(4).#2(3).#2(2).#2(1).#2(Liftoff!).
*/
```
### 从任务中产生返回值
```Java
import java.util.ArrayList;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class Multitask {
	public static void	main(String[] argus) {
		ExecutorService exec = Executors.newSingleThreadExecutor();
		ArrayList<Future<String>> results = new ArrayList<>();
		for (int i = 0; i < 3; i++) {
			TaskWithResults task = new TaskWithResults(i);
			Future<String> result = exec.submit(task);//submin返回Future对象
			results.add(result);
		}
		for (Future<String> future : results) {
			if (future.isDone()) {
				try {
					System.out.println(future.get());
				} catch (InterruptedException e) {
					e.printStackTrace();
				} catch (ExecutionException e) {
					e.printStackTrace();
				} finally {
					exec.shutdown();//必须有
				}
			}
		}
	}
}
class TaskWithResults implements Callable<String>{
	private int id;
	public TaskWithResults(int id) {
		this.id = id;
	}
	@Override
	public String call() throws Exception {
		return "result of task "+id;
	}
}
/*
result of task 0
result of task 1
result of task 2
*/
```

### 休眠
将run()中的yield()改为sleep()，使得任务中止执行一定时间
```Java
class LiftOff implements Runnable{
	protected int countDown = 10;
	private static int taskCount = 0;
	private final int id = taskCount++;
	public LiftOff() { }
	public LiftOff(int countDown) {
		this.countDown = countDown;
	} 
	public String status(){
		return "#"+id+"("+(countDown>0?countDown:"Liftoff!")+").";
	}
	@Override
	public void run() {
		try {
			while(countDown-->0){
				System.out.print(status());
				//Thread.yield();
				//Old style
				//Thread.sleep(100);
				//Java SE5/6 style
				TimeUnit.MILLISECONDS.sleep(100);
			}
		} catch (InterruptedException e) {
			e.printStackTrace();//异常不能跨线程
		}

	}
}
```

### 优先级
优先级只是决定执行的频率，而一般也不设置优先级。并且不同系统优先级等级分层不同，因此建议使用`MAX_PRIORITY`、`MIN_PRIORITY`、`NORM_PRIORITY`等
```Java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
public class Multitask {
	public static void	main(String[] argus) {
		ExecutorService exec = Executors.newCachedThreadPool();
		for (int i = 0; i < 5; i++) {
			LiftOff liftOff = new LiftOff(Thread.MIN_PRIORITY);
			exec.execute(liftOff);
		}
		exec.execute(new LiftOff(Thread.MAX_PRIORITY));
		exec.shutdown();
	}
}
class LiftOff implements Runnable{
	protected int countDown = 5;
	private volatile double d;//发现 volatile 变量的最新值
	private int priority;
	public LiftOff(int priority) {
		this.priority = priority;
	} 
	public String toString(){//覆盖
		return Thread.currentThread()+":"+countDown;
	}
	@Override
	public void run() {
		Thread.currentThread().setPriority(priority);
		while(true) {
			for (int i = 0; i < 100000; i++) {
				d += (Math.PI+Math.E)/(double)i;
				if(i%1000==0){
					Thread.yield();
				}
			}
			System.out.println(this);
			if (--countDown==0) {
				return;
			}
		}
	}
}
```
```Java
/*
Thread[pool-1-thread-6,10,main]:5
Thread[pool-1-thread-3,1,main]:5
Thread[pool-1-thread-5,1,main]:5
Thread[pool-1-thread-1,1,main]:5
Thread[pool-1-thread-6,10,main]:4
Thread[pool-1-thread-2,1,main]:5
Thread[pool-1-thread-4,1,main]:5
Thread[pool-1-thread-6,10,main]:3
Thread[pool-1-thread-1,1,main]:4
Thread[pool-1-thread-5,1,main]:4
Thread[pool-1-thread-3,1,main]:4
Thread[pool-1-thread-4,1,main]:4
Thread[pool-1-thread-2,1,main]:4
Thread[pool-1-thread-6,10,main]:2
Thread[pool-1-thread-1,1,main]:3
Thread[pool-1-thread-5,1,main]:3
Thread[pool-1-thread-3,1,main]:3
Thread[pool-1-thread-4,1,main]:3
Thread[pool-1-thread-6,10,main]:1
Thread[pool-1-thread-2,1,main]:3
Thread[pool-1-thread-3,1,main]:2
Thread[pool-1-thread-5,1,main]:2
Thread[pool-1-thread-1,1,main]:2
Thread[pool-1-thread-4,1,main]:2
Thread[pool-1-thread-2,1,main]:2
Thread[pool-1-thread-3,1,main]:1
Thread[pool-1-thread-1,1,main]:1
Thread[pool-1-thread-5,1,main]:1
Thread[pool-1-thread-4,1,main]:1
Thread[pool-1-thread-2,1,main]:1
*/
```
### 让步
`yield()`表示**建议**暂停当前线程，执行其他相同优先级的其他线程。
### 后台线程
非后台线程结束后，会杀死所有后台线程
```Java
import java.util.concurrent.TimeUnit;
public class Multitask {
	public static void	main(String[] argus) {
		Thread t = new Thread(new ADaemon());
		t.setDaemon(true);
		t.start();
		System.out.println("finish");
	}
}
class ADaemon implements Runnable{
	@Override
	public void run() {
		try {
			System.out.println("Starting ADaemon");
			TimeUnit.SECONDS.sleep(1);
		} catch (InterruptedException e) {
			e.printStackTrace();
		} finally {
			System.out.println("always run?");
		}
	}

}
/*
finish
Starting ADaemon
*/
```
同时，后台线程创建的线程仍然是后台线程

### 编码的变体
看源码Thread类也是实现Runnable接口。
所以实现线程有这么几种方式：

* 实现`Runnable`接口
    * 实现其中的`run()`
    * 传入`Thread`对象，执行`start()`。  
* 自管理的`Runnable`。
    * 类中变量`Thread t = new Thread(this)`，并在构造函数中执行`t.start()`
* 直接继承`Thread`类
    * 实现`run()`
    * 构造函数中执行`start()`
* 用内部类隐藏线程代码
    * 实现一个内部类，该内部类继承`Thread`
    * 实现该内部类的`run()`和构造函数中执行`start()`
    * 该类有一个变量为这个内部类`private InnerClassName test;`
    * 在类的构造函数中new这个内部类`test = new InnerClassName()`
* `Executor`管理的线程池
    * 实现`Runnable`接口
    * 传入到`Executor`中

### 加入一个线程 Join()
如下，Joiner线程需要等到Sleeper线程结束或者中断`sleeper.interrupt()`，才会接着执行
```Java
class Joiner extends Thread{
	private Thread sleeper;
	public Joiner(String name, Thread sleeper) {
		super(name);
		this.sleeper = sleeper;
		start();
	}
	@Override
	public void run() {
		try {
			sleeper.join();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println(getName()+" join completed");
	}
}
```
### 捕获异常
JavaSE5前需要用线程组。现在给线程附上一个异常处理前就可以捕获异常了。
当异常传到线程外时，已经无法被捕捉(unknown source)，即使外部再调用try-catch也没用。
但是用
```Java
Thread t = new Thread(new Runnable类);
t.setUncaughtExceptionHandler(
    new UncaughtExceptionHandler() {
		@Override
		public void uncaughtException(Thread t, Throwable e) {			
		//do something
		}
	}//匿名内部类
);
```
也可以统一给Thread设置一个静态域
```Java
Thread.setUncaughtExceptionHandler(new MyUncaughtExceptionHandler())
```
## 共享受限资源
解决共享资源竞争：

* synchronized关键字
对资源加锁。第一个访问该资源的任务（或者说是线程）锁定该资源。
Java中，当任务要执行synchronized关键字保护的代码片段时，会检查锁是否可用，然后获得锁。通常将域设置为private，而能访问该域的方法都用synchronized。
synchronized则是在类的范围内防止对static数据的并发访问。
* Lock对象
JavaSE5中还有Lock对象的显式互斥机制。人工加锁解锁
比之synchronized，Lock更灵活，但是代码更多。一般用前者。但是当有特殊要求synchronized获取锁失败、需要给获取锁加一定时间期限等，用Lock类自定义显然更理想。
```Java
private Lock lock = new ReentrantLock();
public int next(){
    lock.lock;
    try{
        currValue++;
        currValue++;
        return currValue;
    } finally{
        lock.unlock();
    }
}
```
* volatile关键字
volatile获取原子性和可视性，避免编译器的优化。Java中的基本类型的操作是原子性的，但是读写long、double等会出现分成两个字节来处理的情况，不是原子性的。并且注意到，Java递增操作也不是原子性的。
* 原子类
JavaSE5引入了一些原子性变量类，主要用于性能调优。Atomic类主要用来构建java.util.concurrent中的类。详情见书
* 临界区
想要只对部分代码（临界区）进行控制，可以用同步控制块来处理
```Java
synchronized(指定某个对象如syncObject){
//为进入这段代码，需要获得该对象syncObject的锁
//最好是使用其方法正在被调用的当前对象即synchronized(this)
}
```
* 在其他对象上同步
即如下代码传入其他对象。为了起到互斥作用，所有相关的任务应该都是在这个对象上完成的，否则达不到目的
```Java
private Object syncObject = new Object();
synchronized(syncObject){
//something
}
```
* 线程本地存储
```Java
public static ThreadLocal<Integer> value = new ThreadLocal<>(){
	protected synchronized Integer initialValue() {
		return 3;//为每个线程都返回值
	}
};
```
## 终结任务
exec.awaitTermination(250,TimeUnit.MILLISECONDS)若所有任务都在超时前结束，返回true

### sleep的阻塞
wait() 或者sleep()会使得线程进入阻塞状态，在阻塞时终结线程，需要执行中断操作。

* `Thread`类有`interrupt()`方法，执行后会抛出`InterruptedException`异常
* `Thread.interrupted()`来做`run()`中的循环条件
* `Executor`调用`shutdownNow()`会中断所有的任务
* Executor调用submit()得到的返回值泛型`Future<?> tmp`，执行`tmp.cancle(true)`将会中断特定的线程

### io的阻塞
**注意到之前IO阻塞和synchronized造成的阻塞无法中断**
解决：

* 在`Executor`调用`shutdownNow()`后，其他IO的阻塞需要用`System.in.close()`和`ServerSocket`的`close()`方法手动关掉发生阻塞的底层资源
* 对于NIO的channel类，则
    * `Executor`调用`shutdownNow()`会中断所有的任务
    * Executor调用submit()得到的返回值泛型`Future<?> tmp`，执行`tmp.cancle(true)`将会中断特定的线程
    * 仍旧可以手动关掉资源`close()`

### 被互斥所阻塞
为解决synchronized造成的阻塞无法中断，JavaSE5添加了特性，`ReentrantLock`上阻塞的任务可以被中断
即在`Runnable`中阻塞
```Java
private Lock lock = new ReentranLock();
lock.lock();
//something
//lock.unlock();//注释后造成阻塞
```
可以用`Thread`的`interrupt()`中断

### 检查中断
假如并没有阻塞，导致`interrupt()`后继续`run()`
考虑到`InterruptedException`异常和`Thread.interrupted()`都会在调用后清楚中断的状态，因此我们可以在`run()`中这么处理：
```Java
public void run(){
    while(!Thread.interrupted()){
    //do something
    }
}
```
## 线程之间的协作
### wait() notifyAll()

* `wait()`期间锁是释放的
* 可以通过`notify()`、`notifyAll()` 或者`wait()`时间到期，从wait中恢复执行
* `wait()`、`notify()`、`notifyAll()`只能在同步控制方法或同步控制块中调用
```Java
public void run(){
    while(Thread.interrupted()){
        //something
        /* synchronized的方法中 
        flag_A = true;
        notifyAll();
        */
        
        /* synchronized的方法中 
        while(flag_B == false) wait();
        */
    }
}
```
实际情况中，可能线程1还未`wait()`,线程2已经发出`notify()`，导致错失的信号

### notify() notifyAll()
无论notify() notifyAll()，都只是唤醒等待同一个锁的线程。

### 生产者消费者
>![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2812342-677d34b35686b0bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 使用显式的Lock和Condition对象
类似于wait() notify()部分
```Java
public void run(){
    while(Thread.interrupted()){
        
        /* 已经不用synchronized关键词,某个方法中：
        lock.lock();
        try{
            flag_A = true;
            condition.signAll();//需要拥有这个锁
        } finally{
            lock.unlock();
        }
        */
        
        /* 已经不用synchronized关键词,某个方法中：
        lock.lock();
        try{
            while(flag_B == false) wait();
        } finally{
            lock.unlock();
        }
        */
    }
}
```

### 生产者消费者队列
wait() notify()方法很低级，用更高级抽象的`同步队列`来解决。
java.util.concurrent.BlockingQueue
LinkedBlockingQueue
ArrayBlockingQueue
消费者任务试图从队列里获取对象，如果此时为空，会挂起消费者，等到有元素时再恢复。
```Java
private BlockingQueue<类> rockets;
rockets = 某些队列
rockets.put(类的对象)
rockets.take()//取出来
```

### 任务间使用管道进行输入输出
管道输入，输出时候空会自动阻塞，
值得一提的是，不同于普通的I/O，管道是可中断的。
```Java
import java.io.IOException;
import java.io.PipedReader;
import java.io.PipedWriter;
import java.util.Random;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;
public class Multitask {
	public static void	main(String[] argus) {
		Sender sender = new Sender();
		Receiver receiver = new Receiver(sender);
		ExecutorService exec  = Executors.newCachedThreadPool();
		exec.execute(sender);
		exec.execute(receiver);
		try {
			TimeUnit.SECONDS.sleep(4);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		exec.shutdownNow();
		
	}
}
class Sender implements Runnable{
	private Random random = new Random(47);
	private PipedWriter out = new PipedWriter();
	public PipedWriter getPipeWriter(){return out;}
	@Override
	public void run() {
		try {
			while (true) {
				for (char c = 'A'; c <= 'Z'; c++) {
					out.write(c);
					TimeUnit.MILLISECONDS.sleep(random.nextInt(500));
				}
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
class Receiver implements Runnable{
	private PipedReader in;
	public Receiver(Sender sender) {
		try {
			in = new PipedReader(sender.getPipeWriter());
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
	@Override
	public void run() {
		try {
			while (true) {
				System.out.println("Read: "+(char)in.read()+",");
			}
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
}
```


## 死锁
书中对哲学家就餐问题进行了模拟。
同时指出死锁四个必要条件：

* 互斥条件。存在资源不能共享
* 占有某资源且等待另一资源
* 资源不能被抢占
* 存在循环等待。A等待B，B等待A

## 仿真
书中提到的几种对现实场景的仿真，如饭店服务、银行柜台服务等。这里不具体展开。
之前读过一篇经济学的文章，比较有意思，叫[《大糖帝国》](https://www.cchere.com/article/3816960)。是说在一个Sugarscape的棋盘上，一些人根据固定的生存法则自主的生存，最终一定会是贫富分化，满足二八定律。但是文章的结论同时指出，对于致富，“天赋论”和“出身决定一切”都不是决定性的。那么什么是决定性的？然而什么也不是，这个过程是复杂的。

有兴趣可以再进一步实现一下，假设每个单元(姑且这么称号，毕竟实际生活还有公司法人、自然人、乃至国家等各种经济体)就像《自私的基因》一样惟利是图（然而现实中有无数献身公益事业的人），而我们指定策略（如税收等财富再分配手段）来进行干预。但是这过程又需要每个单元进行智能的博弈。感觉会很有趣，留坑。

## 新类库中的构件
待阅读、实验。

JavaSE5的java.util.concurrent引入了很多好用的新类。

* CountDownLatch
* CyclicBarrier
* DelayQueue
* PriorityBlockingQueue
* ScheduledExecutor
* Semaphore
* Exchanger

## 性能调优
待阅读、实验。

介绍不同方式的性能比较。
Vector 和 Hashtable TreeSet 线程安全的免锁容器

## 活动对象
待阅读、实验。

多线程模型的替换方式。基于代理

## 总结及进阶读物

* Java Concurrency in Practice
* Concurrent Programming in Java
* The Java Language Specifacation 
