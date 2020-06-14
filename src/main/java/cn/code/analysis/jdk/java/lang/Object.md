# java.lang.Object 
[Java面试系列第2篇-Object类中的方法](https://www.cnblogs.com/extjs4/p/12772027.html)

[JDK1.8源码(一)——java.lang.Object类](https://www.cnblogs.com/ysocean/p/8419559.html)
```java
package java.lang;
/* *
 * 所有类的父类
 */
public class Object {

    private static native void registerNatives();
    static {
        registerNatives();// 保证在clinit()最先执行，从而调native方法
    }
    // 返回此 Object的运行时类。返回的是个泛型，运行时泛型会进行类型擦除，实际返回以自身为边界
    public final native Class<?> getClass();
    // 返回对象的哈希码值。 
    public native int hashCode();
    // 指示一些其他对象是否等于此。 
    public boolean equals(Object obj) {
        return (this == obj);
    }
    // 创建并返回此对象的副本。 
    protected native Object clone() throws CloneNotSupportedException;
    // 返回对象的字符串表示形式。 建议所有类都重写这个方法，因为不直观。字符串内容，类名加上hashcode转成16进制
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
    // 随机唤醒正在等待对象监视器的一个线程。 
    //被唤醒的线程不会立刻拿到对象的监视器锁，而是和其他在线程共同竞争，拿到锁之后恢复到调用wait方法时的同步状态
    // 同一时间只有一个线程能拿到监视器锁
    public final native void notify();
    // 唤醒正在等待对象监视器的所有线程。
    public final native void notifyAll();
    // 导致当前线程等待，直到另一个线程调用 notify()方法或该对象的 notifyAll()方法，或者指定的时间已过。 
    //  持有对象监视器的线程可以执行awit方法把自己放入等待池中，并且放弃持有该对象的所有同步资源，不会放弃获取的其他对象的同步资源
    public final native void wait(long timeout) throws InterruptedException;
    // 提供纳秒级别的等待控制，导致当前线程等待，直到另一个线程调用该对象的 notify()方法或 notifyAll()方法，或者某些其他线程中断当前线程，或等待时间到了。 
    public final void wait(long timeout, int nanos) throws InterruptedException {
        if (timeout < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }

        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                                "nanosecond timeout value out of range");
        }

        if (nanos > 0) {
            timeout++;
        }

        wait(timeout);
    }
    // 导致当前线程等待，直到另一个线程调用该对象的 notify()方法或 notifyAll()方法。 没有超时时间。
    public final void wait() throws InterruptedException {
        wait(0);
    }
    //当垃圾收集器判断此对象没有被任何引用，是垃圾对象了，由垃圾收集器执行此方法销毁对象，所以何时执行是不确定的
    //JVM保证垃圾收集器调用此方法时候，该对象的锁不被任何线程持有了
    //finalize方法只会被执行一次，所以我们可以重写此方法特定一些清除工作，或者给对象一次复活机会，但是不建议重写此方法
    protected void finalize() throws Throwable { }
}
```
