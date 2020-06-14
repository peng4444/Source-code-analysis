# java.util.concurrent.CountDownLatch
```java
public class CountDownLatch {
    /**
     * Synchronization control For CountDownLatch.
     * Uses AQS state to represent count.
     */
    private static final class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 4982264981922014374L;

        Sync(int count) {
            setState(count);
        }

        int getCount() {
            return getState();
        }

        protected int tryAcquireShared(int acquires) {
            return (getState() == 0) ? 1 : -1;
        }

        protected boolean tryReleaseShared(int releases) {
            // Decrement count; signal when transition to zero
            for (;;) {
                int c = getState();
                if (c == 0)// c == 0 说明栅栏已经打开过了，CountDownLatch 是一次性的，直接false
                    return false;
                int nextc = c-1;
                if (compareAndSetState(c, nextc))
                    return nextc == 0;// cas 递减状态，达到0的时候返回 true 栅栏打开
            }
        }
    }

    private final Sync sync;

     // Constructs a {@code CountDownLatch} initialized with the given count.
    public CountDownLatch(int count) {
        if (count < 0) throw new IllegalArgumentException("count < 0");
        this.sync = new Sync(count);//可以看到 CountDownLatc 内部也实现了一个 AQS
    }

     // Causes the current thread to wait until the latch has counted down to
     // zero, unless the thread is {@linkplain Thread#interrupt interrupted}.
    public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }

     // Causes the current thread to wait until the latch has counted down to
     // zero, unless the thread is {@linkplain Thread#interrupt interrupted},
     // or the specified waiting time elapses.
    public boolean await(long timeout, TimeUnit unit)
        throws InterruptedException {
        return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
    }

    // Decrements the count of the latch, releasing all waiting threads if the count reaches zero.
    public void countDown() {
        sync.releaseShared(1);
    }

    /**
     * Returns the current count.
     * <p>This method is typically used for debugging and testing purposes.
     * @return the current count
     */
    public long getCount() {
        return sync.getCount();
    }

    /**
     * Returns a string identifying this latch, as well as its state.
     * The state, in brackets, includes the String {@code "Count ="}
     * followed by the current count.
     * @return a string identifying this latch, as well as its state
     */
    public String toString() {
        return super.toString() + "[Count = " + sync.getCount() + "]";
    }
}
```
