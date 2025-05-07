
## wait&notify原理

![](assets/Java原理synchronized/5hnqNGDAzIUOibO7KEtcKWyc6IqLpCWM-f7j8PqUjJQ.png)

- Owner 线程发现条件不满足，调用 wait 方法，即可进入 WaitSet 变为 WAITING 状态
- BLOCKED 和 WAITING 的线程都处于阻塞状态，不占用 CPU 时间片
- BLOCKED 线程会在 Owner 线程释放锁时唤醒
- WAITING 线程会在 Owner 线程调用 notify 或 notifyAll 时唤醒，但唤醒后并不意味者立刻获得锁，仍需进入EntryList 重新竞争

### API

- `obj.wait()`：让进入 object 监视器的线程到 waitSet 等待
- `wait(long n)`：有时限的等待，到 n 毫秒后结束等待，或是被 notify
- `obj.notify()`：在 object 上正在 waitSet 等待的线程中挑一个唤醒
- `obj.notifyAll()`：让 object 上正在 waitSet 等待的线程全部唤醒

它们都是线程之间进行协作的手段，都属于 Object 对象的方法。**必须获得此对象的锁，才能调用这几个方法。**

wait() 方法会释放对象的锁，进入 WaitSet 等待区，从而让其他线程就机会获取对象的锁。无限制等待，直到notify 为止。

```java
final static Object obj = new Object();  
public static void main(String[] args) {  
    new Thread(() -> {  
        synchronized (obj) {  
            log.debug("执行....");  
            try {  
                obj.wait(); // 让线程在obj上一直等待下去  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
            log.debug("其它代码....");  
        }  
    }, "t1").start();  
    
    new Thread(() -> {  
        synchronized (obj) {  
            log.debug("执行....");  
            try {  
                obj.wait(); // 让线程在obj上一直等待下去  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
            log.debug("其它代码....");  
        }  
    }, "t2").start();  
    
    // 主线程两秒后执行  
    sleep(0.5);  
    log.debug("唤醒 obj 上其它线程");  
    synchronized (obj) {  
        obj.notify(); // 唤醒obj上一个线程  
        // obj.notifyAll(); // 唤醒obj上所有等待线程  
    }  
}
```



### `sleep(long n)`和`wait(long n)`的区别

1. sleep 是 Thread 方法，而 wait 是 Object 的方法
2. sleep 不需要强制和 synchronized 配合使用，但 wait 需要和 synchronized 一起用
3. sleep 在睡眠的同时，不会释放对象锁的，但 wait 在等待的时候会释放对象锁 
4. 它们状态都是`TIMED_WAITING`

## join原理

join方法的原理和[同步模式：保护性暂停](同步模式：保护性暂停)的实现是一样的

join方法的源码：
```java
public final synchronized void join(long millis)  
throws InterruptedException {  
    long base = System.currentTimeMillis();  
    long now = 0;  
    if (millis < 0) {  
        throw new IllegalArgumentException("timeout value is negative");  
    }  
    if (millis == 0) {  
        while (isAlive()) {  
            wait(0);  
        }  
    } else {  
	    // 如果线程存活，计算剩余需要等待的时间，并进入 wait 方法
        while (isAlive()) {  
            long delay = millis - now;  
            if (delay <= 0) {  
                break;  
            }  
            wait(delay);  
            now = System.currentTimeMillis() - base;  
        }  
    }  
}
```