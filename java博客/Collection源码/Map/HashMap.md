https://blog.csdn.net/weixin_45690436/article/details/109768120

https://blog.csdn.net/weixin_40413961/article/details/106740423

属性：

```java
初始容量 16 （这里是桶的数量）
 static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

最大容量 2^30 次方 （桶的最大容量）
static final int MAXIMUM_CAPACITY = 1 << 30;

默认的装载因子
static final float DEFAULT_LOAD_FACTOR = 0.75f;

当一个元素被添加到有至少 TREEIFY_THRESHOLD 个节点的桶中，桶中链表将会被转换为树形结构
static final int TREEIFY_THRESHOLD = 8;

将树形结构转换为链表的最小节点
static final int UNTREEIFY_THRESHOLD = 6;

桶可能被转换成数结构的最小容量
static final int MIN_TREEIFY_CAPACITY = 64;

HashMap中存放KV的数量（为链表和树中的KV的总和）。
transient int size;

当HashMap的size大于threshold时会执行resize操作 threshold=capacity*loadFactor
int threshold;

final float loadFactor;
```



![在这里插入图片描述](20160407175503308.png)



#### 约定

> 约定前面的数组结构的每一个格格称为**桶** 
> 约定桶后面存放的每一个数据称为**bin** 
> bin这个术语来自于JDK 1.8的HashMap注释。



​		当好多bin被映射到同一个桶时，如果这个桶中bin的数量小于 TREEIFY_THRESHOLD 当然不会转化成树形结构存储；如果这个桶中bin的数量大于了 `TREEIFY_THRESHOLD` ，但是capacity小于`MIN_TREEIFY_CAPACITY` 则依然使用链表结构进行存储，此时会对HashMap进行扩容；如果capacity大于了`MIN_TREEIFY_CAPACITY` ，则会进行树化。

****







#### putVal 源码：

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
       // 空表则初始化
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
       //非空表且没有碰撞，则直接插入元素到散列表中
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
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



#### resize() :

https://blog.csdn.net/weixin_41565013/article/details/93190786

https://blog.csdn.net/qq_40574571/article/details/97612100

```java
final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;      / / 数组长度
        int oldThr = threshold;
        int newCap, newThr = 0;
       // 原数组长度大于0
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                // 阀值扩大为2倍
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            // 初始化阀值和容量
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
    // 原数组不为空，把原数组复制到扩容后的数组
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                // 遍历数组，把之前的桶移到新的桶中
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    // 很单纯的数组元素（即链表不存在后续节点），直接往新数组后面添加就行
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        // 树
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { 
   			   /*桶中为链表，对链表进行拆分*/
			/*拆分成两个链表*/
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        // 链表的下一元素
                        Node<K,V> next;
                        do {
                            next = e.next;
                           // 选择出扩容后在同一个桶中的节点。
                            if ((e.hash & oldCap) == 0) {
                                // 尾节点为空，说明 lo 链表为空
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null); // 循环遍历链表元素，构建两个链表
                        // 解释了之前为什么需要把尾节点为空的赋值为e
                        if (loTail != null) {
                            // 把上次用作标记尾节点的节点去除。
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            // 新节点的位置是现在的位置加之前 table 长度的。因为是2次幂的扩展。
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```

###### 扩容位置变化分析：

https://blog.csdn.net/qq_40574571/article/details/97612100

​		jdk1.8 使用的是2次幂的扩展(指长度扩为原来2倍)，所以，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置。看下图可以明白这句话的意思，n为table的长度，图（a）表示扩容前的key1和key2两种key确定索引位置的示例，图（b）表示扩容后key1和key2两种key确定索引位置的示例，其中hash1是key1对应的哈希与高位运算结果。扩容一倍，左移一位，故新的 n-1 位是之前  n-1 左移动一位。

![img](20190728110949685.png)

​		元素在重新计算hash之后，因为n变为2倍，那么n-1的mask范围在高位多1bit(红色)，因此新的index就会发生这样的变化：

![在这里插入图片描述](20190728111006889.png)

​		因此，我们在扩充HashMap的时候，不需要像JDK1.7的实现那样重新计算hash，只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“原索引+oldCap”，可以看看下图为16扩充为32的resize示意图：

![img](2019072811103759.png)



###### e.hash & oldCap == 0 算法：

https://blog.csdn.net/u010425839/article/details/106620440/

![img](20200608155000351.png)

简单的说，存的是 e.hash & (oldCap - 1),这时候是位置，而 e.hash & oldCap 时，因为 oldCap 的长度是2的n次幂整数，所以计算结果肯定是0.这里关注 e.hash 的值。