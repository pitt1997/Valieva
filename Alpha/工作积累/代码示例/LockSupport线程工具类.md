# LockSupport：一个很灵活的线程工具类

LockSupport是一个编程工具类，主要是为了阻塞和唤醒线程用的。使用它我们可以实现很多功能，今天主要就是对这个工具类的讲解，希望对你有帮助：

## **一、LockSupport简介**

### **1、LockSupport是什么**

刚刚开头提到过，LockSupport是一个线程工具类，所有的方法都是静态方法，可以让线程在任意位置阻塞，也可以在任意位置唤醒。

它的内部其实两类主要的方法：park（停车阻塞线程）和unpark（启动唤醒线程）。

```java
package java.util.concurrent.locks;
import sun.misc.Unsafe;

public class LockSupport {
    private LockSupport() {} // Cannot be instantiated.

    // 恢复当前线程
    public static void unpark(Thread thread) {
        if (thread != null)
            UNSAFE.unpark(thread);
    }

    // 阻塞当前线程
    public static void park(Object blocker) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker);
        UNSAFE.park(false, 0L);
        setBlocker(t, null);
    }

    // 暂停当前线程 有超时时间
    public static void parkNanos(Object blocker, long nanos) {
        if (nanos > 0) {
            Thread t = Thread.currentThread();
            setBlocker(t, blocker);
            UNSAFE.park(false, nanos);
            setBlocker(t, null);
        }
    }

    // 暂停当前线程 直到某个时间
    public static void parkUntil(Object blocker, long deadline) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker);
        UNSAFE.park(true, deadline);
        setBlocker(t, null);
    }

    // 无期限暂停当前线程
    public static void park() {
        UNSAFE.park(false, 0L);
    }
     
    // 暂停当前线程 不过有超时时间限制
    public static void parkNanos(long nanos) {
        if (nanos > 0)
            UNSAFE.park(false, nanos);
    }

    // 暂停当前线程 直到某个时间
    public static void parkUntil(long deadline) {
        UNSAFE.park(true, deadline);
    }

    ...
}

```

注意上面的方法中有一个blocker，这个blocker是用来记录线程被阻塞时被谁阻塞的。用于线程监控和分析工具来定位原因的。

现在我们知道了LockSupport是用来阻塞和唤醒线程的，而且之前相信我们都知道wait/notify也是用来阻塞和唤醒线程的，那么它相比，LockSupport有什么优点呢？

### **2、与wait/notify对比**

这里假设你已经了解了wait/notify的机制，如果不了解，可以在网上一搜，很简单。相信你既然学到了这个LockSupport，相信你已经提前已经学了wait/notify。

（1）wait和notify都是Object中的方法,在调用这两个方法前必须先获得锁对象，但是park不需要获取某个对象的锁就可以锁住线程。

（2）notify只能随机选择一个线程唤醒，无法唤醒指定的线程，unpark却可以唤醒一个指定的线程。

区别就是这俩，还是主要从park和unpark的角度来解释的。既然这个LockSupport这么强，我们就深入一下他的源码看看。

## **二、源码分析（基于jdk1.8）**

### **1、park方法**

blocker是用来记录线程被阻塞时被谁阻塞的。用于线程监控和分析工具来定位原因的。setBlocker(t, blocker)方法的作用是记录t线程是被broker阻塞的。因此我们只关注最核心的方法，也就是UNSAFE.park(false, 0L)。

UNSAFE是一个非常强大的类，他的的操作是基于底层的，也就是可以直接操作内存，因此我们从JVM的角度来分析一下：

```
    // 阻塞当前线程
    public static void park(Object blocker) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker);
        UNSAFE.park(false, 0L);
        setBlocker(t, null);
    }
```

https://www.jianshu.com/p/1f16b838ccd8

## **三、LockSupport使用**

https://www.jianshu.com/p/1f16b838ccd8