相对于 synchronized 它具备如下特点

- 可中断
- 可以设置超时时间
- 可以设置为公平锁
- 支持多个条件变量

与 synchronized 一样，都支持可重入

基本语法：

```java
// 获取锁
reentrantLock.lock();

try {
	// 临界区
} finally {
	// 释放锁
	reentrantLock.unlock();
}
```


## 可重入

可重入是指同一个线程如果首次获得了这把锁，那么因为它是这把锁的拥有者，因此有权利再次获取这把锁。

> 如果是不可重入锁，那么第二次获得锁时，自己也会被锁挡住。


```java
static ReentrantLock lock = new ReentrantLock();  

public static void main(String[] args) {  
    method1();  
}  

public static void method1() {  
    lock.lock();  
    try {  
        log.debug("execute method1");  
        method2();  
    } finally {  
        lock.unlock();  
    }  
}  

public static void method2() {  
    lock.lock();  
    try {  
        log.debug("execute method2");  
        method3();  
    } finally {  
        lock.unlock();  
    }  
}  

public static void method3() {  
    lock.lock();  
    try {  
        log.debug("execute method3");  
    } finally {  
        lock.unlock();  
    }  
}
```


## 可打断

可以使在阻塞队列中的线程打断阻塞，以防止死锁的产生。

```java
ReentrantLock lock = new ReentrantLock();  
Thread t1 = new Thread(() -> {  
    log.debug("启动...");  
    try {  
        // 尝试获得锁，如果没有竞争，直接获得锁  
        // 如果有竞争，进入阻塞队列，可以被其他线程使用interrupt方法打断  
        lock.lockInterruptibly();  
    } catch (InterruptedException e) {  
        e.printStackTrace();  
        log.debug("等锁的过程中被打断");  
        // 没有获得锁，直接退出方法  
        return;  
    }  
    try {  
        log.debug("获得了锁");  
    } finally {  
        lock.unlock();  
    }  
}, "t1");  

lock.lock();  
log.debug("获得了锁");  
// 先使主线程获得锁，再启动 t1 线程  
t1.start();  

try {  
    sleep(1);  
    t1.interrupt();  
    log.debug("执行打断");  
} finally {  
    lock.unlock();  
}
```


## 锁超时

可以使在阻塞队列中的线程超时后放弃获取锁，以防止死锁的产生。

```java
ReentrantLock lock = new ReentrantLock();  
Thread t1 = new Thread(() -> {  
    log.debug("启动...");  
    try {  
        // 等待一秒，不传参数的话表示不等待  
        if (!lock.tryLock(1, TimeUnit.SECONDS)  
            log.debug("获取等待 1s 后失败，返回");  
            return;  
        }  
    } catch (InterruptedException e) {  
        e.printStackTrace();  
    }  
    try {  
        log.debug("获得了锁");  
    } finally {  
        lock.unlock();  
    }  
}, "t1");  

lock.lock();  
log.debug("获得了锁");  

t1.start();  

try {  
    sleep(2);  
} finally {  
    lock.unlock();  
}
```

## 公平锁

ReentrantLock 默认是不公平的，不公平的情况可能会导致饥饿锁。先入先得，公平锁一般没有必要，会降低并发度。一般使用锁超时可以解决问题。

```java
ReentrantLock lock = new ReentrantLock(ture);  
lock.lock();  
for (int i = 0; i < 20; i++) {  
    new Thread(() -> {  
        lock.lock();  
        try {  
            System.out.println(Thread.currentThread().getName() + " running...");  
        } finally {  
            lock.unlock();  
        }  
    }, "t" + i).start();  
}  

// 1s 之后去争抢锁  
Thread.sleep(1000);  
for (int i = 0; i < 5; i++) {  
    new Thread(() -> {  
        lock.lock();  
        try {  
            System.out.println(Thread.currentThread().getName() + " running...");  
        } finally {  
            lock.unlock();  
        }  
    }, "强行插入").start();  
}  

lock.unlock();
```


## 条件变量

synchronized 中也有条件变量，就是 waitSet，当条件不满足时进入 waitSet 等待。

![](assets/Java原理synchronized/5hnqNGDAzIUOibO7KEtcKWyc6IqLpCWM-f7j8PqUjJQ.png)

ReentrantLock 的条件变量比 synchronized 强大之处在于，它是支持多个条件变量的，这就好比 synchronized 是那些不满足条件的线程都在一间休息室等消息。
而 ReentrantLock 支持多间休息室，有专门等烟的休息室、专门等早餐的休息室、唤醒时也是按休息室来唤醒。

使用要点：
- await 前需要获得锁
- await 执行后，会释放锁，进入 conditionObject 等待
- await 的线程被唤醒（或打断、或超时）取重新竞争 lock 锁
- 竞争 lock 锁成功后，从 await 后继续执行

```java
static boolean hasCigarette = false;  
static boolean hasTakeout = false;  
static ReentrantLock ROOM = new ReentrantLock();  
// 等待烟的休息室  
static Condition waitCigaretteSet = ROOM.newCondition();  
// 等外卖的休息室  
static Condition waitTakeoutSet = ROOM.newCondition();  
public static void main(String[] args) {  
    new Thread(() -> {  
        // 加锁  
        ROOM.lock();  
        try {  
            log.debug("有烟没？[{}]", hasCigarette);  
            while (!hasCigarette) {  
                log.debug("没烟，先歇会！");  
                try {  
                    // 如果不满足条件，则进行等待  
                    waitCigaretteSet.await();  
                } catch (InterruptedException e) {  
                    throw new RuntimeException(e);  
                }  
            }  
            log.debug("可以开始干活了");  
        } finally {  
            // 结束后释放锁  
            ROOM.unlock();  
        }  
    }, "小明").start(); 
    
	 
    new Thread(() -> {  
        ROOM.lock();  
        try {  
            log.debug("外卖送到没？[{}]", hasTakeout);  
            while (!hasTakeout) {  
                log.debug("没外卖，先歇会！");  
                try {  
                    waitTakeoutSet.await();  
                } catch (InterruptedException e) {  
                    throw new RuntimeException(e);  
                }  
            }  
            log.debug("可以开始干活了");  
        } finally {  
            ROOM.unlock();  
        }  
    }, "小红").start();  
	
	
    sleep(1);  
    new Thread(() -> {  
        ROOM.lock();  
        try {  
            hasTakeout = true;  
            waitTakeoutSet.signal();  
        } finally {  
            ROOM.unlock();  
        }  
    }, "送外卖的").start();  
	
	
    sleep(1);  
    new Thread(() -> {  
        // 加锁  
        ROOM.lock();  
        try {  
            // 当完成某些条件后，使等待的线程进入可运行的状态  
            hasCigarette = true;  
            waitCigaretteSet.signal();  
            // waitCigaretteSet.signalAll();  
        } finally {  
            // 释放锁  
            ROOM.unlock();  
        }  
    }, "送烟的").start();  
}
```

