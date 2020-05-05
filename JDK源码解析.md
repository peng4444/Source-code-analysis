# JDK源码解析
### java.lang.Object [Java面试系列第2篇-Object类中的方法](https://www.cnblogs.com/extjs4/p/12772027.html)
```java
public class Object {

    private static native void registerNatives();
    static {
        registerNatives();
    }
    // 返回此 Object的运行时类。
    public final native Class<?> getClass();
    // 返回对象的哈希码值。 
    public native int hashCode();
    // 指示一些其他对象是否等于此。 
    public boolean equals(Object obj) {
        return (this == obj);
    }
    // 创建并返回此对象的副本。 
    protected native Object clone() throws CloneNotSupportedException;
    // 返回对象的字符串表示形式。 
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
    // 唤醒正在等待对象监视器的单个线程。 
    public final native void notify();
    // 唤醒正在等待对象监视器的所有线程。
    public final native void notifyAll();
    // 导致当前线程等待，直到另一个线程调用 notify()方法或该对象的 notifyAll()方法，或者指定的时间已过。 
    public final native void wait(long timeout) throws InterruptedException;
    // 导致当前线程等待，直到另一个线程调用该对象的 notify()方法或 notifyAll()方法，或者某些其他线程中断当前线程，或一定量的实时时间。 
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
    // 导致当前线程等待，直到另一个线程调用该对象的 notify()方法或 notifyAll()方法。 
    public final void wait() throws InterruptedException {
        wait(0);
    }
    // 当垃圾收集确定不再有对该对象的引用时，垃圾收集器在对象上调用该对象。 
    protected void finalize() throws Throwable { }
}

```
### java.util.Objects [java.util.Objects 简介](https://blog.csdn.net/lkforce/article/details/56289349)
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
## 容器集合
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


[当前标签：JDK源码解析](https://www.cnblogs.com/ysocean/tag/JDK%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/)
