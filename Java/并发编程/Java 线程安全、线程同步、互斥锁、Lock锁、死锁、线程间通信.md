

各个线程是通过竞争CPU时间而获得运行机会的，什么时候得到CPU，占用多久，是不可预测的，一个正在运行的线程在什么地方被暂停是不确定的，所以，要解决多线程并发访问一个资源的安全性问题，Java中提供了同步机制(`synchronized`)来解决。

在某个线程修改共享资源的时候，其他线程不能修改该资源，等待修改完毕同步之后，才能去抢夺CPU资源，完成对应的操作，保证了数据的同步性，解决了线程不安全的现象。

> 可以理解为：线程同步，即当有一个线程在对内存(数据)进行操作时，其他线程都不可以对这个内存地址(数据)进行操作，直到该线程完成操作，其他线程才可以对该内存地址(数据)进行操作

**实现的三种方式：**
- 同步方法
- 同步代码块
- 锁机制


## 互斥锁
在Java语言中，引入了互斥锁的概念，来保证共享数据操作的完整性。

每个对象都对应于一个可以成为“互斥锁”的标记，这个标记用来保证在任一时刻，只能有一个线程访问该对象。

关键字`synchronized`来与对象的互斥锁联系。当某个对象用`synchronized`修饰时，表明该对象在任一时刻只能有一个线程访问。

**同步的局限性：导致程序的执行效率要降低。**

> ① 锁对象 可以是任意类型。 
> ② 多个线程对象 要使用同一把锁。

在任何时候，最多允许一个线程拥有同步锁，谁拿到锁就进入代码块，其他的线程只能在外等着(**BLOCKED**)。

## 同步方法
使用 `synchronized` 修饰的方法，就叫做同步方法，保证线程执行该方法的时候，其他线程只能在方法外等着。

**语法格式：**
```java
public synchronized void method(){
     // 可能会产生线程安全问题的代码
}
```

对于非static方法，同步锁就是this。(也可以是一个其它对象，要求是同一个对象，看下一章节)

对于static方法，使用当前方法所在类的字节码对象(`类名.class`)。


**测试案例：**
```java
public class ThreadTest {
    public static void main(String[] args) {

        Ticket t = new Ticket();
        Thread thread = new Thread(t);
        Thread thread2 = new Thread(t);
        Thread thread3 = new Thread(t);

        // 开启三个线程
        thread.start();
        thread2.start();
        thread3.start();

    }
}

class Ticket implements Runnable {

    private int ticket = 100;   // 总票数
    private boolean loop = true; // 是否售票

    private synchronized void sellTicket() {
        if (ticket <= 0) {
            System.out.println("票卖完了");
            loop = false;
            return;
        }
        // 有票 可以卖
        // 出票操作
        // 使用sleep模拟一下出票时间
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //获取当前线程对象的名字
        String name = Thread.currentThread().getName();
        // 卖好后 票的数量-1
        System.out.println(name + "正在卖:" + ticket--);

    }

    @Override
    public void run() {
        // 执行卖票操作，只要有票，卖票窗口永远开启
        while (loop) {
            sellTicket();
        }
    }
}

```


## 同步代码块
`synchronized`关键字可以用于方法中的某个区块中，表示只对这个区块的资源实行互斥访问。

**语法格式：**
```java
synchronized(同步锁){	// 同步锁：就是一个对象，得到该锁对象的线程，才能操作同步代码

    // 需要同步操作的代码（可能出现线程安全问题的代码）
    
}
```
> **同步锁：** 对象的同步锁只是一个概念，可以想象为在对象上标记了一个锁。在任何时候，最多允许一个线程拥有同步锁，谁拿到锁就进入代码块，其他的线程只能在外等着(BLOCKED)。
> 1. 锁对象 可以是任意类型。 
> 2. 多个线程对象 要使用同一把锁。



可以创建一个对象，用于作为锁对象。

**测试案例：**
```java
public class ThreadTest {
    public static void main(String[] args) {

        Ticket t = new Ticket();
        Thread thread = new Thread(t);
        Thread thread2 = new Thread(t);
        Thread thread3 = new Thread(t);

        // 开启三个线程
        thread.start();
        thread2.start();
        thread3.start();

    }
}

class Ticket implements Runnable {

    private int ticket = 100;   // 总票数
    private boolean loop = true; // 是否售票
    private final Object obj = new Object();

    @Override
    public void run() {
        // 执行卖票操作，只要有票，卖票窗口永远开启
        while (loop) {
            synchronized (obj) {
                if (ticket <= 0) {
                    System.out.println("票卖完了");
                    loop = false;
                    return;
                }
                // 有票 可以卖
                // 出票操作
                // 使用sleep模拟一下出票时间
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                //获取当前线程对象的名字
                String name = Thread.currentThread().getName();
                // 卖好后 票的数量-1
                System.out.println(name + "正在卖:" + ticket--);

            }
        }
    }
}
```

## 线程死锁
多个线程都占用了对方的锁资源，但不肯想让，导致了死锁。

**需要避免该问题**

> 比如：
> 妈妈：你先写完作业，才能玩手机
> 小明：我先玩会手机，再写作业


**死锁案例：**
```java
public class ThreadTest {
    public static void main(String[] args) {

        Ticket t1 = new Ticket(true);
        Ticket t2 = new Ticket(false);

        t1.start();
        t2.start();

    }
}

class Ticket extends Thread {

    private final boolean flag;

    public Ticket(boolean flag) {
        this.flag = flag;
    }

    static final Object o1 = new Object();
    static final Object o2 = new Object();

    @Override
    public void run() {
        // 如果 flag 为 true ，线程1 就会先得到 o1 的对象锁，然后尝试获取 o2 的对象锁
        // 如果 线程1 得不到 o2 的对象锁，就会进入 Blocked 状态

        // 如果 flag 为 false ，线程2 就会先得到 o2 的对象锁，然后尝试获取 o1 的对象锁
        // 如果 线程2 得不到 o1 的对象锁，就会进入 Blocked 状态
        if (flag) {
            synchronized (o1) {
                System.out.println(Thread.currentThread().getName() + "进入1");
                synchronized (o2) {
                    System.out.println(Thread.currentThread().getName() + "进入2");
                }
            }
        } else {
            synchronized (o2) {
                System.out.println(Thread.currentThread().getName() + "进入3");
                synchronized (o1) {
                    System.out.println(Thread.currentThread().getName() + "进入4");
                }
            }
        }
    }
}

```

运行该程序，可以看到下面结果：
> 两个线程都进入了 Blocked 状态，都需要获得锁才能继续执行，都不能释放锁，所以导致了死锁。
> 
![在这里插入图片描述](assets/Java%20线程安全、线程同步、互斥锁、Lock锁、死锁、线程间通信/2f2bbf2d01b40841ca56dc91aa7bd82e_MD5.png)

## 释放锁

下面几种情况会释放锁：
> ① 当前线程的同步方法、同步代码块执行结束
> ② 当前线程在同步方法、同步代码块遇到 return、break
> ③ 当前线程在同步方法、同步代码块中出现了未处理的Error或Exception，导致异常结束
> ④ 当前线程在同步方法、同步代码块中执行了线程对象的`wait()`方法，当前线程暂停，并释放锁


而下面的情况不会释放锁：
> ① 线程执行同步方法、同步代码块时，线程调用`Thread.sleep()`、`Thread.yield()`方法暂停当前线程的执行，不会释放锁
> ② 线程执行同步代码块时，其他行程调用了该线程的`suspend()`方法将该线程挂起，线程不会释放锁，应尽量避免使用`suspend()`和`resume()`方法来控制线程

## Lock锁
`java.util.concurrent.locks.Lock`机制提供了比`synchronized`代码块和`synchronized`方法更广泛的锁定操作，同步代码块/同步方法具有的功能Lock都有，除此之外更强大，更体现面向对象。

Lock锁也称同步锁，加锁与释放锁方法化了，如下：

`void lock()`：加同步锁
`void unlock()`：释放同步锁

使用案例：
```java
 public class Ticket implements Runnable{  
     private int ticket = 100;  
     Lock lock = new ReentrantLock();  
     /** 执行卖票操作*/  
     @Override  
     public void run() {  
        //每个窗口卖票的操作  
        //窗口 永远开启  
        while(true){  
             lock.lock();  
             if(ticket>0){  
                 //有票 可以卖  
                 //出票操作  
                 //使用sleep模拟一下出票时间  
                 try {  
                     Thread.sleep(50);  
                } catch (InterruptedException e) {  
                     // TODO Auto‐generated catch block  
                    e.printStackTrace();  
                 }  
                 //获取当前线程对象的名字  
                String name = Thread.currentThread().getName();  
                System.out.println(name+"正在卖:"+ticket‐‐);  
             }  
            lock.unlock();  
         }  
    }  
}  
```


## 线程间通信

多个线程在处理同一个资源，但是处理的动作（线程的任务）却不相同。

多个线程并发执行时，在默认情况下CPU是随机切换线程的，当需要多个线程来共同完成一件任务，并且希望他们有规律的执行，那么多线程之间需要一些协调通信，以此来达到多线程共同操作一份数据。

多个线程在处理同一个资源，并且任务不同时，需要线程通信来帮助解决线程之间对同一个变量的使用或操作。就是多个线程在操作同一份数据时， 避免对同一共享变量的争夺。也就是需要通过一定的手段使各个线程能有效的利用资源。而这种手段即——等待唤醒机制。

### 等待唤醒机制
等待唤醒机制是多个线程间的一种协作机制。谈到线程我们经常想到的是线程间的竞争（race），比如去争夺锁，但这并不是故事的全部，线程间也会有协作机制。就好比在公司里你和你的同事们，你们可能存在在晋升时的竞争，但更多时候你们是一起合作以完成某些任务。

就是在一个线程进行了规定操作后，就进入等待状态（`wait()`），等待其他线程执行完他们的指定代码过后再将其唤醒（`notify()`；在有多个线程进行等待时，如果需要，可以使用`notifyAll()`来唤醒所有的等待线程。

**wait/notify 就是线程间的一种协作机制。**


等待唤醒中的方法：  `java.lang.Object`

等待唤醒机制就是用于解决线程间通信的问题的，使用到的3个方法的含义如下：

`wait()`：使线程不再活动，不再参与调度，进入wait set中，因此不会浪费 CPU 资源，也不会去竞争锁了，这时的线程状态即是WAITING。它还要等着别的线程执行一个特别的动作，也即是“通知(notify)”在这个对象上等待的线程从wait set中释放出来，重新进入到调度队列（ready queue）中。

`notify()`：选取所通知对象的wait set中的一个线程释放。例如，餐馆有空位置后，等候就餐最久的顾客最先入座。

`notifyAll()`：则释放所通知对象的wait set上的全部线程。


哪怕只通知了一个等待的线程，被通知线程也不能立即恢复执行，因为它当初中断的地方是在同步块内，而此刻它已经不持有锁，所以它需要再次尝试去获取锁（很可能面临其它线程的竞争），成功后才能在当初调用 wait 方法之后的地方恢复执行。


如果能获取锁，线程就从 WAITING 状态变成 RUNNABLE 状态；否则，从 wait set 出来，又进入 entry set，线程就从 WAITING 状态又变成 BLOCKED 状态。




> ① wait方法与notify方法必须要由同一个锁对象调用。因为对应的锁对象可以通过notify唤醒使用同一个锁对象调用的wait方法后的线程。
> ② wait方法与notify方法是属于Object类的方法的。因为锁对象可以是任意对象，而任意对象的所属类都是继承了Object类的。
> ③ wait方法与notify方法必须要在同步代码块或者是同步函数中使用。因为必须要通过锁对象调用这2个方法。









