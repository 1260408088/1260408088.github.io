---
title: 并发编程-java内存模型
date: 2019-06-13 18:24:45
categories: 并发编程
tags: [java内存模型，多线程]
---
# 线程安全

​	当多个线程同时共享，同一个**全局变量或静态变量**，做写的操作时，可能会发生数据冲突问题，也就是线程安全问题。但是做读操作是不会发生数据冲突问题。

<!--more-->

最经典的买票的问题：

``` java
public class Ticket implements Runnable {
    int total = 100;

    @Override
    public void run() {
        while (total > 0) {
            try {
                Thread.sleep(5);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            sale();
        }
    }

    private synchronized void sale() { // 不加synchronized，就会有一张票多次被卖的问题
        if (total > 0) {
            System.out.println(Thread.currentThread().getName() + ",出售第" + (100 - total + 1) + "张票");
            total--;
        }
    }

    public static void main(String[] args) {
        Ticket T = new Ticket();
        Thread thread1 = new Thread(T,"one");
        Thread thread2 = new Thread(T,"two");
        Thread thread3 = new Thread(T,"there");
        thread1.start();
        thread2.start();
        thread3.start();
    }
}
```

# synchronized关键字

内置锁使用synchronized关键字实现，synchronized关键字有两种用法：

1. 修饰需要进行同步的方法（所有访问状态变量的方法都必须进行同步），此时充当锁的对象为调用同步方法的对象(默认this)

2. 同步代码块和直接使用synchronized修饰需要同步的方法是一样的，但是锁的粒度可以更细，并且充当锁的对象不一定是this，也可以是其它对象，所以使用起来更加灵活

``` java
synchronized(同一个数据){
 可能会发生线程冲突问题
}
就是同步代码块 
synchronized(对象) //这个对象可以为任意对象 
{ 
 需要被同步的代码 
} 

```

同步的前提： 

1. 必须要有两个或者两个以上的线程 

2. 必须是多个线程使用同一个锁 

必须保证同步中只能有一个线程在运行 

好处：解决了多线程的安全问题 

弊端：多个线程需要判断锁，较为消耗资源、抢锁的资源。

## 同步方法

``` java
public synchronized void sale() {
		if (trainCount > 0) {
			System.out.println(Thread.currentThread().getName() + ",出售第" + (100 - trainCount + 1) + "张票");
			trainCount--;
		}
	}
```

## 静态同步函数

方法上加上static关键字，使用synchronized 关键字修饰 或者使用类.class文件。静态的同步函数使用的锁是  该函数所属字节码文件对象，可以用 getClass方法获取，也可以用当前  类名.class 表示。

``` java
public static void sale() {
		synchronized (ThreadTrain3.class) {
			if (trainCount > 0) {
				System.out.println(Thread.currentThread().getName() + ",出售第" + (100 - trainCount + 1) + "张票");
				trainCount--;
			}
		}
}
```

## tip

synchronized 修饰方法使用锁是当前this锁。

synchronized 修饰静态方法使用锁是当前类的字节码文件。

# 线程死锁

​	同步中嵌套同步,导致锁无法释放

``` java
public class Deadlock implements Runnable {
    private int trainCount = 100;
    private Object oj = new Object();
    public boolean flag = true;

    public void run() {

        if (flag) {
            while (trainCount > 0) {
                synchronized (oj) {
                    try {
                        Thread.sleep(10);
                    } catch (Exception e) {
                        // TODO: handle exception
                    }
                    sale();
                }

            }
        } else {
            while (trainCount > 0) {
                sale();
            }

        }

    }

    public synchronized void sale() {
        synchronized (oj) {
            try {
                Thread.sleep(10);
            } catch (Exception e) {

            }
            if (trainCount > 0) {
                System.out.println(Thread.currentThread().getName() + "," + "出售第" + (100 - trainCount + 1) + "票");
                trainCount--;
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Deadlock threadTrain = new Deadlock();
        Thread t1 = new Thread(threadTrain, "窗口1");
        Thread t2 = new Thread(threadTrain, "窗口2");
        t1.start();
        Thread.sleep(40);
        threadTrain.flag = false;
        t2.start();
    }

}
```

# Threadlocal

​		当使用ThreadLocal维护变量时，ThreadLocal为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。

ThreadLocal的接口方法

ThreadLocal类接口很简单，只有4个方法：

- void set(Object value)设置当前线程的线程局部变量的值。

- public Object get()该方法返回当前线程所对应的线程局部变量。

- public void remove()将当前线程局部变量的值删除，目的是为了减少内存的占用，该方法是JDK 5.0新增的方法。需要指出的是，当线程结束后，对应该线程的局部变量将自动被垃圾回收，所以显式调用该方法清除线程的局部变量并不是必须的操作，但它可以加快内存回收的速度。

- protected Object initialValue()返回该线程局部变量的初始值，该方法是一个protected的方法，显然是为了让子类覆盖而设计的。这个方法是一个延迟调用方法，在线程第1次调用get()或set(Object)时才执行，并且仅执行1次。ThreadLocal中的缺省实现直接返回一个null。

## demo

创建三个线程，每个线程生成自己独立序列号

``` java
class Res {
    // 生成序列号共享变量
    public static Integer count = 0;
    public static ThreadLocal<Integer> threadLocal = new ThreadLocal<Integer>() {
        protected Integer initialValue() {
            return 0;
        }
    };

    public Integer getNum() {
        int count = threadLocal.get() + 1;
        threadLocal.set(count);
        return count;
    }
}

public class ThreadlocalTest extends Thread {
    private Res res;

    public ThreadlocalTest(Res res) {
        this.res = res;
    }

    @Override
    public void run() {
        for (int i = 0; i < 3; i++) {
            System.out.println(Thread.currentThread().getName() + "---" + "i---" + i + "--num:" + res.getNum());
        }
    }

    public static void main(String[] args) {
        Res res = new Res();
        ThreadlocalTest threadLocaDemo1 = new ThreadlocalTest(res);
        ThreadlocalTest threadLocaDemo2 = new ThreadlocalTest(res);
        ThreadlocalTest threadLocaDemo3 = new ThreadlocalTest(res);
        threadLocaDemo1.start();
        threadLocaDemo2.start();
        threadLocaDemo3.start();
    }
}
```

## ThreadLoca实现原理

ThreadLoca 通过map集合。Map.put(“当前线程”,值)；

# 线程的三大特性

## 原子性

即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。

一个很经典的例子就是银行账户转账问题： 
 比如从账户A向账户B转1000元，那么必然包括2个操作：从账户A减去1000元，往账户B加上1000元。这2个操作必须要具备原子性才能保证不出现一些意外的问题。

我们操作数据也是如此，比如i = i+1；其中就包括，读取i的值，计算i，写入i。这行代码在[Java](http://lib.csdn.net/base/java)中是不具备原子性的，则多线程运行肯定会出问题，所以也需要我们使用同步和lock这些东西来确保这个特性了。 原子性其实就是保证数据一致、线程安全一部分，

## 可见性

当多个线程访问同一个变量时，一个线程修改了这个变量的值，其他线程能够立即看得到修改的值。若两个线程在不同的cpu，那么线程1改变了i的值还没刷新到主内存，线程2又使用了i，那么这个i值肯定还是之前的，线程1对变量的修改而线程2没看到，这就是可见性问题。

## 有序性

程序执行的顺序按照代码的先后顺序执行。一般来说处理器为了提高程序运行效率，可能会对输入代码进行优化，它不保证程序中各个语句的执行先后顺序同代码中的顺序一致，<span style="color:red">但是它会保证程序最终执行结果和代码顺序执行的结果是一致的。</span>如下：

``` java
int a = 10;   //语句1
int r = 2;    //语句2
a = a + 3;    //语句3
r = a*a;      //语句4
```

则因为重排序，他还可能执行顺序为 2-1-3-4，1-3-2-4 但绝不可能 2-1-4-3，因为这打破了依赖关系。 显然重排序对单线程运行是不会有任何问题，而多线程就不一定了，所以我们在多线程编程时就得考虑这个问题了。

# java内存模型

​		共享内存模型指的就是Java内存模型(简称JMM)，**JMM** 决定一个线程对共享变量的写入时,能对另一个线程可见。从抽象的角度来看，JMM定义了线程和主内存之间的抽象关系：**线程之间的共享变量存储在主内存（main memory）中，每个线程都有一个私有的本地内存（local memory），本地内存中存储了该线程以读/写共享变量的副本**。本地内存是JMM的一个抽象概念，并不真实存在。它涵盖了缓存，写缓冲区，寄存器以及其他的硬件和编译器优化。

![JMM](JMM.png)                           

从上图来看，线程A与线程B之间如要通信的话，必须要经历下面2个步骤：

1. 首先，线程A把本地内存A中更新过的共享变量刷新到主内存中去。

2. 然后，线程B到主内存中去读取线程A之前已更新过的共享变量。 

下面通过示意图来说明这两个步骤： 
    ![步骤](guocheng.png)

如上图所示，本地内存A和B有主内存中共享变量x的副本。假设初始时，这三个内存中的x值都为0。线程A在执行时，把更新后的x值（假设值为1）临时存放在自己的本地内存A中。当线程A和线程B需要通信时，线程A首先会把自己本地内存中修改后的x值刷新到主内存中，此时主内存中的x值变为了1。随后，线程B到主内存中去读取线程A更新后的x值，此时线程B的本地内存的x值也变为了1。

从整体来看，这两个步骤实质上是线程A在向线程B发送消息，而且这个通信过程必须要经过主内存。JMM通过控制主内存与每个线程的本地内存之间的交互，来为java程序员提供内存可见性保证。

总结：什么是Java内存模型：java内存模型简称jmm，定义了一个线程对另一个线程可见。共享变量存放在主内存中，每个线程都有自己的本地内存，当多个线程同时访问一个数据的时候，可能本地内存没有及时刷新到主内存，所以就会发生线程安全问题。

# Volatile关键字

​		可见性也就是说一旦某个线程修改了该被volatile修饰的变量，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，可以立即获取修改之后的值。在Java中为了加快程序的运行效率，对一些变量的操作通常是在该线程的寄存器或是CPU缓存上进行的，之后才会同步到主存中，而加了volatile修饰符的变量则是直接读写主存。Volatile 保证了线程间共享变量的及时可见性，但不能保证原子性

``` java
class ThreadVolatileDemo extends Thread {
	public volatile boolean flag = true; // 一个线程给设置为false后，另一个线程会可见的
	@Override
	public void run() {
		System.out.println("开始执行子线程....");
		while (flag) {
		}
		System.out.println("线程停止");
	}
	public void setRuning(boolean flag) {
		this.flag = flag;
	}

}

public class ThreadVolatile {
	public static void main(String[] args) throws InterruptedException {
		ThreadVolatileDemo threadVolatileDemo = new ThreadVolatileDemo();
		threadVolatileDemo.start();
		Thread.sleep(3000);
		threadVolatileDemo.setRuning(false);
		System.out.println("flag 已经设置成false");
		Thread.sleep(1000);
		System.out.println(threadVolatileDemo.flag);
	}
}

```

## 特性

1. 保证此变量对所有的线程的可见性，这里的“可见性”，如本文开头所述，当一个线程修改了这个变量的值，volatile 保证了新值能立即同步到主内存，以及每次使用前立即从主内存刷新。但普通变量做不到这点，普通变量的值在线程间传递均需要通过主内存（详见：Java内存模型）来完成。

2. <span style="color:red">禁止指令重排序优化。</span>(就是编译器自己优化代码顺序，结果保持不变)有volatile修饰的变量，赋值后多执行了一个“load addl $0x0, (%esp)”操作，这个操作相当于一个内存屏障（指令重排序时不能把后面的指令重排序到内存屏障之前的位置），只有一个CPU访问内存时，并不需要内存屏障；（什么是指令重排序：是指CPU采用了允许将多条指令不按程序规定的顺序分开发送给各相应电路单元处理）。

 volatile 性能：

　　volatile 的读性能消耗与普通变量几乎相同，但是写操作稍慢，因为它需要在本地代码中插入许多内存屏障指令来保证处理器不发生乱序执行。

## volatile与synchronized区别

1. 从而我们可以看出volatile虽然具有可见性但是并不能保证原子性。

2. 性能方面，synchronized关键字是防止多个线程同时执行一段代码，就会影响程序执行效率，而volatile关键字在某些情况下性能要优于synchronized。

但是要注意volatile关键字是无法替代synchronized关键字的，因为volatile关键字无法保证操作的原子性。（单独使用 volatile 还不足以实现计数器），volatile变量不会像锁那样造成[线程阻塞](https://baike.baidu.com/item/线程阻塞)，（同步方法一次只能一个线程访问）因此也很少造成可伸缩性问题。在某些情况下，如果读操作远远大于写操作，volatile变量还可以提供优于锁的性能优势。

