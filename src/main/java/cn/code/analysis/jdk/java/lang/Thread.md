# java.lang.Thread
[Java读源码之Thread](https://www.cnblogs.com/freshchen/p/11674575.html)
```java
package java.lang;

public class Thread implements Runnable {

    private volatile String name;
    // 优先级
    private int            priority;
    //是否后台
    private boolean     daemon = false;
    /* JVM state */
    private boolean     stillborn = false;
    // 要跑的任务
    private Runnable target;
    // 线程组
    private ThreadGroup group;
    // 上下文加载器
    private ClassLoader contextClassLoader;
    // 权限控制上下文
    private AccessControlContext inheritedAccessControlContext;
    // 线程默认名字“Thread-{{ threadInitNumber }}”
    private static int threadInitNumber;
    // 线程本地局部变量，每个线程拥有各自独立的副本
    ThreadLocal.ThreadLocalMap threadLocals = null;
    // 有时候线程本地局部变量需要被子线程继承
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;
    // 线程初始化时申请的JVM栈大小
    private long stackSize;
    // 线程ID
    private long tid;
    // 线程init之后的ID
    private static long threadSeqNumber;
    // 0就是线程还处于NEW状态，没start
    private volatile int threadStatus = 0;
    // 给LockSupport.park用的需要竞争的对象
    volatile Object parkBlocker;
    // 给中断用的需要竞争的对象
    private volatile Interruptible blocker;
    // 线程最小优先级
    public final static int MIN_PRIORITY = 1;
    // 线程默认优先级
    public final static int NORM_PRIORITY = 5;
    // 线程最大优先级
    public final static int MAX_PRIORITY = 10;
    // Thread类中的枚举
    public enum State {
        // 线程刚创建出来还没start
        NEW,
        // 线程在JVM中运行了，需要去竞争资源，例如CPU
        RUNNABLE,
        // 线程等待获取对象监视器锁，损被别人拿着就阻塞
        BLOCKED,
        // 线程进入等待池了，等待觉醒
        WAITING,
        // 指定了超时时间
        TIMED_WAITING,
        // 线程终止
        TERMINATED;
    }
    //sleep方法和Object.wait方法如出一辙，都是调用本地方法，提供毫秒和纳秒两种级别的控制，唯一区别就是，sleep不会放弃任何占用的监视器锁
    public static native void sleep(long millis) throws InterruptedException;
    // 纳秒级别控制
    public static void sleep(long millis, int nanos) throws InterruptedException {
        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }
    
        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                "nanosecond timeout value out of range");
        }
    
        if (nanos >= 500000 || (nanos != 0 && millis == 0)) {
            millis++;
        }
    
        sleep(millis);
    } 
    //初始化
    private void init(ThreadGroup g, Runnable target, String name,
                          long stackSize) {
            init(g, target, name, stackSize, null, true);
    }
    private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize, AccessControlContext acc,
                      boolean inheritThreadLocals) {
        if (name == null) {
            throw new NullPointerException("name cannot be null");
        }
    
        this.name = name;
    	// 获取当前线程，也就是需要被创建线程的爸爸
        Thread parent = currentThread();
        SecurityManager security = System.getSecurityManager();
        if (g == null) {
            // 通过security获取线程组，其实就是拿的当前线程的组
            if (security != null) {
                g = security.getThreadGroup();
            }
    
            // 获取当前线程的组，这下确保肯定有线程组了
            if (g == null) {
                g = parent.getThreadGroup();
            }
        }
    
        // check一下组是否存在和是否有线程组修改权限
        g.checkAccess();
    
        // 子类执行权限检查，子类不能重写一些不是final的敏感方法
        if (security != null) {
            if (isCCLOverridden(getClass())) {
                security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
            }
        }
    	// 组里未启动的线程数加1，长时间不启动就会被回收
        g.addUnstarted();
    	// 线程的组，是否后台，优先级，初始全和当前线程一样
        this.group = g;
        this.daemon = parent.isDaemon();
        this.priority = parent.getPriority();
        if (security == null || isCCLOverridden(parent.getClass()))
            // 子类重写check没过或者就没有security，这里要check下是不是连装载的权限都没有
            this.contextClassLoader = parent.getContextClassLoader();
        else
            this.contextClassLoader = parent.contextClassLoader;
        // 访问控制上下文初始化
        this.inheritedAccessControlContext =
            acc != null ? acc : AccessController.getContext();
        // 任务初始化
        this.target = target;
        // 设置权限
        setPriority(priority);
        // 如果有需要继承的ThreadLocal局部变量就copy一下
        if (inheritThreadLocals && parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals =
            ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
        // 初始化JVM中待创建线程的栈大小
        this.stackSize = stackSize;
    
        // threadSeqNumber线程号加1
        tid = nextThreadID();
    }
    //构造方法 进入初始New状态
    public Thread() {
        init(null, null, "Thread-" + nextThreadNum(), 0);
    }
    public Thread(Runnable target) {
            init(null, target, "Thread-" + nextThreadNum(), 0);
    }
    Thread(Runnable target, AccessControlContext acc) {
            init(null, target, "Thread-" + nextThreadNum(), 0, acc, false);
    }
    public Thread(ThreadGroup group, Runnable target) {
            init(group, target, "Thread-" + nextThreadNum(), 0);
    }
    public Thread(String name) {
            init(null, null, name, 0);
    }
    public Thread(ThreadGroup group, String name) {
            init(group, null, name, 0);
    }
    public Thread(Runnable target, String name) {
            init(null, target, name, 0);
    }
    public Thread(ThreadGroup group, Runnable target, String name) {
            init(group, target, name, 0);
    }
    public Thread(ThreadGroup group, Runnable target, String name,
                      long stackSize) {
            init(group, target, name, stackSize);
    }
    //线程处于NEW状态了，需要调用start方法，让线程进入RUNNABLE状态
    //当获得了执行任务所需要的资源后，JVM便会调用target（Runnable）的run方法。
    //注意：我们永远不要对同一个线程对象执行两次start方法
    public synchronized void start() {
        // 0就是NEW状态
        if (threadStatus != 0)
            throw new IllegalThreadStateException();
    
        // 把当前线程加到线程组的线程数组中，然后nthreads线程数加1，nUnstartedThreads没起的线程数减1
        group.add(this);
    
        boolean started = false;
        // 请求资源
        try {
            start0();
            started = true;
        } finally {
            try {
                if (!started) {
        // 起失败啦，把当前线程从线程组的线程数组中删除，然后nthreads减1，nUnstartedThreads加1
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                // start0出问题会自己打印堆栈信息
            }
        }
    }
    // 设置别的线程中断
    public void interrupt() {
        if (this != Thread.currentThread())
            checkAccess();
    	// 拿一个可中断对象Interruptible的锁
        synchronized (blockerLock) {
            Interruptible b = blocker;
            if (b != null) {
                interrupt0();           // 设置中断标志位
                b.interrupt(this);
                return;
            }
        }
        interrupt0();
    }
    private native void start0();
     @Override
        public void run() {
            if (target != null) {
                target.run();
            }
        }
     private void exit() {
            if (group != null) {
                group.threadTerminated(this);
                group = null;
            }
            /* Aggressively null out all reference fields: see bug 4006245 */
            target = null;
            /* Speed the release of some of these resources */
            threadLocals = null;
            inheritableThreadLocals = null;
            inheritedAccessControlContext = null;
            blocker = null;
            uncaughtExceptionHandler = null;
        }
      public void interrupt() {
             if (this != Thread.currentThread())
                 checkAccess();
     
             synchronized (blockerLock) {
                 Interruptible b = blocker;
                 if (b != null) {
                     interrupt0();           // Just to set the interrupt flag
                     b.interrupt(this);
                     return;
                 }
             }
             interrupt0();
         }
    // 获取当前线程中断标志位，然后重置中断标志位
    public static boolean interrupted() {
        return currentThread().isInterrupted(true);
    }
    
    // 检查线程中断标志位
    public boolean isInterrupted() {
        return isInterrupted(false);
    }
    //有些场景是需要根据线程的优先级来调度的，优先级越大越先执行，最大10，默认5，最小1
    public final void setPriority(int newPriority) {
        ThreadGroup g;
        checkAccess();
        if (newPriority > MAX_PRIORITY || newPriority < MIN_PRIORITY) {
            throw new IllegalArgumentException();
        }
        if((g = getThreadGroup()) != null) {
            // 如果设置的优先级，比线程所属线程组中优先级的最大值还大，我们需要更新最大值
            if (newPriority > g.getMaxPriority()) {
                newPriority = g.getMaxPriority();
            }
            // 本地方法
            setPriority0(priority = newPriority);
        }
    }
    //获取优先级
    public final int getPriority() {
            return priority;
    }
    //join方法会让线程进入WAITING，等待另一个线程的终止，整个方法和Object.wait方法也是很像，而且实现中也用到了wait，既然用到了wait方法，自然也会释放调用对象的监视器锁
    public final synchronized void join(long millis) throws InterruptedException {
        long base = System.currentTimeMillis();
        long now = 0;
    
        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }
    
        if (millis == 0) {
            // 判断调用join的线程是否活着，这里的活着是指RUNNABLE,BLOCKED,WAITING,TIMED_WAITING这四种状态，如果活着就一直等着，wait(0)意味着无限等
            while (isAlive()) {
                wait(0);
            }
        } else {
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
    
    // 纳秒级别控制
    public final synchronized void join(long millis, int nanos)
        throws InterruptedException {
    
        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }
    
        if (nanos < 0 || nanos > 999999) {
            throw new IllegalArgumentException(
                "nanosecond timeout value out of range");
        }
    
        if (nanos >= 500000 || (nanos != 0 && millis == 0)) {
            millis++;
        }
    
        join(millis);
    }
    
    public final void join() throws InterruptedException {
        join(0);
    }
}
```
