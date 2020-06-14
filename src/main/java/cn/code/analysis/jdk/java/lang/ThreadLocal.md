# java.lang.ThreadLocal
```java
package java.lang;

public class ThreadLocal<T> {
    
    // hash值，类似于Hashmap，用于计算放在map内部数组的哪个index上
    private final int threadLocalHashCode = nextHashCode();
    private static int nextHashCode() { return nextHashCode.getAndAdd(HASH_INCREMENT);}
	// 初始0
    private static AtomicInteger nextHashCode = new AtomicInteger();
	// 神奇的值，这个hash值的倍数去计算index，分布会很均匀，总之很6 
    private static final int HASH_INCREMENT = 0x61c88647;
    
    static class ThreadLocalMap {

        // 注意这是一个弱引用
        static class Entry extends WeakReference<ThreadLocal<?>> {
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
        // 初始容量16，一定要是2的倍数
        private static final int INITIAL_CAPACITY = 16;
        // map内部数组
        private Entry[] table;
        // 当前储存的数量
        private int size = 0;
        // 扩容指标，计算公式 threshold = 总容量 * 2 / 3，默认初始化之后为10
        private int threshold;
    }
    //增改方法
    public void set(T value) {
        Thread t = Thread.currentThread();
        // 拿到当前Thread对象中的threadLocals引用，默认threadLocals值是null 
        ThreadLocalMap map = getMap(t);
        if (map != null)
            // 如果ThreadLocalMap已经初始化过，就把当前ThreadLocal实例的引用当key，设置值
            map.set(this, value); //下文详解
        else
            // 如果不存在就创建一个ThreadLocalMap并且提供初始值
            createMap(t, value);
    }
    
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
    
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
    //map.set(this, value)具体怎么操作ThreadLocalMap
    private void set(ThreadLocal<?> key, Object value) {
    	// 获取ThreadLocalMap内部数组
        Entry[] tab = table;
        int len = tab.length;
        // 算出需要放在哪个桶里
        int i = key.threadLocalHashCode & (len-1);
    	// 如果当前桶冲突了，这里没有用拉链法，而是使用开放定指法，index递增直到找到空桶，数据量很小的情况这样效率高
        for (Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
            // 拿到目前桶中key
            ThreadLocal<?> k = e.get();
    		// 如果桶中key和我们要set的key一样，直接更新值就ok了
            if (k == key) {
                e.value = value;
                return;
            }
    		// 桶中key是null，因为是弱引用，可能被回收掉了，这个时候我们直接占为己有，并且进行cleanSomeSlots，当前key附近局部清理其他key是空的桶
            if (k == null) {
                replaceStaleEntry(key, value, i);
                return;
            }
        }
    	// 如果没冲突直接新建
        tab[i] = new Entry(key, value);
        int sz = ++size;
        // 当前key附近局部清理key是空的桶，如果一个也没清除并且当前容量超过阈值了就扩容
        if (!cleanSomeSlots(i, sz) && sz >= threshold)
            rehash();
    }
    
    private void rehash() {
        // 这个方法会清除所有key为null的桶，清理完后size的大小会变小
        expungeStaleEntries();
    
        // 此时size还大于阈值的3/4就扩容
        if (size >= threshold - threshold / 4)
            // 2倍扩容
            resize();
    }
    //查询操作
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);  //下文详解
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        // 返回null
        return setInitialValue();
    }
    
    private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            // 如果只是threadLocals.Entry是空，就设置value为null
            map.set(this, value);
        else
            // 如果threadLocals是空，就new 一个key是当前ThreadLocal，value是空的ThreadLocalMap
            createMap(t, value);
        return value;
    }
    
    protected T initialValue() {
        return null;
    }
    // map.getEntry(this)具体怎么操作ThreadLocalMap
    private Entry getEntry(ThreadLocal<?> key) {
        int i = key.threadLocalHashCode & (table.length - 1);
        Entry e = table[i];
        if (e != null && e.get() == key)
            // 最好情况，定位到了Entry，并且key匹配
            return e;
        else
            // 可能是hash冲突重定址了，也可能是key被回收了
            return getEntryAfterMiss(key, i, e);
    }
    
    private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
        Entry[] tab = table;
        int len = tab.length;
    	// 向后遍历去匹配key，同时清除key为null的桶
        while (e != null) {
            ThreadLocal<?> k = e.get();
            if (k == key)
                return e;
            if (k == null)
                expungeStaleEntry(i);
            else
                i = nextIndex(i, len);
            e = tab[i];
        }
        return null;
    }
    //如何避免内存泄漏  养成良好习惯，只要使用完ThreadLocal，一定要进行remove防止内存泄漏
    public void remove() {
        ThreadLocalMap m = getMap(Thread.currentThread());
        if (m != null)
            m.remove(this);
    }
    
    private void remove(ThreadLocal<?> key) {
        Entry[] tab = table;
        int len = tab.length;
        int i = key.threadLocalHashCode & (len-1);
        for (Entry e = tab[i]; e != null; e = tab[i = nextIndex(i, len)]) {
            if (e.get() == key) {
                // 主要多了这一步，让this.referent = null，GC会提供特殊处理
                e.clear();
                expungeStaleEntry(i);
                return;
            }
        }
    }
}
```
