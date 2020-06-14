# JDK源码解析
[杂谈篇之我是怎么读源码的，授之以渔](https://www.cnblogs.com/youzhibing/p/9553752.html)
>> 首先我们要对我们的目标有所了解，知道她有什么特点，有些什么功能。
>> 最好的学习的方式就是模仿，接下来才是创造。
## Java基础
### java.util.Objects 
[java.util.Objects 简介](https://blog.csdn.net/lkforce/article/details/56289349)
```java
package java.util;
import java.util.function.Supplier;
public final class Objects {
    private Objects() {
        throw new AssertionError("No java.util.Objects instances for you!");
    }
    // 比较对象a和对象b，使用的是第一个参数的equals()方法，
    // 如果两个参数中有一个是null，则返回false，如果两个参数都是null，则返回true。
    public static boolean equals(Object a, Object b) {
            return (a == b) || (a != null && a.equals(b));
    }
    // 比较对象a和对象b是否深度相等，使用的其实是Arrays.deepEquals()方法
    // 只有a和b对应位置的元素都相等时，才返回true，a好b都是null也返回true，否则返回false
    public static boolean deepEquals(Object a, Object b) {
            if (a == b)
                return true;
            else if (a == null || b == null)
                return false;
            else
                return Arrays.deepEquals0(a, b);
    }
    // 得到一个对象的hash code，如果参数为null，返回0
    public static int hashCode(Object o) {
            return o != null ? o.hashCode() : 0;
    }
    // 为输入值序列生成哈希码。
    public static int hash(Object... values) {
            return Arrays.hashCode(values);
    }
    // 返回非 null参数调用 toString和 "null"参数的 "null"的 null 。 
     public static String toString(Object o) {
            return String.valueOf(o);
    }
    // 如果第一个参数不是 null ，则返回第一个参数调用 toString的结果， toString返回第二个参数。 
    public static String toString(Object o, String nullDefault) {
            return (o != null) ? o.toString() : nullDefault;
    }
    // 检查指定的对象引用不是 null 。 
    public static <T> T requireNonNull(T obj) {
            if (obj == null)
                throw new NullPointerException();
            return obj;
    }
    // 检查指定的对象引用不是null并抛出自定义的NullPointerException（如果是）。 
    public static <T> T requireNonNull(T obj, String message) {
            if (obj == null)
                throw new NullPointerException(message);
            return obj;
    }
    // 返回 true如果提供的引用是 null否则返回 false 。
    public static boolean isNull(Object obj) {
            return obj == null;
    }
    // 返回 true如果提供的引用不是 null否则返回 false 。 
    public static boolean nonNull(Object obj) {
            return obj != null;
    }
    // 检查指定的对象引用不是null并抛出一个自定义的NullPointerException（如果是）。 
    public static <T> T requireNonNull(T obj, Supplier<String> messageSupplier) {
            if (obj == null)
                throw new NullPointerException(messageSupplier.get());
            return obj;
    }
}
```
### java.lang.String 
[Java源码之String](https://www.cnblogs.com/chentang/p/13067765.html)
```markdown
继承三个接口:Comparable接口、CharSequence接口、Serializable接口。  【留个坑：String为什么实现Charsequence这个接口呢】
构造方法：本身相关，字节相关，字符相关。
String常用的方法大概有60多个。在java的String操作中，大多数情况下还是对char[]数组的操作。
[String 中的 equals 是如何重写的](https://www.cnblogs.com/cxuanBlog/p/13099235.html)
```
## Java容器集合
Iterable——>Collection——>Queue——>BlockingQueue
### java.lang.Iterable接口
```java
public interface Iterable<T> {
    // 返回类型为 T元素的迭代器。 
    Iterator<T> iterator();
    // 对Iterable的每个元素执行给定的操作，直到所有元素都被处理或动作引发异常。 JDK1.8
    default void forEach(Consumer<? super T> action) {
            Objects.requireNonNull(action);
            for (T t : this) {
                action.accept(t);
            }
    }
    // 在Iterable描述的元素上创建一个Iterable 。 JDK1.8
    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
}
```
### java.util.Collection接口
```java
public interface Collection<E> extends Iterable<E> {
    // 返回此集合中的元素数。 如果此收藏包含超过Integer.MAX_VALUE个元素，则返回Integer.MAX_VALUE 。 
    int size();
    // 如果此集合不包含元素，则返回 true 。true如果此集合不包含元素 
    boolean isEmpty();
    // 如果此集合包含指定的元素，则返回true 。
    boolean contains(Object o);
    // 返回此集合中的元素的迭代器。 没有关于元素返回顺序的保证（除非这个集合是提供保证的某个类的实例）。
    Iterator<E> iterator();
    // 返回一个包含此集合中所有元素的数组。 如果此集合对其迭代器返回的元素的顺序做出任何保证，则此方法必须以相同的顺序返回元素。 
    Object[] toArray();
    // 返回包含此集合中所有元素的数组; 返回的数组的运行时类型是指定数组的运行时类型。 如果集合适合指定的数组，则返回其中。 
    // 否则，将为指定数组的运行时类型和此集合的大小分配一个新数组。
    <T> T[] toArray(T[] a);
    // 确保此集合包含指定的元素（可选操作）。 
    boolean add(E e);
    // 从该集合中删除指定元素的单个实例（如果存在）（可选操作）。 
    boolean remove(Object o);
    // 如果此集合包含指定 集合中的所有元素，则返回true。 
    boolean containsAll(Collection<?> c);
    // 将指定集合中的所有元素添加到此集合（可选操作）。 
    boolean addAll(Collection<? extends E> c);
    // 删除指定集合中包含的所有此集合的元素（可选操作）。 
    boolean removeAll(Collection<?> c);
    // 删除满足给定谓词的此集合的所有元素。 
    default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while (each.hasNext()) {
            if (filter.test(each.next())) {
                each.remove();
                removed = true;
            }
        }
        return removed;
    }
    // 仅保留此集合中包含在指定集合中的元素（可选操作）。 
    boolean retainAll(Collection<?> c);
    // 从此集合中删除所有元素（可选操作）。 
    void clear();
    // 将指定的对象与此集合进行比较以获得相等性。 
    boolean equals(Object o);
    // 返回此集合的哈希码值。 
    int hashCode();
    // 创建一个Spliterator在这个集合中的元素。 
    @Override
    default Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, 0);
    }
    // 返回以此集合作为源的顺序 Stream 。 
    default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }
    // 返回可能并行的 Stream与此集合作为其来源。 
    default Stream<E> parallelStream() {
        return StreamSupport.stream(spliterator(), true);
    }
}
```
### java.util.Queue接口
```java
public interface Queue<E> extends Collection<E> {
    // 将指定的元素插入到此队列中，如果可以立即执行此操作，而不会违反容量限制， true在成功后返回 
    // IllegalStateException如果当前没有可用空间，则抛出IllegalStateException。 
    boolean add(E e);
    // 如果在不违反容量限制的情况下立即执行，则将指定的元素插入到此队列中。 
    boolean offer(E e);
    // 检索并删除此队列的头。 
    E remove();
    // 检索并删除此队列的头，如果此队列为空，则返回 null 。 
    E poll();
    // 检索，但不删除，这个队列的头。 
    E element();
    // 检索但不删除此队列的头，如果此队列为空，则返回 null 。 
    E peek();
}
```
### java.util.concurrent.BlockingQueue接口
```java
//阻塞队列接口  继承Queue接口，Queue接口继承Collection接口，Collection接口继承Iterable接口，Iterable是所有的容器集合的父接口
public interface BlockingQueue<E> extends Queue<E> {
        // 将指定的元素插入到此队列中，如果可以立即执行此操作而不违反容量限制， true在成功后返回 
        // IllegalStateException如果当前没有可用空间，则抛出IllegalStateException。 
        boolean add(E e);
        // 将指定的元素插入到此队列中，等待指定的等待时间（如有必要）才能使空间变得可用。 
        boolean offer(E e);
        
        void put(E e) throws InterruptedException;
        // 将指定的元素插入到此队列中，等待指定的等待时间（如有必要）才能使空间变得可用。
        boolean offer(E e, long timeout, TimeUnit unit)
                throws InterruptedException;
        // 检索并删除此队列的头，如有必要，等待元素可用。 
        E take() throws InterruptedException;
        // 检索并删除此队列的头，等待指定的等待时间（如有必要）使元素变为可用。 
        E poll(long timeout, TimeUnit unit)
                throws InterruptedException;
        // 返回该队列最好可以（在没有存储器或资源约束）接受而不会阻塞，或附加的元素的数量 Integer.MAX_VALUE如果没有固有的限制。 
        int remainingCapacity();
        // 从该队列中删除指定元素的单个实例（如果存在）。 
        boolean remove(Object o);
        // 如果此队列包含指定的元素，则返回 true 。 
        public boolean contains(Object o);
        // 从该队列中删除所有可用的元素，并将它们添加到给定的集合中。
        int drainTo(Collection<? super E> c);
        // 最多从该队列中删除给定数量的可用元素，并将它们添加到给定的集合中。 
        int drainTo(Collection<? super E> c, int maxElements);
}
```
### java.util.concurrent.LinkedBlockingQueue类
#### [Java 经典面试题：聊一聊 JUC 下的 LinkedBlockingQueue](https://www.cnblogs.com/jamaler/p/12849927.html)
```java
public class LinkedBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, java.io.Serializable {
    private static final long serialVersionUID = -6903933977591709194L;
    
    static class Node<E> {
            E item;
            Node<E> next;
            Node(E x) { item = x; }
    }
        private final int capacity;
    
        private final AtomicInteger count = new AtomicInteger();
    
        transient Node<E> head;

        private transient Node<E> last;
    
        private final ReentrantLock takeLock = new ReentrantLock();
    
        private final Condition notEmpty = takeLock.newCondition();
    
        private final ReentrantLock putLock = new ReentrantLock();
    
        private final Condition notFull = putLock.newCondition();
   
        private void signalNotEmpty() {
            final ReentrantLock takeLock = this.takeLock;
            takeLock.lock();
            try {
                notEmpty.signal();
            } finally {
                takeLock.unlock();
            }
        }
    
        private void signalNotFull() {
            final ReentrantLock putLock = this.putLock;
            putLock.lock();
            try {
                notFull.signal();
            } finally {
                putLock.unlock();
            }
        }

        private void enqueue(Node<E> node) {
            // assert putLock.isHeldByCurrentThread();
            // assert last.next == null;
            last = last.next = node;
        }
    
        private E dequeue() {
            // assert takeLock.isHeldByCurrentThread();
            // assert head.item == null;
            Node<E> h = head;
            Node<E> first = h.next;
            h.next = h; // help GC
            head = first;
            E x = first.item;
            first.item = null;
            return x;
        }
    
        void fullyLock() {
            putLock.lock();
            takeLock.lock();
        }
    
        /**
         * Unlocks to allow both puts and takes.
         */
        void fullyUnlock() {
            takeLock.unlock();
            putLock.unlock();
        }
    
    //     /**
    //      * Tells whether both locks are held by current thread.
    //      */
    //     boolean isFullyLocked() {
    //         return (putLock.isHeldByCurrentThread() &&
    //                 takeLock.isHeldByCurrentThread());
    //     }
    
        public LinkedBlockingQueue() {
            this(Integer.MAX_VALUE);
        }

        public LinkedBlockingQueue(int capacity) {
            if (capacity <= 0) throw new IllegalArgumentException();
            this.capacity = capacity;
            last = head = new Node<E>(null);
        }
    
       
        public LinkedBlockingQueue(Collection<? extends E> c) {
            this(Integer.MAX_VALUE);
            final ReentrantLock putLock = this.putLock;
            putLock.lock(); // Never contended, but necessary for visibility
            try {
                int n = 0;
                for (E e : c) {
                    if (e == null)
                        throw new NullPointerException();
                    if (n == capacity)
                        throw new IllegalStateException("Queue full");
                    enqueue(new Node<E>(e));
                    ++n;
                }
                count.set(n);
            } finally {
                putLock.unlock();
            }
        }
    
        
        public int size() {
            return count.get();
        }
    
        public int remainingCapacity() {
            return capacity - count.get();
        }

        public void put(E e) throws InterruptedException {
            if (e == null) throw new NullPointerException();
            // Note: convention in all put/take/etc is to preset local var
            // holding count negative to indicate failure unless set.
            int c = -1;
            Node<E> node = new Node<E>(e);
            final ReentrantLock putLock = this.putLock;
            final AtomicInteger count = this.count;
            putLock.lockInterruptibly();
            try {
                /*
                 * Note that count is used in wait guard even though it is
                 * not protected by lock. This works because count can
                 * only decrease at this point (all other puts are shut
                 * out by lock), and we (or some other waiting put) are
                 * signalled if it ever changes from capacity. Similarly
                 * for all other uses of count in other wait guards.
                 */
                while (count.get() == capacity) {
                    notFull.await();
                }
                enqueue(node);
                c = count.getAndIncrement();
                if (c + 1 < capacity)
                    notFull.signal();
            } finally {
                putLock.unlock();
            }
            if (c == 0)
                signalNotEmpty();
        }
    
        /**
         * Inserts the specified element at the tail of this queue, waiting if
         * necessary up to the specified wait time for space to become available.
         *
         * @return {@code true} if successful, or {@code false} if
         *         the specified waiting time elapses before space is available
         * @throws InterruptedException {@inheritDoc}
         * @throws NullPointerException {@inheritDoc}
         */
        public boolean offer(E e, long timeout, TimeUnit unit)
            throws InterruptedException {
    
            if (e == null) throw new NullPointerException();
            long nanos = unit.toNanos(timeout);
            int c = -1;
            final ReentrantLock putLock = this.putLock;
            final AtomicInteger count = this.count;
            putLock.lockInterruptibly();
            try {
                while (count.get() == capacity) {
                    if (nanos <= 0)
                        return false;
                    nanos = notFull.awaitNanos(nanos);
                }
                enqueue(new Node<E>(e));
                c = count.getAndIncrement();
                if (c + 1 < capacity)
                    notFull.signal();
            } finally {
                putLock.unlock();
            }
            if (c == 0)
                signalNotEmpty();
            return true;
        }

        public boolean offer(E e) {
            if (e == null) throw new NullPointerException();
            final AtomicInteger count = this.count;
            if (count.get() == capacity)
                return false;
            int c = -1;
            Node<E> node = new Node<E>(e);
            final ReentrantLock putLock = this.putLock;
            putLock.lock();
            try {
                if (count.get() < capacity) {
                    enqueue(node);
                    c = count.getAndIncrement();
                    if (c + 1 < capacity)
                        notFull.signal();
                }
            } finally {
                putLock.unlock();
            }
            if (c == 0)
                signalNotEmpty();
            return c >= 0;
        }
    
        public E take() throws InterruptedException {
            E x;
            int c = -1;
            final AtomicInteger count = this.count;
            final ReentrantLock takeLock = this.takeLock;
            takeLock.lockInterruptibly();
            try {
                while (count.get() == 0) {
                    notEmpty.await();
                }
                x = dequeue();
                c = count.getAndDecrement();
                if (c > 1)
                    notEmpty.signal();
            } finally {
                takeLock.unlock();
            }
            if (c == capacity)
                signalNotFull();
            return x;
        }
    
        public E poll(long timeout, TimeUnit unit) throws InterruptedException {
            E x = null;
            int c = -1;
            long nanos = unit.toNanos(timeout);
            final AtomicInteger count = this.count;
            final ReentrantLock takeLock = this.takeLock;
            takeLock.lockInterruptibly();
            try {
                while (count.get() == 0) {
                    if (nanos <= 0)
                        return null;
                    nanos = notEmpty.awaitNanos(nanos);
                }
                x = dequeue();
                c = count.getAndDecrement();
                if (c > 1)
                    notEmpty.signal();
            } finally {
                takeLock.unlock();
            }
            if (c == capacity)
                signalNotFull();
            return x;
        }
    
        public E poll() {
            final AtomicInteger count = this.count;
            if (count.get() == 0)
                return null;
            E x = null;
            int c = -1;
            final ReentrantLock takeLock = this.takeLock;
            takeLock.lock();
            try {
                if (count.get() > 0) {
                    x = dequeue();
                    c = count.getAndDecrement();
                    if (c > 1)
                        notEmpty.signal();
                }
            } finally {
                takeLock.unlock();
            }
            if (c == capacity)
                signalNotFull();
            return x;
        }
    
        public E peek() {
            if (count.get() == 0)
                return null;
            final ReentrantLock takeLock = this.takeLock;
            takeLock.lock();
            try {
                Node<E> first = head.next;
                if (first == null)
                    return null;
                else
                    return first.item;
            } finally {
                takeLock.unlock();
            }
        }
    
        /**
         * Unlinks interior Node p with predecessor trail.
         */
        void unlink(Node<E> p, Node<E> trail) {
            // assert isFullyLocked();
            // p.next is not changed, to allow iterators that are
            // traversing p to maintain their weak-consistency guarantee.
            p.item = null;
            trail.next = p.next;
            if (last == p)
                last = trail;
            if (count.getAndDecrement() == capacity)
                notFull.signal();
        }

        public boolean remove(Object o) {
            if (o == null) return false;
            fullyLock();
            try {
                for (Node<E> trail = head, p = trail.next;
                     p != null;
                     trail = p, p = p.next) {
                    if (o.equals(p.item)) {
                        unlink(p, trail);
                        return true;
                    }
                }
                return false;
            } finally {
                fullyUnlock();
            }
        }

        public boolean contains(Object o) {
            if (o == null) return false;
            fullyLock();
            try {
                for (Node<E> p = head.next; p != null; p = p.next)
                    if (o.equals(p.item))
                        return true;
                return false;
            } finally {
                fullyUnlock();
            }
        }
    
        /**
         * Returns an array containing all of the elements in this queue, in
         * proper sequence.
         *
         * <p>The returned array will be "safe" in that no references to it are
         * maintained by this queue.  (In other words, this method must allocate
         * a new array).  The caller is thus free to modify the returned array.
         *
         * <p>This method acts as bridge between array-based and collection-based
         * APIs.
         *
         * @return an array containing all of the elements in this queue
         */
        public Object[] toArray() {
            fullyLock();
            try {
                int size = count.get();
                Object[] a = new Object[size];
                int k = 0;
                for (Node<E> p = head.next; p != null; p = p.next)
                    a[k++] = p.item;
                return a;
            } finally {
                fullyUnlock();
            }
        }
    
        /**
         * Returns an array containing all of the elements in this queue, in
         * proper sequence; the runtime type of the returned array is that of
         * the specified array.  If the queue fits in the specified array, it
         * is returned therein.  Otherwise, a new array is allocated with the
         * runtime type of the specified array and the size of this queue.
         *
         * <p>If this queue fits in the specified array with room to spare
         * (i.e., the array has more elements than this queue), the element in
         * the array immediately following the end of the queue is set to
         * {@code null}.
         *
         * <p>Like the {@link #toArray()} method, this method acts as bridge between
         * array-based and collection-based APIs.  Further, this method allows
         * precise control over the runtime type of the output array, and may,
         * under certain circumstances, be used to save allocation costs.
         *
         * <p>Suppose {@code x} is a queue known to contain only strings.
         * The following code can be used to dump the queue into a newly
         * allocated array of {@code String}:
         *
         *  <pre> {@code String[] y = x.toArray(new String[0]);}</pre>
         *
         * Note that {@code toArray(new Object[0])} is identical in function to
         * {@code toArray()}.
         *
         * @param a the array into which the elements of the queue are to
         *          be stored, if it is big enough; otherwise, a new array of the
         *          same runtime type is allocated for this purpose
         * @return an array containing all of the elements in this queue
         * @throws ArrayStoreException if the runtime type of the specified array
         *         is not a supertype of the runtime type of every element in
         *         this queue
         * @throws NullPointerException if the specified array is null
         */
        @SuppressWarnings("unchecked")
        public <T> T[] toArray(T[] a) {
            fullyLock();
            try {
                int size = count.get();
                if (a.length < size)
                    a = (T[])java.lang.reflect.Array.newInstance
                        (a.getClass().getComponentType(), size);
    
                int k = 0;
                for (Node<E> p = head.next; p != null; p = p.next)
                    a[k++] = (T)p.item;
                if (a.length > k)
                    a[k] = null;
                return a;
            } finally {
                fullyUnlock();
            }
        }
    
        public String toString() {
            fullyLock();
            try {
                Node<E> p = head.next;
                if (p == null)
                    return "[]";
    
                StringBuilder sb = new StringBuilder();
                sb.append('[');
                for (;;) {
                    E e = p.item;
                    sb.append(e == this ? "(this Collection)" : e);
                    p = p.next;
                    if (p == null)
                        return sb.append(']').toString();
                    sb.append(',').append(' ');
                }
            } finally {
                fullyUnlock();
            }
        }
    
        /**
         * Atomically removes all of the elements from this queue.
         * The queue will be empty after this call returns.
         */
        public void clear() {
            fullyLock();
            try {
                for (Node<E> p, h = head; (p = h.next) != null; h = p) {
                    h.next = h;
                    p.item = null;
                }
                head = last;
                // assert head.item == null && head.next == null;
                if (count.getAndSet(0) == capacity)
                    notFull.signal();
            } finally {
                fullyUnlock();
            }
        }
    
        /**
         * @throws UnsupportedOperationException {@inheritDoc}
         * @throws ClassCastException            {@inheritDoc}
         * @throws NullPointerException          {@inheritDoc}
         * @throws IllegalArgumentException      {@inheritDoc}
         */
        public int drainTo(Collection<? super E> c) {
            return drainTo(c, Integer.MAX_VALUE);
        }
    
        /**
         * @throws UnsupportedOperationException {@inheritDoc}
         * @throws ClassCastException            {@inheritDoc}
         * @throws NullPointerException          {@inheritDoc}
         * @throws IllegalArgumentException      {@inheritDoc}
         */
        public int drainTo(Collection<? super E> c, int maxElements) {
            if (c == null)
                throw new NullPointerException();
            if (c == this)
                throw new IllegalArgumentException();
            if (maxElements <= 0)
                return 0;
            boolean signalNotFull = false;
            final ReentrantLock takeLock = this.takeLock;
            takeLock.lock();
            try {
                int n = Math.min(maxElements, count.get());
                // count.get provides visibility to first n Nodes
                Node<E> h = head;
                int i = 0;
                try {
                    while (i < n) {
                        Node<E> p = h.next;
                        c.add(p.item);
                        p.item = null;
                        h.next = h;
                        h = p;
                        ++i;
                    }
                    return n;
                } finally {
                    // Restore invariants even if c.add() threw
                    if (i > 0) {
                        // assert h.item == null;
                        head = h;
                        signalNotFull = (count.getAndAdd(-i) == capacity);
                    }
                }
            } finally {
                takeLock.unlock();
                if (signalNotFull)
                    signalNotFull();
            }
        }
    
        /**
         * Returns an iterator over the elements in this queue in proper sequence.
         * The elements will be returned in order from first (head) to last (tail).
         *
         * <p>The returned iterator is
         * <a href="package-summary.html#Weakly"><i>weakly consistent</i></a>.
         *
         * @return an iterator over the elements in this queue in proper sequence
         */
        public Iterator<E> iterator() {
            return new Itr();
        }
    
        private class Itr implements Iterator<E> {
            /*
             * Basic weakly-consistent iterator.  At all times hold the next
             * item to hand out so that if hasNext() reports true, we will
             * still have it to return even if lost race with a take etc.
             */
    
            private Node<E> current;
            private Node<E> lastRet;
            private E currentElement;
    
            Itr() {
                fullyLock();
                try {
                    current = head.next;
                    if (current != null)
                        currentElement = current.item;
                } finally {
                    fullyUnlock();
                }
            }
    
            public boolean hasNext() {
                return current != null;
            }
    
            /**
             * Returns the next live successor of p, or null if no such.
             *
             * Unlike other traversal methods, iterators need to handle both:
             * - dequeued nodes (p.next == p)
             * - (possibly multiple) interior removed nodes (p.item == null)
             */
            private Node<E> nextNode(Node<E> p) {
                for (;;) {
                    Node<E> s = p.next;
                    if (s == p)
                        return head.next;
                    if (s == null || s.item != null)
                        return s;
                    p = s;
                }
            }
    
            public E next() {
                fullyLock();
                try {
                    if (current == null)
                        throw new NoSuchElementException();
                    E x = currentElement;
                    lastRet = current;
                    current = nextNode(current);
                    currentElement = (current == null) ? null : current.item;
                    return x;
                } finally {
                    fullyUnlock();
                }
            }
    
            public void remove() {
                if (lastRet == null)
                    throw new IllegalStateException();
                fullyLock();
                try {
                    Node<E> node = lastRet;
                    lastRet = null;
                    for (Node<E> trail = head, p = trail.next;
                         p != null;
                         trail = p, p = p.next) {
                        if (p == node) {
                            unlink(p, trail);
                            break;
                        }
                    }
                } finally {
                    fullyUnlock();
                }
            }
        }
    
        /** A customized variant of Spliterators.IteratorSpliterator */
        static final class LBQSpliterator<E> implements Spliterator<E> {
            static final int MAX_BATCH = 1 << 25;  // max batch array size;
            final LinkedBlockingQueue<E> queue;
            Node<E> current;    // current node; null until initialized
            int batch;          // batch size for splits
            boolean exhausted;  // true when no more nodes
            long est;           // size estimate
            LBQSpliterator(LinkedBlockingQueue<E> queue) {
                this.queue = queue;
                this.est = queue.size();
            }
    
            public long estimateSize() { return est; }
    
            public Spliterator<E> trySplit() {
                Node<E> h;
                final LinkedBlockingQueue<E> q = this.queue;
                int b = batch;
                int n = (b <= 0) ? 1 : (b >= MAX_BATCH) ? MAX_BATCH : b + 1;
                if (!exhausted &&
                    ((h = current) != null || (h = q.head.next) != null) &&
                    h.next != null) {
                    Object[] a = new Object[n];
                    int i = 0;
                    Node<E> p = current;
                    q.fullyLock();
                    try {
                        if (p != null || (p = q.head.next) != null) {
                            do {
                                if ((a[i] = p.item) != null)
                                    ++i;
                            } while ((p = p.next) != null && i < n);
                        }
                    } finally {
                        q.fullyUnlock();
                    }
                    if ((current = p) == null) {
                        est = 0L;
                        exhausted = true;
                    }
                    else if ((est -= i) < 0L)
                        est = 0L;
                    if (i > 0) {
                        batch = i;
                        return Spliterators.spliterator
                            (a, 0, i, Spliterator.ORDERED | Spliterator.NONNULL |
                             Spliterator.CONCURRENT);
                    }
                }
                return null;
            }
    
            public void forEachRemaining(Consumer<? super E> action) {
                if (action == null) throw new NullPointerException();
                final LinkedBlockingQueue<E> q = this.queue;
                if (!exhausted) {
                    exhausted = true;
                    Node<E> p = current;
                    do {
                        E e = null;
                        q.fullyLock();
                        try {
                            if (p == null)
                                p = q.head.next;
                            while (p != null) {
                                e = p.item;
                                p = p.next;
                                if (e != null)
                                    break;
                            }
                        } finally {
                            q.fullyUnlock();
                        }
                        if (e != null)
                            action.accept(e);
                    } while (p != null);
                }
            }
    
            public boolean tryAdvance(Consumer<? super E> action) {
                if (action == null) throw new NullPointerException();
                final LinkedBlockingQueue<E> q = this.queue;
                if (!exhausted) {
                    E e = null;
                    q.fullyLock();
                    try {
                        if (current == null)
                            current = q.head.next;
                        while (current != null) {
                            e = current.item;
                            current = current.next;
                            if (e != null)
                                break;
                        }
                    } finally {
                        q.fullyUnlock();
                    }
                    if (current == null)
                        exhausted = true;
                    if (e != null) {
                        action.accept(e);
                        return true;
                    }
                }
                return false;
            }
    
            public int characteristics() {
                return Spliterator.ORDERED | Spliterator.NONNULL |
                    Spliterator.CONCURRENT;
            }
        }
    
        /**
         * Returns a {@link Spliterator} over the elements in this queue.
         *
         * <p>The returned spliterator is
         * <a href="package-summary.html#Weakly"><i>weakly consistent</i></a>.
         *
         * <p>The {@code Spliterator} reports {@link Spliterator#CONCURRENT},
         * {@link Spliterator#ORDERED}, and {@link Spliterator#NONNULL}.
         *
         * @implNote
         * The {@code Spliterator} implements {@code trySplit} to permit limited
         * parallelism.
         *
         * @return a {@code Spliterator} over the elements in this queue
         * @since 1.8
         */
        public Spliterator<E> spliterator() {
            return new LBQSpliterator<E>(this);
        }
    
        /**
         * Saves this queue to a stream (that is, serializes it).
         *
         * @param s the stream
         * @throws java.io.IOException if an I/O error occurs
         * @serialData The capacity is emitted (int), followed by all of
         * its elements (each an {@code Object}) in the proper order,
         * followed by a null
         */
        private void writeObject(java.io.ObjectOutputStream s)
            throws java.io.IOException {
    
            fullyLock();
            try {
                // Write out any hidden stuff, plus capacity
                s.defaultWriteObject();
    
                // Write out all elements in the proper order.
                for (Node<E> p = head.next; p != null; p = p.next)
                    s.writeObject(p.item);
    
                // Use trailing null as sentinel
                s.writeObject(null);
            } finally {
                fullyUnlock();
            }
        }
    
        /**
         * Reconstitutes this queue from a stream (that is, deserializes it).
         * @param s the stream
         * @throws ClassNotFoundException if the class of a serialized object
         *         could not be found
         * @throws java.io.IOException if an I/O error occurs
         */
        private void readObject(java.io.ObjectInputStream s)
            throws java.io.IOException, ClassNotFoundException {
            // Read in capacity, and any hidden stuff
            s.defaultReadObject();
    
            count.set(0);
            last = head = new Node<E>(null);
    
            // Read in all elements and place in queue
            for (;;) {
                @SuppressWarnings("unchecked")
                E item = (E)s.readObject();
                if (item == null)
                    break;
                add(item);
            }
        }
}
```
## Java并发多线程

##
[当前标签：JDK源码解析](https://www.cnblogs.com/ysocean/tag/JDK%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/)

[深入理解，源码解析 Integer 神奇的老哥](https://www.cnblogs.com/huasheng/p/9995543.html)

[随笔分类 - 08. Java基础类型](https://www.cnblogs.com/noteless/category/1307424.html)
