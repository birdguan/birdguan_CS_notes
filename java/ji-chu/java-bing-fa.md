# JAVA并发

## 一、使用线程 <a id="%E4%B8%80%E3%80%81%E4%BD%BF%E7%94%A8%E7%BA%BF%E7%A8%8B"></a>

有三种使用线程的方法：

* 实现Runnable接口；
* 实现Callable接口；
* 继承Thread类；

实现Runnable和Callable接口的类只能当作一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过Thread来调用。可以理解为任务是通过线程驱动从而执行的。

### 实现Runnable接口 <a id="%E5%AE%9E%E7%8E%B0Runnable%E6%8E%A5%E5%8F%A3"></a>

需要实现Runnable接口中的run\(\)方法，使用Runnable实例再创建一个Thread实例，然后调用Thread实例的start\(\)方法来启动线程。

```text
public class RunnableDemo {
    public static class MyRunnable implements Runnable {
        @Override
        public void run() {
            System.out.println("runnable");
        }
    }

    public static void main(String[] args) {
        MyRunnable instance = new MyRunnable();
        Thread thread = new Thread(instance);
        thread.start();
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 或者使用Lambda表达式：

```text
public class RunnableDemo {
    public static void main(String[] args) {
        Thread thread = new Thread(()->{
            System.out.println("lambda runnable");
        });
        thread.start();
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 实现Callable接口 <a id="%E5%AE%9E%E7%8E%B0Callable%E6%8E%A5%E5%8F%A3"></a>

与Runnable相比，Callable可以有返回值，返回值可以通过FutureTask进行封装。

```text
public class CallableDemo {
    public static class MyCallable implements Callable{

        @Override
        public Object call() throws Exception {
            System.out.println("callable");
            return "returnString";
        }
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        MyCallable callable = new MyCallable();
        FutureTask<String> futureTask = new FutureTask<>(callable);
        Thread thread = new Thread(futureTask);
        thread.start();
        System.out.println(futureTask.get());
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 继承Thread类

继承Thread类也需要实现run\(\)方法，因为Thread类也实现了Runable接口。

```text
public class MyThreadDemo {
    public static class Mythread extends Thread {
        @Override
        public void run() {
            System.out.println("my thread");
        }
    }

    public static void main(String[] args) {
        Mythread mythread = new Mythread();
        mythread.start();
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 实现接口VS继承Thread <a id="%E5%AE%9E%E7%8E%B0%E6%8E%A5%E5%8F%A3VS%E7%BB%A7%E6%89%BFThread"></a>

实现接口会更好一些，因为：

* Java不支持多重继承，因此继承了Thread类就无法继承其他类，但是可以实现多个接口；
* 类可能只要求可执行就行，继承整个Thread类开销过大。

## 二、基础线程机制 <a id="%E4%BA%8C%E3%80%81%E5%9F%BA%E7%A1%80%E7%BA%BF%E7%A8%8B%E6%9C%BA%E5%88%B6"></a>

### Executor <a id="Executor"></a>

Executor管理多个异步任务的执行，而无需程序员显式地管理线程的生命周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。

主要有三种Executor:

* CachedThreadPool：一个任务创建一个线程；
* FixedThreadPool：所有任务只能使用固定大小的线程；
* SingleThreadExecutor：相当于大小为1的FixedThreadPool。

```text
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Executor {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < 5; i++) {
            int finalI = i;
            executorService.execute(() -> {System.out.println("thread: " + finalI);});
        }
        executorService.shutdown();
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### Daemon <a id="Daemon"></a>

守护进程是程序运行时在后台提供服务的进程，不属于程序中不可或缺的部分。

当所有的非守护线程结束时，程序也就终止，同时会杀死所有的守护进程。

main\(\)属于非守护进程。

在线程启动之前使用setDaemon\(\)方法可以将一个线程设置为守护线程。

```text
public class DaemonDemo {
    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            while(true);
        });
        thread.setDaemon(true);
        thread.start();
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 因为thread被设置为Daemon线程，所有当main结束线程也就结束。

### sleep\(\) <a id="sleep()"></a>

Thread.sleep\(millisec\)方法会休眠当前正在执行的线程，millisec单位为毫秒。

sleep\(\)可能会抛出InterruptedException，因为异常不能跨线程传播回main\(\)中，因此必须在本地进行处理。线程中抛出的其它异常也同样要在本地进行处理。

```text
import java.text.SimpleDateFormat;
import java.util.Date;

public class SleepDemo {
    public static void main(String[] args) {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        System.out.println(dateFormat.format(new Date()));
        Thread thread = new Thread(() -> {
            try {
                Thread.sleep(2000);
                System.out.println(dateFormat.format(new Date()));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        thread.start();
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 结果：

```text
2020-03-10 10:02:29
2020-03-10 10:02:31
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### yield\(\) <a id="yield()"></a>

对静态方法Thread.yield\(\)的调用声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。该方法只是对线程调度器的一个建议，而且也只是建议具有相同优先级的其它线程可以运行。

```text
public class YieldDemo {
    public static void main(String[] args) {
        Thread thread1 = new Thread(() -> {
                Thread.yield();
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("thread1");
        });
        Thread thread2 = new Thread(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("thread2");
        });
        thread1.setDaemon(true);
        thread1.start();
        thread2.start();
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 三、中断 <a id="%E4%B8%89%E3%80%81%E4%B8%AD%E6%96%AD"></a>

一个线程执行完毕之后会自动结束，如果在运行过程中发生异常也会提前结束。

### InterruptedException <a id="InterruptedException"></a>

通过调用线程的interrupt\(\)可中断该线程，如果该线程处于阻塞、限期等待或者无期限等待状态，那么就会抛出InterruptedException，从而提前结束该线程。但是不能中断I/O阻塞和sychronized锁阻塞。

对于以下代码，在main\(\)中启动一个线程之后在中断它，由于线程中调用了Thread.sleep\(\)方法，因此会抛出一个InterruptedException，从而提前结束线程，不执行后面的语句。

```text
public class InterruptDemo {
    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            try {
                Thread.sleep(2000);
                System.out.println("thread");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        thread.start();
        thread.interrupt();
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### interrupted\(\) <a id="interrupted()"></a>

如果一个线程的run\(\)方法执行一个无限循环，并且没有执行sleep\(\)等会抛出InterruptedException的操作，那么调用线程的interrupt\(\)方法就无法使线程提前结束。

但是调用interrupt\(\)方法会设置线程的中断标记，此时调用interrupted方法会返回ture。因此可以在循环体中使用interrupted\(\)方法来判断线程是否处于中断状态，从而提前结束线程。

```text
import static java.lang.Thread.interrupted;

public class InterruptedDemo {
    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            while (! interrupted()) {
                System.out.println("while loop");
            }
            System.out.println("thread ended");
        });
        thread.start();
        thread.interrupt();
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### Executor的中断操作 <a id="Executor%E7%9A%84%E4%B8%AD%E6%96%AD%E6%93%8D%E4%BD%9C"></a>

调用Executor的shutdown\(\)方法会等待线程都执行完毕后再关闭，但是如果调用的是shutdownNow\(\)方法，相当于调用每个线程的interrupt\(\)方法。

```text
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ShutDownNow {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();
        executorService.execute(() -> {
            try {
                Thread.sleep(1000);
                System.out.println("thread ended");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        executorService.shutdownNow();
        System.out.println("main ended");
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 如果只想中断Executor中的一个线程，可以通过使用submit\(\)方法来提交一个线程，它会返回一个Future&lt;?&gt;对象，通过调用该对象的cancel\(true\)方法就可以中断线程。

```text
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class ShutDownNow {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();
        Future<?> future1 = executorService.submit(() -> {
            try {
                Thread.sleep(1000);
                System.out.println("thread1");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        executorService.execute(() -> {
            try {
                Thread.sleep(2000);
                System.out.println("thread2");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        future1.cancel(true);
        executorService.shutdown();
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```text
"C:\Program Files\Java\jdk-12.0.1\bin\java.exe" "-javaagent:C:\Program Files\JetBrains\IntelliJ IDEA Community Edition 2019.1.1\lib\idea_rt.jar=63625:C:\Program Files\JetBrains\IntelliJ IDEA Community Edition 2019.1.1\bin" -Dfile.encoding=UTF-8 -classpath D:\LearningFiles\Leetcode\out\production\Leetcode ShutDownNow
java.lang.InterruptedException: sleep interrupted
	at java.base/java.lang.Thread.sleep(Native Method)
	at ShutDownNow.lambda$main$0(ShutDownNow.java:10)
	at java.base/java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:515)
	at java.base/java.util.concurrent.FutureTask.run(FutureTask.java:264)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1128)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:628)
	at java.base/java.lang.Thread.run(Thread.java:835)
thread2

Process finished with exit code 0
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 四、互斥同步 <a id="%E5%9B%9B%E3%80%81%E4%BA%92%E6%96%A5%E5%90%8C%E6%AD%A5"></a>

Java提供了两种锁机制来控制多个线程对共享资源的互斥访问，第一个是JVM实现的sychronized，而另一个是JDK实现的ReentrantLock。

### synchronized <a id="synchronized"></a>

1.同步一个代码块

```text
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class SynchronizedDemo {
    public void func() {
        synchronized (this) {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        }
    }

    public static void main(String[] args) {
        SynchronizedDemo synchronizedDemo = new SynchronizedDemo();
        ExecutorService executorService = Executors.newCachedThreadPool();
        executorService.execute(() -> synchronizedDemo.func());
        executorService.execute(() -> synchronizedDemo.func());
        executorService.shutdown();
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```text
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

对于上面的代码，使用ExecutorService执行了两个线程，由于调用的是同一个对象的同步代码块，因此这两个线程会进行同步，当一个线程进入同步语句块时，另一个线程就必须等待。

但是其只作用于一个对象，如果调用两个对象上的同步代码块，就不会进行同步，例如：

```text
public static void main(String[] args) {
        SynchronizedDemo synchronizedDemo1 = new SynchronizedDemo();
        SynchronizedDemo synchronizedDemo2 = new SynchronizedDemo();
        ExecutorService executorService = Executors.newCachedThreadPool();
        executorService.execute(() -> synchronizedDemo1.func());
        executorService.execute(() -> synchronizedDemo2.func());
        executorService.shutdown();
    }
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```text
0 0 1 1 2 2 3 3 4 4 5 5 6 6 7 7 8 8 9 9 
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

2. 同步一个方法

```text
public synchronized void func() {
    // ...
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 3. 同步一个类

作用于整个类，也就是说两个线程调用同一个类的不同对象上的这种同步语句，也会进行同步。

```text
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class SynchronizedDemo {
    public void func() {
        synchronized (SynchronizedDemo.class) {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        }
    }

    public static void main(String[] args) {
        SynchronizedDemo synchronizedDemo1 = new SynchronizedDemo();
        SynchronizedDemo synchronizedDemo2 = new SynchronizedDemo();
        ExecutorService executorService = Executors.newCachedThreadPool();
        executorService.execute(() -> synchronizedDemo1.func());
        executorService.execute(() -> synchronizedDemo2.func());
        executorService.shutdown();
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```text
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

4. 同步一个静态方法

作用于整个类。

```text
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class SynchronizedDemo {
    public synchronized static void func() {
        for (int i = 0; i < 10; i++) {
            System.out.print(i + " ");
        }
    }

    public static void main(String[] args) {
        SynchronizedDemo synchronizedDemo1 = new SynchronizedDemo();
        SynchronizedDemo synchronizedDemo2 = new SynchronizedDemo();
        ExecutorService executorService = Executors.newCachedThreadPool();
        executorService.execute(() -> synchronizedDemo1.func());
        executorService.execute(() -> synchronizedDemo2.func());
        executorService.shutdown();
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### ReentrantLock <a id="ReentrantLock"></a>

ReentrantLock是java.util.concurrent（J.U.C）包中的锁。

```text
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockDemo {
    private Lock lock = new ReentrantLock();

    public void func() {
        lock.lock();
        try {
            for (int i = 0; i < 10; i++) {
                System.out.print(i + " ");
            }
        } finally {
            lock.unlock(); // 确保释放锁，从而避免发生死锁
        }
    }

    public static void main(String[] args) {
        ReentrantLockDemo reentrantLockDemo = new ReentrantLockDemo();
        ExecutorService executorService = Executors.newCachedThreadPool();
        executorService.execute(() -> reentrantLockDemo.func());
        executorService.execute(() -> reentrantLockDemo.func());
        executorService.shutdown();
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```text
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### 比较

**1.锁的实现**

synchronized是JVM实现的，而ReentrantLock是JDK实现的。

**2. 性能**

新版本Java对synchronized进行了很多优化，例如自旋锁等，synchonized与ReentrantLock大致相同。

**3. 等待可中断**

当持有锁的线程长期不释放锁时，正在等待的线程可以选择放弃等待，改为处理其他事情。

ReentrantLock可中断，而synchronized不行。

**4. 公平锁**

公平锁是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。

synchonized中的锁是非公平的，ReentrantLock默认情况下也是非公平的，但是也可以是公平的。

**5. 锁绑定多个条件**

一个ReentrantLock可以同时绑定多个Condition对象。

### 使用选择 <a id="%E4%BD%BF%E7%94%A8%E9%80%89%E6%8B%A9"></a>

除非需要使用ReentrantLock的高级特性，否则优先使用synchronized。这是因为synchronized是JVM实现的一种锁机制，JVM原生地支持它，而ReentrantLock不是所有的版本都支持。并且使用synchronized不用担心没有释放锁而导致死锁的问题，因为JVM会确保锁的释放。

## 五、线程之间的协作 <a id="%E4%BA%94%E3%80%81%E7%BA%BF%E7%A8%8B%E4%B9%8B%E9%97%B4%E7%9A%84%E5%8D%8F%E4%BD%9C"></a>

当多个线程可以一起工作去解决某个问题时，如果某些部分必须在其它部分之前完成，那么就需要对线程进行协调。

### join\(\) <a id="join()"></a>

在线程中调用另一个线程的join\(\)方法，会将当前线程挂起，而不是忙等待，直到目标线程结束。

```text
public class JoinExample {
    private class A extends Thread {
        @Override
        public void run() {
            System.out.println("A");
        }
    }

    private class B extends Thread {
        private A a;
        B(A a) {
            this.a = a;
        }
        @Override
        public void run() {
            try {
                a.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("B");
        }
    }

    public void test() {
        A a = new A();
        B b = new B(a);
        b.start();
        a.start();
    }

    public static void main(String[] args) {
        JoinExample joinExample = new JoinExample();
        joinExample.test();
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```text
A
B
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### wait\(\) notify\(\) notifyAll\(\)

调用wait\(\)使得线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使得这个条件满足时，其它线程会调用notify\(\)或者notifyAll\(\)来唤醒挂起的线程。

它们都属于Object的一部分，而不属于Thread。

只能在同步方法或者同步块中使用，否则会在运行时抛出IllegalMonitorStateException。

使用wait\(\)挂起期间，线程会释放锁。这是因为，如果没有释放锁，那么其它线程就无法进入对象的同步方法或同步块中，那么就无法执行notify\(\)或者notifyAll\(\)来唤醒挂起的线程，造成死锁。

```text
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class WaitNotifyExample {
    public synchronized void before () {
        System.out.println("before");
        notifyAll();
    }

    public synchronized void after() {
        try {
            wait();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("after");
    }

    public static void main(String[] args) {
        WaitNotifyExample waitNotifyExample = new WaitNotifyExample();
        ExecutorService executorService = Executors.newCachedThreadPool();
        executorService.execute(() -> waitNotifyExample.after());
        executorService.execute(() -> waitNotifyExample.before());
        executorService.shutdown();
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```text
before
after
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 wait\(\)和sleep\(\)的区别

* wait\(\)是Object的方法，而sleep\(\)是Thread的静态方法；
* wait\(\)会释放锁，sleep\(\)不会。

### await\(\) signal\(\) signalAll\(\) <a id="await()%20signal()%20signalAll()"></a>

java.util.concurrent类库中提供了Condition类来实现线程之间的协调，可以在Condition上调用await\(\)方法使线程等待，其它线程调用signal\(\)或signalAll\(\)方法唤醒等待的线程。

相比于wait\(\)这种等待方式，await\(\)可以指定等待的条件，因此更加灵活。

使用Lock来获取一个Condition对象：

```text
import java.awt.event.ActionListener;
import java.sql.Connection;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class AwaitSingalExample {
    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();

    public void before() {
        lock.lock();
        try {
            System.out.println("before");
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public void after() {
        lock.lock();
        try {
            condition.await();
            System.out.println("after");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
        AwaitSingalExample awaitSingalExample = new AwaitSingalExample();
        ExecutorService executorService = Executors.newCachedThreadPool();
        executorService.execute(() -> awaitSingalExample.after());
        executorService.execute(() -> awaitSingalExample.before());
        executorService.shutdown();
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```text
before
after
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

##  六、线程状态

一个线程只能处于一种状态，并且这里的线程状态特指Java虚拟机的线程状态，不能反映线程在特定操作系统下的状态。

### 新建（NEW） <a id="%E6%96%B0%E5%BB%BA%EF%BC%88NEW%EF%BC%89"></a>

创建后尚未启动。

### 可运行（RUNABLE） <a id="%E5%8F%AF%E8%BF%90%E8%A1%8C%EF%BC%88RUNABLE%EF%BC%89"></a>

正在Java虚拟机中运行。但是在操作系统层面，它可能处于运行状态，也可能等待资源调度（例如处理器资源），资源调度完成就进入运行状态。所以该状态的可运行是指可以被运行，具体有没有运行要看底层操作系统的资源调度。

### 阻塞（BLOCKED） <a id="%E9%98%BB%E5%A1%9E%EF%BC%88BLOCKED%EF%BC%89"></a>

请求获取monitor lock从而进入synchronized函数或者代码块，但是其它线程已经占用了该monitor lock，所有处于阻塞状态。要结束该状态进入RUNABLE需要其它线程释放monitor lock。

### 无期限等待（WAITING） <a id="%E6%97%A0%E6%9C%9F%E9%99%90%E7%AD%89%E5%BE%85%EF%BC%88WAITING%EF%BC%89"></a>

等待其它线程显式地唤醒。

阻塞和等待的区别在于，阻塞是被动的，它是在等待获取monitor lock。而等待是主动的，通过调用Object.wait\(\)等方法进入。

| 进入方法 | 退出方法 |
| :--- | :--- |
| 没有设置Timeout参数的Object.wait\(\)方法 | Object.notify\(\)/Object.notifyAll\(\) |
| 没有设置Timeout参数的Thread.join\(\)方法 | 被调用的线程执行完毕 |
| LockSupport.park\(\)方法 | LockSupport.unpark\(Thread\) |

### 期限等待（TIMED\_WAITING） <a id="%E6%9C%9F%E9%99%90%E7%AD%89%E5%BE%85%EF%BC%88TIMED_WAITING%EF%BC%89"></a>

无需等待其它线程显式唤醒，在一定时间之后会被系统自动唤醒。

| 进入方法 | 推出方法 |
| :--- | :--- |
| Thread.sleep\(\)方法 | 时间结束 |
| 设置了Timeout参数的Object.wait\(\) | 时间结束/Object.notify\(\)/Object.notifyAll\(\) |
| 设置了Timeout参数的Thread.join\(\)方法 | 时间结束/被调用的线程执行完毕 |
| LockSupport.parkNanos\(\)方法 | LockSupport.unpark\(Thread\) |
| LockSupport.parkUntil\(\)方法 | LockSupport.unpark\(Thread\) |

 调用Thread.sleep\(\)方法使线程进入期限等待状态时，常常用“使一个线程睡眠”进行描述。调用Object.wait\(\)方法使线程进入期限等待或者无期限等待时，常常用“挂起一个线程”进行描述。睡眠和挂起是用来描述行为，而阻塞和等待用来描述状态。

### 死亡（TERMINATED） <a id="%E6%AD%BB%E4%BA%A1%EF%BC%88TERMINATED%EF%BC%89"></a>

可以是线程结束任务后自己结束，或者产生了异常而结束。

## 七、J.U.C - AQS <a id="%E4%B8%83%E3%80%81J.U.C%20-%20AQS"></a>

java.util.concurrent\(J.U.C\)大大提高了并发性能，AQS被认为是J.U.C的核心。

**关于AQS**

AQS\(AbstractQuenedSynchronizer， 抽象队列式同步器\)，是除java自带的synchronized关键字之外的锁机制。

AQS的核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并将共享资源设置为锁定状态，如果被请求得共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时所分配的机制，这个机制使用CLH队列锁实现的，即将暂时获取不到锁的线程加入到队列中。

### CountDownLatch <a id="CountDownLatch"></a>

用来控制一个或者多个线程等待多个线程。

维护了一个计数器cnt，每次调用countDown\(\)方法会让计数器值减1，减到0的时候，那些因为调用await\(\)方法而在等待的线程就会被唤醒。

![](https://img-blog.csdnimg.cn/20200311120225259.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

```text
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class CountDownLatchExample {
    public static void main(String[] args) throws InterruptedException {
        final int totalThread = 10;
        CountDownLatch countDownLatch = new CountDownLatch(totalThread);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < totalThread; i++) {
            executorService.execute(() -> {
                System.out.print("run..");
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        System.out.println("end");
        executorService.shutdown();
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### CyclicBarrier <a id="CyclicBarrier"></a>

用来控制多个线程相互等待，只有当多个线程都到达时，这些线程才会继续执行。

和CountDownLatch相似，都是通过维护计数器来实现的。线程执行await\(\)方法之后计数器会减1，并进行等待，直到计数器为0，所有嗲用await\(\)方法而在等待的线程才能继续执行。

CyclicBarrier和CountDownLatch的一个区别是，CyclicBarrier的计数器通过调用reset\(\)方法可以循环使用，所以它才叫做循环屏障。

CyclicBarrier有两个构造函数，其中parties指示计数器的初始值，barrierAction在所有线程都到达屏障的时候会执行一次。

```text
public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}

public CyclicBarrier(int parties) {
    this(parties, null);
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

![](https://img-blog.csdnimg.cn/20200311120243520.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

```text
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class CyclicBarrierExample {
    public static void main(String[] args) {
        final int totalThread = 10;
        CyclicBarrier cyclicBarrier = new CyclicBarrier(totalThread);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < totalThread; i++) {
            executorService.execute(() -> {
                System.out.print("before...");
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
                System.out.print("after...");
            });
        }
        executorService.shutdown();
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```text
before...before...before...before...before...before...before...before...before...before...after...after...after...after...after...after...after...after...after...after...
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### Semaphore <a id="Semaphore"></a>

Semaphore类似于操作系统中的信号量，可以控制对互斥资源的访问线程数。

以下代码模拟了对某个服务的并发请求，每次只能有3个客户端同时访问，请求总数为10。

```text
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

public class SemaphoreExample {
    public static void main(String[] args) {
        final int clientLimit = 3;
        final int requestCount = 10;
        Semaphore semaphore = new Semaphore(clientLimit);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < requestCount; i++) {
            executorService.execute(() -> {
                try {
                    semaphore.acquire();
                    System.out.print(semaphore.availablePermits() + " ");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    semaphore.release();
                }
            });
        }
        executorService.shutdown();
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```text
2 1 0 1 1 1 1 1 2 1 
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 八、J.U.C - 其他组件 <a id="%E5%85%AB%E3%80%81J.U.C%20-%20%E5%85%B6%E4%BB%96%E7%BB%84%E4%BB%B6"></a>

### FutureTask <a id="FutureTask"></a>

FutureTask实现了RunnableFuture接口，该接口继承自Runnable和Future接口，这使得FutureTask既可以当作一个任务执行，也可以有返回值。

```text
public class FutureTask<V> implements RunnableFuture<V>
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```text
public interface RunnableFuture<V> extends Runnable, Future<V>
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 FutureTask可用于异步获取结果或取消任务的场景。当一个计算任务需要执行很长时间，那么就可以用FutureTask来封装这个任务，主线程在完成自己的任务之后再去获取结果。

```text
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

public class FutureTaskExample {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<Integer> futureTask = new FutureTask<Integer>(() -> {
            int result = 0;
            for (int i = 0; i < 100; i++) {
                Thread.sleep(10);
                result += i;
            }
            return result;
        });
        Thread computeThread = new Thread(futureTask);
        computeThread.start();
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(futureTask.get());
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### BlockingQueue <a id="BlockingQueue"></a>

java.util.concurrent.BlockingQueue接口有以下阻塞队列的实现：

* FIFO队列：LinkedBlockingQueue、ArrayBlockingQueue
* 优先级队列：PriorityBlockingQueue

提供了阻塞的take\(\)和put\(\)方法：如果队列为空take\(\)将阻塞，直到队列中有内容；如果队列满put\(\)将阻塞，直到队列有空闲位置。

**使用BlockingQueue实现生产者消费者问题**

```text
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.BlockingQueue;

public class ProducerConsumer {
    private static BlockingQueue<String> queue = new ArrayBlockingQueue<>(5);

    private static class Producer extends Thread {
        @Override
        public void run() {
            try {
                queue.put("product");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("produce..");
        }
    }

    private static class Consumer extends Thread {
        @Override
        public void run() {
            try {
                String result = queue.take();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("consumer..");
        }
    }

    public static void main(String[] args) {
        for (int i = 0; i < 2; i++) {
            Producer producer = new Producer();
            producer.start();
        }
        for (int i = 0; i < 5; i++) {
            Consumer consumer = new Consumer();
            consumer.start();
        }
        for (int i = 0; i < 3; i++) {
            Producer producer = new Producer();
            producer.start();
        }
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```text
produce..produce..consumer..consumer..produce..consumer..produce..consumer..produce..consumer..
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

### ForkJoin <a id="ForkJoin"></a>

主要用在并行计算中，和MapReduce原理类似，都是把大的计算任务拆分成多个小任务并行计算。

```text
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.Future;
import java.util.concurrent.RecursiveTask;

public class ForkJoinExample extends RecursiveTask<Integer> {
    private final int threshold = 5;
    private int first;
    private int last;

    public ForkJoinExample(int first, int last) {
        this.first = first;
        this.last = last;
    }

    @Override
    protected Integer compute() {
        int result = 0;
        if (last - first <= threshold) {
            // 任务足够小则直接计算
            for (int i = first; i <= last; i++) {
                result += i;
            }
        } else {
            // 拆分成小任务
            int middle = first + (last - first) / 2;
            ForkJoinExample leftTask = new ForkJoinExample(first, middle);
            ForkJoinExample rightTask = new ForkJoinExample(middle+1, last);
            leftTask.fork();
            rightTask.fork();
            result = leftTask.join() + rightTask.join();
        }
        return result;
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ForkJoinExample forkJoinExample = new ForkJoinExample(1, 10000);
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        Future<Integer> result = forkJoinPool.submit(forkJoinExample);
        System.out.println(result.get());
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

ForkJoin使用ForkJoinPool来启动，它是一个特殊的线程池，线程数量取决于CPU核数。 

## 九、线程不安全示例 <a id="%E4%B9%9D%E3%80%81%E7%BA%BF%E7%A8%8B%E4%B8%8D%E5%AE%89%E5%85%A8%E7%A4%BA%E4%BE%8B"></a>

如果多个线程对同一个共享数据进行访问而不采取同步操作的话，那么操作的结果是不一致的。

```text
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ThreadUnsageExample {
    private int cnt = 0;
    public void add() {
        cnt++;
    }
    public int get() {
        return cnt;
    }

    public static void main(String[] args) throws InterruptedException {
        final int threadSize = 1000;
        ThreadUnsageExample threadUnsageExample = new ThreadUnsageExample();
        CountDownLatch countDownLatch = new CountDownLatch(threadSize);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < threadSize; i++) {
            executorService.execute(() -> {
                threadUnsageExample.add();
                countDownLatch.countDown();
            });
        }
        countDownLatch.await();
        executorService.shutdown();
        System.out.println(threadUnsageExample.get());
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```text
980
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 十、Java内存模型 <a id="%E5%8D%81%E3%80%81Java%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B"></a>

Java内存模型试图屏蔽各种硬件和操作系统的内存访问差异，以实现让Java程序在各种平台下都能达到一致的访问效果。

### 主内存与工作内存 <a id="%E4%B8%BB%E5%86%85%E5%AD%98%E4%B8%8E%E5%B7%A5%E4%BD%9C%E5%86%85%E5%AD%98"></a>

处理器上的寄存器的读写速度比内存快几个数量级，为了解决这种速度矛盾，在它们之间加入了高速缓存。

加入高速缓存带来了一个新的问题：缓存一致性。如果多个缓存共享同一块主内存区域，那么多个缓存的数据可能会不一致，需要一些协议来解决这个问题。

![](https://img-blog.csdnimg.cn/20200311154801915.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

所有的变量都存储在主内存中，每个线程还有自己的工作内存，工作内存存储在高速缓存或者寄存器中，保存了该线程使用的变量的主内存副本拷贝。

线程只能直接操作工作内存中的变量，不同线程之间的变量值传递需要通过主内存来完成。

![](https://img-blog.csdnimg.cn/2020031115504185.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

### 内存间交互操作 <a id="%E5%86%85%E5%AD%98%E9%97%B4%E4%BA%A4%E4%BA%92%E6%93%8D%E4%BD%9C"></a>

Java内存模型定义了8个操作来完成主内存和工作内存的交互操作。

![](https://img-blog.csdnimg.cn/202003111551369.png)

* read：把一个变量的值从主内存传输到工作内存中
* load：在read之后执行，把read得到的值放入到工作内存的变量副本中
* use：把工作内存中的一个变量的值传递给执行引擎
* assign：把一个从执行引擎接收到的值赋给工作内存的变量
* store：把工作内存的一个变量的值传送到主内存中
* write：在store之后执行，把store得到的值放入到主内存变量中
* lock：作用于主内存变量
* unlock

### 内存模型三大特性 <a id="%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B%E4%B8%89%E5%A4%A7%E7%89%B9%E6%80%A7"></a>

**1. 原子性**

**2. 可见性**

可见性指当一个线程修改了共享变量的值，其它线程能够立即得知这个修改。Java内存模型是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值来实现可见性的。

主要有三种实现可见性的方式：

* volatile（不能保证原子性）
* synchronized，对一个变量执行unlock操作之前，必须把变量值同步回主内存
* final，被final关键字修饰的字段在构造器中一旦初始化完成，并且没有发生this逃逸（其它线程通过this引用访问到了初始化了一般的对象），那么其它线程就能看见final字段的值

3. 有序性

有序性是指：在本线程内观察，所有的操作都是有序的。在一个线程观察另一个线程，所有的操作都是无需的，无需是因为发生了指令重排序。

### 先行发生原则 <a id="%E5%85%88%E8%A1%8C%E5%8F%91%E7%94%9F%E5%8E%9F%E5%88%99"></a>

上面提到可以用volatile和synchronized来保证有序性。除此之外，JVM还规定了先行发生原则，让一个操作无需控制就能先于另一个操作完成。

1. 单一线程原则

在一个线程内，在程序前面的操作线性发生于后面的操作。

![](https://img-blog.csdnimg.cn/20200311161519126.png)

2. 管程锁定规则

一个unlock操作先行发生于后面对同一个锁的lock操作。

![](https://img-blog.csdnimg.cn/20200311163412392.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

3. volatile变量规则

对一个volatile变量的写操作先行发生于后面对这个变量的读操作。

![](https://img-blog.csdnimg.cn/20200311163500177.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

4. 线程启动规则

Thread对象的start\(\)方法调用先行发生于此线程的每一个动作。

![](https://img-blog.csdnimg.cn/20200311163553806.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

5. 线程加入规则

Thread对象的结束先行发生于join\(\)方法返回。

![](https://img-blog.csdnimg.cn/20200311163700126.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

6. 线程中断规则

对线程interrupt\(\)方法的调用先发生于被中断线程的代码检测到中断事件的发生可通过interrupted\(\)方法检测到是否有中断发生。

7. 对象终结规则

一个对象的初始化完成（构造函数执行结束）先行发生于它的finalize\(\)方法的开始。

8. 传递性

如果操作A先行发生于操作B，操作B先行发生于操作C，那么操作A先行发生于操作C。

## 十一、线程安全 <a id="%E5%8D%81%E4%B8%80%E3%80%81%E7%BA%BF%E7%A8%8B%E5%AE%89%E5%85%A8"></a>

多个线程不管以何种方式访问某个类，并且在主调代码中不需要进行同步，都能表现正确的行为。

线程安全有以下几种实现方式：

### 不可变 <a id="%E4%B8%8D%E5%8F%AF%E5%8F%98"></a>

不可变的对象一定是线程安全的，不需要再采取任何的线程安全保障措施。只要一个不可变的对象被正确地构建出来，永远也不会看到它在多个线程之中处于不一致的状态。多线程环境下，应当尽量使对象称为不可变，来满足线程安全。

不可变的类型：

* final关键字修饰的基本数据类型
* String
* 枚举类型
* Number部分子类，如Long和Double等数值包装类型，BigInteger和BigDecimal等大数据类型。但同为Number的原子类AtomicInteger和AtomicLong则是可变的。

对于集合类型，可以使用Collections.unmodifiableXXX\(\)方法来获取一个不可变的集合。

### 互斥同步 <a id="%E4%BA%92%E6%96%A5%E5%90%8C%E6%AD%A5"></a>

sychronized和ReetrantLock

### 非阻塞同步 <a id="%E9%9D%9E%E9%98%BB%E5%A1%9E%E5%90%8C%E6%AD%A5"></a>

互斥同步最主要的问题就是线程阻塞和唤醒所带来的性能问题，因此这种同步也称为阻塞同步。

互斥同步属于一种悲观的并发策略，总是认为只要不去做正确的同步措施，那就肯定会出现问题。无论共享数据是否真的会出现竞争，它都要进行加锁（这里讨论的是概念模型，实际虚拟机会优化掉很大一部分不必要的加锁）、用户态核心态转换、维护锁计数器和检查是否有被阻塞的线程需要唤醒等操作。

随着硬件指令集的发展，我们可以使用基于冲突检测的乐观并发策略：先进行操作，如果没有其他线程争用共享数据，那操作就成功了，，否则采取补偿措施（不断地重试，直到成功为止）。这种乐观的并发策略的许多实现都不需要将线程阻塞，因此这种同步方式称为非阻塞同步。

1. CAS

乐观锁需要操作和冲突检测这两个步骤具备原子性，这里就不能再使用互斥同步来保证了，只能靠硬件来完成。硬件支持的原子性操作最典型的是：比较并交换（Compare-and-Swap, CAS）。CAS指令需要有3个操作数，分别是内存地址V、旧的预期值A和新值B。当执行操作时，只有当V的值等于A，才将V的值更新为B。

2. AtomicInteger

J.U.C包里面的整数原子类AtomicInteger的方法调用了Unsafe类的CAS操作。

3. ABA

如果一个变量初次读取的时候是A值，它的值被改成了B，后来又被改回成A，那CAS操作就会误认为它从来没有被改变过。

J.U.C包提供了一个带有标记的原子引用类AtomicStampedReference来解决这个问题，它可以用过控制变量值的版本来保证CAS的正确性。大部分情况下ABA问题不会影响程序并发的正确性，如果需要解决ABA问题，改用传统的互斥同步可能会比原子类更高效。

### 无同步方案 <a id="%E6%97%A0%E5%90%8C%E6%AD%A5%E6%96%B9%E6%A1%88"></a>

要保证线程安全，并不一定就要进行同步。如果一个方法本来就不涉及共享数据，那它自然就无需任何同步措施去保证正确性。

1. 栈封闭

多个线程访问同一个方法的局部变量时，不会出现线程安全问题，因为局部变量存储在虚拟机栈中，属于线程私有的。

2. 线程本地存储

如果一段代码中所需要的数据必须与其它代码共享，那就看看这些共享数据的代码是否能保证子同一个线程中执行。如果能保证，我们就可以把共享数据的可见范围限制在同一个线程之内，这样，无需同步也能保证线程之间不出现数据争用的问题。

符合这种特点的应用并不少见，大部分使用消息队列的架构模式（如“生产者-消费者”模式）都会将产品的消费过程尽量在一个线程中消费完。其中最重要的一个应用实例就是经典Web交互模型中的“一个请求对应一个服务器线程”（Thread-per-Request）的处理方式，这种处理方式的广泛应用使得很多Web服务端应用都可以使用线程本地存储来解决线程安全问题。

可以使用java.lang.ThreadLocal类来实现线程本地存储功能。

对于以下代码，thread1中设置threadLocal为1，而thread2设置threadLocal为2。过了一段时间之后，thread1读取threadLocal依然是1，不受thread2的影响。

```text
public class ThreadLocalExample {
    public static void main(String[] args) {
        ThreadLocal<String> threadLocal = new ThreadLocal<>();
        Thread thread1 = new Thread(() -> {
            threadLocal.set("thread1");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(threadLocal.get());
            threadLocal.remove();
        });
        Thread thread2 = new Thread(() -> {
            threadLocal.set("thread2");
            threadLocal.remove();
        });
        thread1.start();
        thread2.start();
    }
}
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```text
thread1
```

![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 每个Thread都有一个ThreadLocal.ThreadLocalMap对象。

![](https://img-blog.csdnimg.cn/2020031118464976.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMwMjQxNzA5,size_16,color_FFFFFF,t_70)

3. 可重入代码

这种代码也叫做纯代码，可以在代码执行的任何时刻中断它，转而去执行另外一段代码，而在控制权返回后，原来的程序不会出现任何错误。

可重入代码有一些共同的特征，例如不依赖存储在堆上的数据和公用的系统资源、用到的状态量都由参数中传入、不可调用非重入的方法等。

## 十二、锁优化 <a id="%E5%8D%81%E4%BA%8C%E3%80%81%E9%94%81%E4%BC%98%E5%8C%96"></a>

这里的锁优化主要是指JVM对synchronized的优化。

### 自旋锁 <a id="%E8%87%AA%E6%97%8B%E9%94%81"></a>

互斥同步进入阻塞状态的开销都很大，应该尽量避免。在许多应用中，共享数据的锁定状态只会持续很短的一段时间。自旋锁的思想是想让一个线程在请求一个共享数据的锁时执行忙循环（自旋）一段时间，如果在这段时间内能获得锁，就可以避免进入阻塞状态。

自旋锁虽然能避免进入拥塞状态从而减小开销，但是它需要进行忙操作占用CPU时间，它只适用于共享数据的锁定状态很短的场景。

在JDK1.6中引入了自适应的自旋锁。自适应意味着自旋锁的次数不再固定了，而是由前一次在同一个锁上的自旋次数及锁的拥有者的状态来决定。

### 锁消除 <a id="%E9%94%81%E6%B6%88%E9%99%A4"></a>

锁消除是指对于检测出来不可能存在竞争的共享数据的锁进行消除。

锁消除主要是通过逃逸分析来支持，如果堆上的共享数据不可能逃逸出去被其他线程访问到，那么就可以把它们当成私有数据对待，也就可以将它们的所进行消除。

### 锁粗化 <a id="%E9%94%81%E7%B2%97%E5%8C%96"></a>

如果一系列的连续操作都对同一个对象反复加锁和解锁，频繁的加锁操作就会导致性能损耗。

如果虚拟机探测到由这样的一串零碎的操作都是对一个对象加锁，就会把加锁的范围扩展（粗化）到整个操作序列的外部。

### 轻量级锁 <a id="%E8%BD%BB%E9%87%8F%E7%BA%A7%E9%94%81"></a>

JDK1.6引入了偏向锁和轻量级锁，从而让锁拥有了四个状态：无琐状态（unlocked）、偏向锁状态（biasble）、轻量级锁状态（lightweight locked）和重量级锁状态（inflated）。

轻量级锁是相对于传统的重量级锁而言，它使用CAS操作来避免重量级锁使用互斥量的开销。对于绝大部分的锁，在整个同步周期内都是不存在竞争的，因此也就不需要都是用互斥量进行同步，可以先采用CAS操作进行同步，如果CAS失败了再改用互斥量进行同步。

### 偏向锁 <a id="%E5%81%8F%E5%90%91%E9%94%81"></a>

偏向锁的思想是偏向于让第一个获取锁对象的相乘获取锁，这个线程在之后获取该锁就不再需要进行同步操作，甚至连CAS操作也不再需要，甚至连CAS操作也不再需要。

## 十三、多线程开发良好的实践 <a id="%E5%8D%81%E4%B8%89%E3%80%81%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%BC%80%E5%8F%91%E8%89%AF%E5%A5%BD%E7%9A%84%E5%AE%9E%E8%B7%B5"></a>

* 给线程取个有意义的名字，这样可以方便找Bug。
* 缩小同步范围，从而减少锁征用。例如对于synchronized，应该尽量使用同步块而不是同步方法。
* 多用同步工具少用wait\(\)和notify\(\)。首先，CountDownLatch，CyclicBarrier, Semaphore和Exchanger这些同步类简化了编码操作，而用wait\(\)和notify\(\)很难实现复杂控制流；其次，这些同步类是由最好的qi'yqiy阿编写和维护的，在后续的JDK中还会不断优化和完善。
* 使用BlockingQueue实现生产者消费者问题。
* 多用并发集合少用同步集合，例如应该使用ConcurrentHashMap而不是HashTable。
* 使用本地变量和不可变类来保证线程安全。
* 使用线程池而不是直接创建线程，这是因为创建线程代价很高，线程池可以有效地利用有限地线程来启动任务。

