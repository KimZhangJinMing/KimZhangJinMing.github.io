---
title: ThreadLocal.md
date: 2020-08-10 22:46:35
categories: 并发
tags: JUC
---



### 1. 两大使用场景 

* 每个线程需要一个自己独享的对象（通常是工具类，例如SimpleDateFormat和Random）
  * 使用时重写ThreadLocal的initValue方法
  * 需要保存到ThreadLocal里的对象的生成由我们控制
* 每个线程内需要保存全局变量（例如在拦截器中获取用户信息），可以让不同方法直接使用，避免参数传递的麻烦
  * 使用时手动调用ThreadLocal的set方法
  * 需要保存到ThreadLocal里的对象的生成时机不由我们随意控制

### 2.ThreadLocal的作用

* 让某个需要用到的对象在线程间隔离
* 在任何方法中都能获取到需要的对象

### 3.使用ThreadLocal的好处

* 达到线程安全
* 不需要加锁，提高执行效率
* 更高效地利用内存，节省开销
* 避免繁琐的传参

#### 栗子1.每个线程都需要一个独享的ThreadLocal对象

```java
/**
 * @Author Ming
 * @Date 2020/07/25 16:29
 * @Description 每个线程都需要一个SimpleDateFormat对象，使用ThreadLocal来达到线程安全
 */
public class ThreadLocalNormalUseage01 {
    private static ExecutorService executorService = Executors.newFixedThreadPool(10);
    private static ThreadLocal<SimpleDateFormat> threadLocal = ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd hh:mm:ss"));

    public static void main(String[] args) {
        for (int i = 0; i < 1000; i++) {
            int finalI = i;
            executorService.submit(() -> {
                String date = date(finalI);
                System.out.println(date);
            });
        }

        executorService.shutdown();
    }

    public static String date(int seconds) {
        Date date = new Date(1000 * seconds);
        return threadLocal.get().format(date);
    }
}
```



### 4.ThreadLocal的原理

* initialValue()
  * 该方法会返回当前线程对应的“初始值”，这是一个延迟加载的方法，只有在调用get()方法的时候，才会触发
  * 当线程第一个使用get方法访问变量时，将调用此方法，除非线程先前调用了set方法，在这种情况下，不会为线程调用本initialValue方法
  * 每个线程最多调用一次initialValue方法，但如果已经调用了remove后，再调用get，则可以再次调用此方法
  * 如果不重写本方法，这个方法会返回null

 使用ThreadLocal维护变量时，ThreadLocal为每个使用该变量的线程提供独立的变量副本，所以每 一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本 

#### 栗子1

```java
public class ThreadDemo21 {
    public static void main(String[] args) throws InterruptedException {

        ThreadLocal<Integer> th = new ThreadLocal<Integer>();

        new Thread(() -> {
            try {
                th.set(100);
                System.out.println("t1 set th=" + th.get()); // 100
                Thread.sleep(2000);
                System.out.println("t1 get th=" + th.get()); // 100
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        Thread.sleep(1000);

        new Thread(() -> {
            Integer ele = th.get();
            System.out.println("t2 get th=" + ele); // null
            th.set(200);
            System.out.println("t2 get th=" + th.get()); // 200
        }).start();
    }
}
```

执行结果：

线程t1先执行，设置th=100,sleep 2s，线程t2执行，先获取th的值=null，而不是线程t1设置的100。

线程t2设置th=200，线程t1sleep2s后获取th的值=100，而不是线程2设置的200。

每个线程的th变量是独立的。

```java
t1 set th=100
t2 get th=null
t2 get th=200
t1 get th=100
```

### ThreadLocal与InheritableThreadLocal

*  Thread类中的threadLocals、inheritableThreadLocals成员变量为ThreadLocal.ThreadLocalMap对象 
*  Map的key值是ThreadLocal对象本身，value是Object类型 
*  ThreadLocal无法解决继承问题，而InheritableThreadLocal可以 
*  InheritableThreadLocal继承自ThreadLocal 
*  InheritableThreadLocal可以帮助我们做链路追踪

#### 栗子1：ThreadLocal只能获取当前线程设置的值

```java
public class ThreadLocalDemo0 {

    public static ThreadLocal<String> tl = new ThreadLocal<>();

    public static void main(String[] args) {

        tl.set("Kevin是一个自由讲师");

        Thread t0 = new Thread(() -> {
            // 子线程无法获取父线程ThreadLocal的值,因为他们是两个独立的用户线程，都使用各自的副本
            System.out.println(Thread.currentThread().getName() + " get tl is : " + tl.get());
        });

        t0.start();

        // 主线程可以获取到
        System.out.println(Thread.currentThread().getName() + " get tl is : " + tl.get());
    }
}
```

运行结果：

```java
main get tl is : Kevin是一个自由讲师
Thread-0 get tl is : null
```

#### 栗子2：InhertiableThreadLocal可以在父子线程中复制传递值

```java
public class ThreadLocalDemo1 extends Thread {

    // 多个线程之间读取副本, 父子线程之间复制传递
    public static InheritableThreadLocal<String> tl = new InheritableThreadLocal<>();

    public static void main(String[] args) throws InterruptedException {

        tl.set("Kevin是一个自由讲师");

        Thread t0 = new Thread(() -> {
            // t0线程先启动，可以获取main线程中设置的值，读取了main线程中的副本
            System.out.println(Thread.currentThread().getName() + " get tl is : " + tl.get());
            // t0线程修改值，并不影响其他线程
            tl.set("After Set The Value Change to " + Thread.currentThread().getName());
            System.out.println(Thread.currentThread().getName() + " get tl is : " + tl.get());
        });

        Thread t1 = new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + " get tl is : " + tl.get());
            tl.set("After Set The Value Change to " + Thread.currentThread().getName());
            System.out.println(Thread.currentThread().getName() + " get tl is : " + tl.get());
        });

        t0.start();

        Thread.sleep(1000);

        t1.start();

        System.out.println(Thread.currentThread().getName() + " get tl is : " + tl.get());
    }
}
```

执行结果：

```java
Thread-0 get tl is : Kevin是一个自由讲师
Thread-0 get tl is : After Set The Value Change to Thread-0
main get tl is : Kevin是一个自由讲师
Thread-1 get tl is : Kevin是一个自由讲师
Thread-1 get tl is : After Set The Value Change to Thread-1
```

### 6.内存泄露

内存泄露：某个对象不再有用，但是占用的内存不能被GC回收。

弱引用：如果这个对象只被弱引用关联（没有任何强引用关联），那么这个对象就可以被回收。ThredLocalMap的key就是一个弱引用

* ThreadLocalMap的每个Entry都是一个对key的弱引用，同时，每个Entry都包含了一个对value的强引用
* 正常情况下，当线程终止，保存在ThreadLocal里的value会被垃圾回收，因为没有任何强引用了
* 但是，如果线程不终止（比如线程需要保持很久），那么key对应的value就不能被回收
* 因为value和Thread之间还存在这个强引用链路，所以导致value无法回收，就可能会出现OOM
* JDK在set，remove，rehash方法中会扫描key为null的Entry，并把对应的value设置为null，这样value对象就可以被回收

如何避免内存泄露：使用完ThreadLocal之后，应该调用remove方法，就会删除对应的Entry对象，可以避免内存泄露

### 7.空指针异常

如果ThreadLocal没有设置初始值，就调用get方法，会返回null。这本身没有问题，但如果遇到拆箱的操作，就会引发NPE。原因在于拆箱调用的是对象的.valueOf方法。

