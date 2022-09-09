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