# 集合的分类
## 线性集合
```shell
list：  有序列表，元素可重复。
set:    无需列表，元素不可重复。
statck: 栈，    先进后出。filo
deque:  队列,   先进先出。fifo
```
## 非线性集合
```
map: 键值对 key-vaule
```

## list 的常见实现
### ArrayList (动态可扩容的数组)
类结构定义：
``` java
    public class ArrayList<E> extends AbstractList<E>
            implements List<E>, RandomAccess, Cloneable, java.io.Serializable
    {
        private static final long serialVersionUID = 8683452581122892189L;

        /**
        * Default initial capacity.
        */
        private static final int DEFAULT_CAPACITY = 10;

        /**
        * Shared empty array instance used for empty instances.
        */
        private static final Object[] EMPTY_ELEMENTDATA = {};

        /**
        * Shared empty array instance used for default sized empty instances. We
        * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
        * first element is added.
        */
        private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

        /**
        * The array buffer into which the elements of the ArrayList are stored.
        * The capacity of the ArrayList is the length of this array buffer. Any
        * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
        * will be expanded to DEFAULT_CAPACITY when the first element is added.
        */
        transient Object[] elementData; // non-private to simplify nested class access

        /**
        * The size of the ArrayList (the number of elements it contains).
        *
        * @serial
        */
        private int size;
    }

```
初始化规则：
``` java
    /***
        note: 
        如果初始化容量 > 0 就开辟一个对应容量的空数组。否则就初始化空对象
    ***/
    private static final Object[] EMPTY_ELEMENTDATA = {};
    ...
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
```
扩容规则：
``` java
    /***
        note: 
        newCapacity = oldCapacity + (oldCapacity >> 1); 新容量 = 旧容量 + 旧容量 * 0.5
        elementData = Arrays.copyOf(elementData, newCapacity); 将旧的数组拷贝到新的数组里
    ***/
    private void grow(int minCapacity) {
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        elementData = Arrays.copyOf(elementData, newCapacity);
    }

```

### LinkedList(双向链表)
类结构定义：
``` java
    public class LinkedList<E>
        extends AbstractSequentialList<E>
        implements List<E>, Deque<E>, Cloneable, java.io.Serializable
    {
        transient int size = 0;

        /**
        * Pointer to first node.
        * Invariant: (first == null && last == null) ||
        *            (first.prev == null && first.item != null)
        */
        transient Node<E> first;

        /**
        * Pointer to last node.
        * Invariant: (first == null && last == null) ||
        *            (last.next == null && last.item != null)
        */
        transient Node<E> last;
    }

```

初始化规则：
``` java
无
```

扩容规则：
``` java
    /***
        始终添加到队尾
    ***/
    public boolean add(E e) {
        linkLast(e);
        return true;
    }
    void linkLast(E e) {
        final Node<E> l = last;
        final Node<E> newNode = new Node<>(l, e, null);
        last = newNode;
        if (l == null)
            first = newNode;
        else
            l.next = newNode;
        size++;
        modCount++;
    }    
```
查找规则：
``` java
    /***
        二分查找
    ***/
    Node<E> node(int index) {
        // assert isElementIndex(index);
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }
```

## set 的常见实现
### HashSet (无序列表)
类结构定义：
``` java
    /***
        对内部是由HashMap实现的
    ***/
    public class HashSet<E>
        extends AbstractSet<E>
        implements Set<E>, Cloneable, java.io.Serializable
    {
        static final long serialVersionUID = -5024744406713321676L;

        private transient HashMap<E,Object> map;

        // Dummy value to associate with an Object in the backing Map
        private static final Object PRESENT = new Object();
    }
```
### TreeSet (树状列表)
``` java

    public class TreeSet<E> extends AbstractSet<E>
        implements NavigableSet<E>, Cloneable, java.io.Serializable
    {
        /**
        * The backing map.
        */
        private transient NavigableMap<E,Object> m;

        // Dummy value to associate with an Object in the backing Map
        private static final Object PRESENT = new Object();

        /**
        * Constructs a set backed by the specified navigable map.
        */
        TreeSet(NavigableMap<E,Object> m) {
            this.m = m;
        }
    }
```

## map 的常见实现

### HashMap (Hash列表)

类结构定义：

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {

    // 初始化容量
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

    // 最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30;

    // 加载因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    // 列表转红黑树阈值
    static final int TREEIFY_THRESHOLD = 8;

    // 红黑树转列表阈值
    static final int UNTREEIFY_THRESHOLD = 6;

    // 最小列表转红黑树阈值(单个列表长度)
    static final int MIN_TREEIFY_CAPACITY = 64;

    // HashMap 中存储的节点对象
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }
        // 计算 hashCode
        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }

    /* ---------------- Static utilities -------------- */
    // 计算 hash 值
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }

    // 实现 Comparable 接口
    static Class<?> comparableClassFor(Object x) {
        if (x instanceof Comparable) {
            Class<?> c; Type[] ts, as; Type t; ParameterizedType p;
            if ((c = x.getClass()) == String.class) // bypass checks
                return c;
            if ((ts = c.getGenericInterfaces()) != null) {
                for (int i = 0; i < ts.length; ++i) {
                    if (((t = ts[i]) instanceof ParameterizedType) &&
                        ((p = (ParameterizedType)t).getRawType() ==
                         Comparable.class) &&
                        (as = p.getActualTypeArguments()) != null &&
                        as.length == 1 && as[0] == c) // type arg is c
                        return c;
                }
            }
        }
        return null;
    }

    // 实现 Comparable 接口
    @SuppressWarnings({"rawtypes","unchecked"}) // for cast to Comparable
    static int compareComparables(Class<?> kc, Object k, Object x) {
        return (x == null || x.getClass() != kc ? 0 :
                ((Comparable)k).compareTo(x));
    }

 	// 扩容方法
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }

    /* ---------------- Fields -------------- */

    transient Node<K,V>[] table;

    transient Set<Map.Entry<K,V>> entrySet;

    transient int size;

    transient int modCount;

	// 阈值
    int threshold;

	// 加载因子，默认 0.75f
    final float loadFactor;
}
```

初始化规则：

```java
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        // 	初始化容量
        this.threshold = tableSizeFor(initialCapacity);
    }
```

扩容规则：

```java

public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; 
    Node<K,V> p; 
    int n, i;
    // 如果 tab 不存在 或者 tab 的 长度是 0，则 resize tab
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // 判断 是不是需要 开辟新的列表
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    
    else {
        Node<K,V> e; 
        K k;
        // 列表的 hash 值相同的话，p 赋值给 e
        if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 红黑树
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    // 写入新值
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        // 覆盖
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```
