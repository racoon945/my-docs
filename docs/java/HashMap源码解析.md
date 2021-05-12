## HashMap源码解析



### 初始化



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
        this.threshold = tableSizeFor(initialCapacity);
    }

    /**
     * Returns a power of two size for the given target capacity.
     * 保证返回一个比入参大的最小的2的正整数次幂
     */
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1; // 最高2位变成 1
        n |= n >>> 2; // 最高4位已经变成了1
        n |= n >>> 4; // 最高8位位已经变成了1
        n |= n >>> 8; // 最高16位已经变成了1
        n |= n >>> 16; // 32位全变为了1
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

> ```java
> // n |= n >>> 1;
>            n = 0000 001x xxxx xxxx
>      n >>> 1 = 0000 0001 xxxx xxxx
> n |= n >>> 1 = 0000 0011 xxxx xxxx 
> 
> // 可以看到此时n的二进制最高两位已经变成了1
> // 运行到 n |= n >>> 16 时
> ```
>
> ```java
> @Test
> public void test() {
> Integer x = tableSizeFor(18);
> 		System.out.println(x);
> }
> 
> Integer tableSizeFor(Integer cap) {
>     int n = cap - 1;
>     System.out.println(Integer.toBinaryString(n));
>     n |= n >>> 1;
>     System.out.println(">>>1: " + Integer.toBinaryString(n));
>     n |= n >>> 2;
>     System.out.println(">>>2: " + Integer.toBinaryString(n));
>     n |= n >>> 4;
>     System.out.println(">>>4: " + Integer.toBinaryString(n));
>     n |= n >>> 8;
>     System.out.println(">>>8: " + Integer.toBinaryString(n));
>     n |= n >>> 16;
>     return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
> }
> /*
> 输出结果为:
> 10001
> >>>1: 11001
> >>>2: 11111
> >>>4: 11111
> >>>8: 11111
> 32
> */
> ```
>
> 

### 数据寻址-hash方法

```java
    static final int hash(Object key) {
        int h;
        // Object#hashCode()为native方法
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

`(h = key.hashCode()) ^ (h >>> 16)`
hash值的低16位与高16位进行异或

```java
// 假设哈希值为  int h = -799451455;
// 则 h^(h>>>16) 如下
1101 0000 0101 1001 0101 0110 1100 0001
                                      ^
0000 0000 0000 0000 1101 0000 0101 1001
=                                      
1101 0000 0101 1001 1000 0110 1001 1000  
```

> table.length是一个2的正整数次幂，类似于000100000，这样的值减一就成了000011111，通过位运算可以高效寻址。
> 
> HashMap内部的bucket数组长度为什么一直都是2的整数次幂？好处之一就是可以通过构造位运算快速寻址定址。

数组下标 index `index = (table.length - 1) & hash`
既然计算出来的哈希值都要与`table.length - 1`做与运算，那就意味着计算出来的hash值只有低位有效，这样会加大碰撞几率，因此让高16位与低16位做异或，让低位保留部分高位信息，可以减少哈希碰撞。

数组容量n必为2的整数次幂 , 则n-1的二进制尾位全为1

```java
// 数组容量 n=16 ， key的哈希值 hash=-799451455 时 
// 则 (n-1)&hash 如下
1101 0000 0101 1001 1000 0110 1001 1000 
                                      &
0000 0000 0000 0000 0000 0000 0000 1111 
=
0000 0000 0000 0000 0000 0000 0000 1000
// 结果：index为8
```



### 数据存储-put方法

`p = tab[i = (n - 1) & hash]` 
i为HashMap数组下标, n为原数组长度
默认初始化过程中, n=16 

```java
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        HashMap.Node<K,V>[] tab; HashMap.Node<K,V> p; int n, i;
        // tab为原HashMap数组   n为原数组长度
        if ((tab = table) == null || (n = tab.length) == 0)
            // 原数组长度为0时, 为tab进行默认初始化, n=16
            n = (tab = resize()).length;
        // p为已确定位置的链表头节点tab[i]
        if ((p = tab[i = (n - 1) & hash]) == null)
            // tab[i] 不存在就直接放入此新节点
            tab[i] = newNode(hash, key, value, null);
        else {     // 链表头节点存在时
            // e为节点临时变量 指向目标节点 (用于临时存储p地址 待后续替换新值) , k为元素p的key
            HashMap.Node<K,V> e; K k;
            // == 比较的是变量(栈)内存中存放的对象的(堆)内存地址  equals 比较的是两个对象的内容是否相等
            // 因为hash值相等不一定对象就相等，hash碰撞
            // (已存在元素p的hash与传参hash引用地址相同 且 传参key与已存在元素p.key引用地址相同) 或 (传参Key非空且传参key与已存在元素p的key对象内容相同)
            if (p.hash == hash &&
                    ((k = p.key) == key || (key != null && key.equals(k))))
                e = p; 
            else if (p instanceof HashMap.TreeNode)  // 或者如果p为树节点
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);

            else { // 如果传参key与链表头节点的key未重复, 且非树
                // 链表迭代
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) { // 如果链表的下一个节点为空
                        // 新节点直接放入链表尾部
                        p.next = newNode(hash, key, value, null);
                        // 如果链表迭代次数>=8 , 链表转红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break; 
                    }
                    if (e.hash == hash &&
                            ((k = e.key) == key || (key != null && key.equals(k))))
                        // 如果节点已存在，则跳出循环
                        break;
                    // 将p.next赋值给p  指针后移，继续后循环
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                // 如果存在key的映射 , 仅当缺值才put值的设置为false 或 原value为空 将新值value赋值给 e
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        // 如果是新增值, size大于阈值就扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```



### 扩容-resize方法

扩容后的链表结点位置迁移规则
`(e.hash & oldCap) == 0` 用于判定e结点是高位结点还是低位结点
`newTab[j] = loHead;`          // 低位结点位置不变
`newTab[j + oldCap] = hiHead;` // 高位结点位置往后迁移oldCap个位置

数组容量oldCap必为2的整数次幂 , 则oldCap的二进制尾位全为0

```java
// 数组容量 oldCap = 16 , e.hash = -799451455 时 
// 则 (e.hash & oldCap) 如下
1101 0000 0101 1001 1000 0110 1001 1000 
                                      &
0000 0000 0000 0000 0000 0000 0001 0000 
=
0000 0000 0000 0000 0000 0000 0001 0000
// 结果不为0 ,则e为高位结点
```

> 桶数组长度为2的正整数幂的第二个优势：
>
> 当桶数组长度为2的正整数幂时，如果桶发生扩容（长度翻倍），则桶中的元素大概只有一半需要切换到新的桶中，另一半留在原先的桶中就可以，并且这个概率可以看做是均等的。

```java
final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) { // 非新创建的HashMap
            // MAXIMUM_CAPACITY= 1 << 30
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            // 新容量扩容1倍  DEFAULT_INITIAL_CAPACITY = 1 << 4
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                // 当旧容量大于等于16时 新扩容阈值增加一倍
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
        // 新创建并已指定 初始化容量和加载因子 (即扩容阈值)
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY; // 16
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY); // 12
        }
        if (newThr == 0) { // 新阈值为0就初始化为 新容量*加载因子
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr; // 启用新阈值
        // 为新数组开辟空间
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab; // 本方法第一行代码 Node<K,V>[] oldTab = table; 已将oldTab已指向原未扩容对象
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) { // 遍历之前的桶数组，对其值重新散列
                Node<K,V> e; 
                if ((e = oldTab[j]) != null) { // 旧桶内有值时
                    oldTab[j] = null;
                    if (e.next == null) // e为单结点(链表只有一个结点)
                        // 将结点置于扩容后的新桶中
                        newTab[e.hash & (newCap - 1)] = e; 
                    else if (e instanceof TreeNode)
                        // e 为树结点就切割
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null; // 低位头尾结点
                        Node<K,V> hiHead = null, hiTail = null; // 高位头尾结点
                        Node<K,V> next; 
                        // e为链表结点
                        do {
                            next = e.next; 
                            if ((e.hash & oldCap) == 0) { // 低位
                                if (loTail == null) // 低位尾指针无指向
                                    loHead = e; // 低位头指针指向当前e
                                else
                                    // 即低位尾指针所指向的上一个对象e,其next指向当前e
                                    loTail.next = e; 
                                loTail = e;  // 低位尾指针指向e
                            }
                            else { // 高位
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null); // e结点指针后移 该指针后移后指针指向null时打破循环
                        if (loTail != null) { // 尾指针指向非空
                            loTail.next = null;  
                            // 低位头指针指向地址赋值给扩容后数组 相对于原数组位置保持不变
                            newTab[j] = loHead; 
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            // 高位头指针指向地址赋值给扩容后数组 相对于原数组位置后移oldCap位
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```

 `((TreeNode<K,V>)e).split(this, newTab, j, oldCap);`
```java
       /**
         * Splits nodes in a tree bin into lower and upper tree bins,
         * or untreeifies if now too small. Called only from resize;
         * see above discussion about split bits and indices.
         *
         * @param map the map
         * @param tab the table for recording bin heads
         * @param index the index of the table being split
         * @param bit the bit of hash to split on
         */
        final void split(HashMap<K,V> map, Node<K,V>[] tab, int index, int bit) {
            // 桶内树结点
            TreeNode<K,V> b = this;
            // Relink into lo and hi lists, preserving order
            TreeNode<K,V> loHead = null, loTail = null;
            TreeNode<K,V> hiHead = null, hiTail = null;
            int lc = 0, hc = 0;
            // for 循环迭代所有红黑树结点
            for (TreeNode<K,V> e = b, next; e != null; e = next) {
                next = (TreeNode<K,V>)e.next;
                e.next = null;
                // bit 为扩容前的oldCap 
                if ((e.hash & bit) == 0) {
                    if ((e.prev = loTail) == null)
                        loHead = e;
                    else
                        loTail.next = e;
                    loTail = e;
                    ++lc;
                }
                else {
                    if ((e.prev = hiTail) == null)
                        hiHead = e;
                    else
                        hiTail.next = e;
                    hiTail = e;
                    ++hc;
                }
            }

            if (loHead != null) {
                if (lc <= UNTREEIFY_THRESHOLD)
                    tab[index] = loHead.untreeify(map);
                else {
                    tab[index] = loHead;
                    if (hiHead != null) // (else is already treeified)
                        loHead.treeify(tab);
                }
            }
            if (hiHead != null) {
                if (hc <= UNTREEIFY_THRESHOLD)
                    tab[index + bit] = hiHead.untreeify(map);
                else {
                    tab[index + bit] = hiHead;
                    if (loHead != null)
                        hiHead.treeify(tab);
                }
            }
        }
```