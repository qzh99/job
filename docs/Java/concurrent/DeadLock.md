# 死锁

## 1、描述

```
两个或以上线程，因为竞争资源而产生互相等待的现象。

比如threa1持有lock1，等待lock2；thread2持有lock2，等待lock1；那么这两个线程会陷入死锁状态。
```

## 2、示例代码

### 2.1、示例一

```java
package me.qzh.thread;

import java.util.concurrent.TimeUnit;

/**
 * @author qinzhenghua
 * @version 1.5
 * @since 2021/3/12 23:13
 */
public class DeadLockTest {
    public static void main(String[] args) {
        Object obj1 = new Object();
        Object obj2 = new Object();

        Runnable r1 = () -> {
            synchronized (obj1) {
                try {
                    TimeUnit.SECONDS.sleep(3);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (obj2){
                    System.out.println("thread1");
                    System.out.println(Thread.currentThread());
                }
            }
        };

        Runnable r2 = () -> {
            synchronized (obj2) {
                try {
                    Thread.sleep(3 * 1000L);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (obj1){
                    System.out.println("thread2");
                    System.out.println(Thread.currentThread());
                }
            }
        };

        Thread t1 = new Thread(r1);
        Thread t2 = new Thread(r2);

        t1.start();
        t2.start();
    }
}
```

### 2.2、示例二

```java
package me.qzh.thread;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @author qinzhenghua
 * @version 1.5
 * @since 2021/3/13 1:04
 */
public class DeadLockTest2 {
    public static void main(String[] args) {
        Lock lock1 = new ReentrantLock();
        Lock lock2 = new ReentrantLock();

        Runnable  r1 = new Runnable1(lock1,lock2);
        Runnable  r2 = new Runnable2(lock1,lock2);
        Thread t1 = new Thread(r1);
        Thread t2 = new Thread(r2);
        t1.start();
        t2.start();

    }
}

class Runnable1 implements Runnable{
    Lock lock1 = null;
    Lock lock2 = null;

    public Runnable1(Lock lock1, Lock lock2){
        this.lock1 = lock1;
        this.lock2 = lock2;
    }

    @Override
    public void run() {
        lock1.lock();
        System.out.println(Thread.currentThread().getName() + " lock1加锁");

        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        lock2.lock();
        System.out.println(Thread.currentThread().getName() + " lock2加锁");
        // do work

        lock1.unlock();
        lock2.unlock();
    }
}

class Runnable2 implements Runnable{
    Lock lock1 = null;
    Lock lock2 = null;

    public Runnable2(Lock lock1, Lock lock2){
        this.lock1 = lock1;
        this.lock2 = lock2;
    }

    @Override
    public void run() {
        lock2.lock();
        System.out.println(Thread.currentThread().getName() + " lock2加锁");

        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        lock1.lock();
        System.out.println(Thread.currentThread().getName() + " lock1加锁");
        // do work

        lock2.unlock();
        lock1.unlock();
    }
}
```

## 3、检测死锁

> 参考：https://www.cnblogs.com/flyingeagle/articles/6853167.html

### 3.1、获取Java进程

```
jps
或
ps -ef | grep java
```

### 3.2、检测死锁

```
jstack -l pid
```

### 3.3、运行示例1

```
Microsoft Windows [版本 10.0.19042.867]
(c) 2020 Microsoft Corporation. 保留所有权利。

C:\Program Files\Java\jdk1.8.0_201\bin>jps
10232
11208 Jps
14268 DeadLockTest
636 Launcher

C:\Program Files\Java\jdk1.8.0_201\bin>jstack -l 14268
2021-03-13 00:41:44
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.201-b09 mixed mode):

"DestroyJavaVM" #14 prio=5 os_prio=0 tid=0x0000000003533000 nid=0x3b80 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"Thread-1" #13 prio=5 os_prio=0 tid=0x000000001ccd1000 nid=0x3948 waiting for monitor entry [0x000000001da4f000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at me.qzh.thread.DeadLockTest.lambda$main$1(DeadLockTest.java:37)
        - waiting to lock <0x0000000780a5d000> (a java.lang.Object)
        - locked <0x0000000780a5d010> (a java.lang.Object)
        at me.qzh.thread.DeadLockTest$$Lambda$2/1747585824.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)

   Locked ownable synchronizers:
        - None

"Thread-0" #12 prio=5 os_prio=0 tid=0x000000001ccd0800 nid=0x2e4c waiting for monitor entry [0x000000001d94f000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at me.qzh.thread.DeadLockTest.lambda$main$0(DeadLockTest.java:23)
        - waiting to lock <0x0000000780a5d010> (a java.lang.Object)
        - locked <0x0000000780a5d000> (a java.lang.Object)
        at me.qzh.thread.DeadLockTest$$Lambda$1/1078694789.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)

   Locked ownable synchronizers:
        - None

"Service Thread" #11 daemon prio=9 os_prio=0 tid=0x000000001ca8e000 nid=0x768 runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"C1 CompilerThread3" #10 daemon prio=9 os_prio=2 tid=0x000000001c9e7800 nid=0x3b9c waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"C2 CompilerThread2" #9 daemon prio=9 os_prio=2 tid=0x000000001c9c4800 nid=0x1e7c waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"C2 CompilerThread1" #8 daemon prio=9 os_prio=2 tid=0x000000001c9c3800 nid=0x3290 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"C2 CompilerThread0" #7 daemon prio=9 os_prio=2 tid=0x000000001c9c2000 nid=0x84 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"Monitor Ctrl-Break" #6 daemon prio=5 os_prio=0 tid=0x000000001c9da800 nid=0x39f0 runnable [0x000000001d24e000]
   java.lang.Thread.State: RUNNABLE
        at java.net.SocketInputStream.socketRead0(Native Method)
        at java.net.SocketInputStream.socketRead(SocketInputStream.java:116)
        at java.net.SocketInputStream.read(SocketInputStream.java:171)
        at java.net.SocketInputStream.read(SocketInputStream.java:141)
        at sun.nio.cs.StreamDecoder.readBytes(StreamDecoder.java:284)
        at sun.nio.cs.StreamDecoder.implRead(StreamDecoder.java:326)
        at sun.nio.cs.StreamDecoder.read(StreamDecoder.java:178)
        - locked <0x0000000780b2e540> (a java.io.InputStreamReader)
        at java.io.InputStreamReader.read(InputStreamReader.java:184)
        at java.io.BufferedReader.fill(BufferedReader.java:161)
        at java.io.BufferedReader.readLine(BufferedReader.java:324)
        - locked <0x0000000780b2e540> (a java.io.InputStreamReader)
        at java.io.BufferedReader.readLine(BufferedReader.java:389)
        at com.intellij.rt.execution.application.AppMainV2$1.run(AppMainV2.java:47)

   Locked ownable synchronizers:
        - None

"Attach Listener" #5 daemon prio=5 os_prio=2 tid=0x000000001c97e000 nid=0x888 waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"Signal Dispatcher" #4 daemon prio=9 os_prio=2 tid=0x000000001c97d000 nid=0x3b2c runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
        - None

"Finalizer" #3 daemon prio=8 os_prio=1 tid=0x000000001c911800 nid=0x2b44 in Object.wait() [0x000000001ceef000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x0000000780908ed0> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:144)
        - locked <0x0000000780908ed0> (a java.lang.ref.ReferenceQueue$Lock)
        at java.lang.ref.ReferenceQueue.remove(ReferenceQueue.java:165)
        at java.lang.ref.Finalizer$FinalizerThread.run(Finalizer.java:216)

   Locked ownable synchronizers:
        - None

"Reference Handler" #2 daemon prio=10 os_prio=2 tid=0x000000001c910800 nid=0x30bc in Object.wait() [0x000000001cdee000]
   java.lang.Thread.State: WAITING (on object monitor)
        at java.lang.Object.wait(Native Method)
        - waiting on <0x0000000780906bf8> (a java.lang.ref.Reference$Lock)
        at java.lang.Object.wait(Object.java:502)
        at java.lang.ref.Reference.tryHandlePending(Reference.java:191)
        - locked <0x0000000780906bf8> (a java.lang.ref.Reference$Lock)
        at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:153)

   Locked ownable synchronizers:
        - None

"VM Thread" os_prio=2 tid=0x000000001ab09800 nid=0x3980 runnable

"GC task thread#0 (ParallelGC)" os_prio=0 tid=0x0000000003548800 nid=0x3248 runnable

"GC task thread#1 (ParallelGC)" os_prio=0 tid=0x000000000354a000 nid=0x3784 runnable

"GC task thread#2 (ParallelGC)" os_prio=0 tid=0x000000000354b800 nid=0x393c runnable

"GC task thread#3 (ParallelGC)" os_prio=0 tid=0x000000000354d000 nid=0x68c runnable

"GC task thread#4 (ParallelGC)" os_prio=0 tid=0x0000000003550800 nid=0x3a58 runnable

"GC task thread#5 (ParallelGC)" os_prio=0 tid=0x0000000003551800 nid=0x3518 runnable

"GC task thread#6 (ParallelGC)" os_prio=0 tid=0x0000000003555000 nid=0x2dd0 runnable

"GC task thread#7 (ParallelGC)" os_prio=0 tid=0x0000000003556000 nid=0xa70 runnable

"VM Periodic Task Thread" os_prio=2 tid=0x000000001cab2800 nid=0x282c waiting on condition

JNI global references: 316


Found one Java-level deadlock:
=============================
"Thread-1":
  waiting to lock monitor 0x000000001ab10de8 (object 0x0000000780a5d000, a java.lang.Object),
  which is held by "Thread-0"
"Thread-0":
  waiting to lock monitor 0x000000001ab13678 (object 0x0000000780a5d010, a java.lang.Object),
  which is held by "Thread-1"

Java stack information for the threads listed above:
===================================================
"Thread-1":
        at me.qzh.thread.DeadLockTest.lambda$main$1(DeadLockTest.java:37)
        - waiting to lock <0x0000000780a5d000> (a java.lang.Object)
        - locked <0x0000000780a5d010> (a java.lang.Object)
        at me.qzh.thread.DeadLockTest$$Lambda$2/1747585824.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)
"Thread-0":
        at me.qzh.thread.DeadLockTest.lambda$main$0(DeadLockTest.java:23)
        - waiting to lock <0x0000000780a5d010> (a java.lang.Object)
        - locked <0x0000000780a5d000> (a java.lang.Object)
        at me.qzh.thread.DeadLockTest$$Lambda$1/1078694789.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)

Found 1 deadlock.
```

