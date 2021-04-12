---
title: callable与future
date: 2019-07-06 18:39:05
categories: 并发编程
tags: [多线程,并发]
---

# callable与future

​		在Java中，创建线程一般有两种方式，一种是继承Thread类，一种是实现Runnable接口。然而，这两种方式的缺点是在线程任务执行结束后，无法获取执行结果。我们一般只能采用共享变量或共享存储区以及线程通信的方式实现获得任务结果的目的。
不过，Java中，也提供了使用Callable和Future来实现获取任务结果的操作。Callable用来执行任务，产生结果，而Future用来获得结果。

<!--more-->

``` java
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}

```

以上Callable的源码，接口带有泛型的返回值。

## Future中的常用方法

1. **V get()** **：**获取异步执行的结果，如果没有结果可用，此方法会阻塞直到异步计算完成。
2. **V get(Long timeout , TimeUnitunit)** ：获取异步执行结果，如果没有结果可用，此方法会阻塞，但是会有时间限制，如果阻塞时间超过设定的timeout时间，该方法将抛出异常。
3. **boolean isDone()** **：**如果任务执行结束，无论是正常结束或是中途取消还是发生异常，都返回true。
4. **boolean isCanceller()** **：**如果任务完成前被取消，则返回true。
5. **boolean cancel(booleanmayInterruptRunning)** **：**如果任务还没开始，执行cancel(...)方法将返回false；如果任务已经启动，执行cancel(true)方法将以中断执行此任务线程的方式来试图停止任务，如果停止成功，返回true；当任务已经启动，执行cancel(false)方法将不会对正在执行的任务线程产生影响(让线程正常执行到完成)，此时返回false；当任务已经完成，执行cancel(...)方法将返回false。mayInterruptRunning参数表示是否中断执行中的线程。

## demo

``` java
public class TestMain {
	public static void main(String[] args) throws InterruptedException, ExecutionException {
		ExecutorService executor = Executors.newCachedThreadPool();
		Future<Integer> future = executor.submit(new AddNumberTask());
		System.out.println(Thread.currentThread().getName() + "线程执行其他任务");
		Integer integer = future.get(); // 调用get后会阻塞，直到future拿到返回的数据
		System.out.println(integer);
		// 关闭线程池
		if (executor != null)
			executor.shutdown();
	}

}

class AddNumberTask implements Callable<Integer> {

	public AddNumberTask() {

	}

	@Override
	public Integer call() throws Exception {
		System.out.println("####AddNumberTask###call()");
		Thread.sleep(5000);
		return 5000;
	}

}

```

## Future模式

Future模式的核心在于：去除了主函数的等待时间，并使得原本需要等待的时间段可以用于处理其他业务逻辑

Futrure模式:对于多线程，如果线程A要等待线程B的结果，那么线程A没必要等待B，直到B有结果，可以先拿到一个未来的Future，等B有结果是再取真实的结果。

原理：本质上是notify与wait的结合使用！附上原理解析代码

​	Data接口

``` java
public interface Data {
    public abstract String getRequest();
}
```

``` java
public class FurureData implements Data {

    public volatile static boolean ISFLAG = false;
    private RealData realData;

    public synchronized void setRealData(RealData realData) {
        // 如果已经获取到结果，直接返回
        if (ISFLAG) {
            return;
        }
        // 如果没有获取到数据,传递真是对象
        this.realData = realData;
        ISFLAG = true;
        // 进行通知
        notify();
    }

    @Override
    public synchronized String getRequest() {
        while (!ISFLAG) {
            try {
                wait();
            } catch (Exception e) {

            }
        }
        // 获取到数据,直接返回
        return realData.getRequest();
    }

}

```

``` java
public class RealData implements Data {
    private String result;

    public RealData(String data) {
        System.out.println("正在使用data:" + data + "网络请求数据,耗时操作需要等待.");
        try {
            Thread.sleep(3000);
        } catch (Exception e) {

        }
        System.out.println("操作完毕,获取结果...");
        result = data;
    }

    @Override
    public String getRequest() {
        return result;
    }
}
```

``` java
public class FutureClient {

    public Data request(final String queryStr) {
        final FurureData furureData = new FurureData();
        new Thread(new Runnable() {

            @Override
            public void run() {
                RealData realData = new RealData(queryStr);
                furureData.setRealData(realData); // set完成时唤醒
            }
        }).start();
        return furureData;

    }

}
```

``` java
public class Main {
    public static void main(String[] args) {
        FutureClient futureClient = new FutureClient();
        Data request = futureClient.request("请求参数.");
        System.out.println("请求发送成功!");
        System.out.println("执行其他任务...");
        String result = request.getRequest(); // 此处如果set未设置上值，会阻塞
        System.out.println("获取到结果..." + result);
    }

}
```

## 运行效果

![](1.PNG)

![](2.PNG)

