---
title: HashMap原理
date: 2019-03-04 12:52:46
categories: 源码解读
catalogue: java
tags: [HashMap]
---
##### 目的：
单纯分析和学习hashmap的实现，不多说与Hashtable、ConcurrentHashMap等的区别。
基于 jdk1.8

在面试中有些水平的公司比较喜欢问HashMap原理，其中涉及的点比较多，而且大多能形成连环炮形式的问题。

       一般连环炮，一环不知道后面试官也就不问了，但是低层连环没连上，恭喜扣分是大大的，连到比较深的时候，说不知道还好点，比如：

关于集合的
1.1Hashmap是不是有序的？   不是继续

1.2有没有有顺序的Map？     TreeMap  LinkedHashMap

1.3它们是怎么来保证顺序的？   一般都要说到其源码，要不说不清为么有序

1.4答两个有序或以上的 继续  你觉得它们有序的区别，那个比较好，在什么场景用哪个好？

1.4答一个也可以问上面的场景  继续

1.5你觉得有没有更好或者更高效的实现方式？有

1.6 答有  这个时候说起来可能就要跑到底层数据结构上去了

数据结构继续衍生 到 算法等等。。。

就这一个遇到大佬问你，能把很多人连到怀疑人生

2.关于hash的

1.1  hashmap基本的节点结构？  Node  键值对

1.2  键是什么样的，我用字符串a那键就是a嘛？   不是会进行hash

1.3  如何hash的  这样hash有什么好处？   源码hashmap的hash算法

1.4  Hash在java中主要作用是什么？

1.5  Hashcode  equal相关   需要同时重写？原因？

1.6  equal引出的对象地址、string带有字符串缓冲区、字符串常量池

等等。。。

3.关于线程安全问题、到concurrent包等

前面说这些就是想说，hashmap中用到的东西很多，深入学习和理解对每个想晋升的程序员来说基本是必须，同时由它引出的对比，也是无限多，有很大的必要学习。

#### HashMap类加载

1.只有一些静态属性会进行赋值，具体每个值什么用，暂时不管

```
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

static final int MAXIMUM_CAPACITY = 1 << 30;

static final float DEFAULT_LOAD_FACTOR = 0.75f;

static final int TREEIFY_THRESHOLD = 8;

static final int UNTREEIFY_THRESHOLD = 6;

static final int MIN_TREEIFY_CAPACITY = 64;
```

#### 开始使用，第一步我们肯定是初始化方法，先从默认的构造方法开始学习

```
public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
1.AbstractMap父类，构造方法也没干事不谈
2.只是赋值loadFactor 0.75f 没干别的事
3.static final float DEFAULT_LOAD_FACTOR = 0.75f;
4.loadFactor属性 作用先放着后面用到再看
5.没干别的事了
```
#### 一般我们的使用第二步就是put了
先看常用的put键值对，这个学完了，那么其他的put方法就没什么问题了，比如putAll、putIfAbsent、putMapEntries

同时put弄明白了 取值就是一个反向就简单了

```
public V put(K key, V value) {
  return putVal(hash(key), key, value, false, true);
}
```

1.先对key进行hash计算，学一下

```
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
1.1 看出key是可以空的  hash为0
1.2 (h = key.hashCode()) ^ (h >>> 16)   第一步取key的hashcode值  关于更底层的hashcode是什么 有兴趣再看
    h ^ (h >>> 16)  第二步 高位参与运算
这个hash值的重要性就不说了，这里这么干是出于性能考虑，底层的移位和异或运算肯定比加减乘除取模等效率好 
hashcode是32位的，无符号右移16位，那生成的就是16位0加原高位的16位值， 就是对半了，异或计算也就变成了高16位和低16位进行异或，
原高16位不变。这么干主要用于当hashmap 数组比较小的时候所有bit都参与运算了，防止hash冲突太大，
所谓hash冲突是指不同的key计算出的hash是一样的，比如a和97，这个肯定是存在的没毛病
```
2.putVal 

```
  /**
     * Implements Map.put and related methods
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value   相同key是不是覆盖值
     * @param evict if false, the table is in creation mode.    在hashmap中没用
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
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
    2.1 执行顺序
        第一句  Node<K,V>[] tab; Node<K,V> p; int n, i; 申明变量
            Node是啥,学习一下：
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
            是内部的一个静态类，看看就明白了，明显是一个带有3个值，hash、key、value和另一个Node对象引用的HashMap子元素结构，即我们装的每个键值对就用一个Node对象存放
            
        第二句 if ((tab = table) == null || (n = tab.length) == 0) 这句
            tab = table赋值，table现在是null的，so n = tab.length不运行了 运行这个if的代码块
        第三句 n = (tab = resize()).length;   从下面的执行知道 n=16
            调用resize()，返回Node数组，这个resize是一个非常重要的方法，我们就依现在的对象状态去看这个方法，不带入其他状态，认真研究学习下
                final Node<K,V>[] resize() {
                    Node<K,V>[] oldTab = table;
                    int oldCap = (oldTab == null) ? 0 : oldTab.length;
                    int oldThr = threshold;
                    int newCap, newThr = 0;
                    if (oldCap > 0) {
                        if (oldCap >= MAXIMUM_CAPACITY) {
                            threshold = Integer.MAX_VALUE;
                            return oldTab;
                        }
                        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                                 oldCap >= DEFAULT_INITIAL_CAPACITY)
                            newThr = oldThr << 1; // double threshold
                    }
                    else if (oldThr > 0) // initial capacity was placed in threshold
                        newCap = oldThr;
                    else {               // zero initial threshold signifies using defaults
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
                    if (oldTab != null) {
                        for (int j = 0; j < oldCap; ++j) {
                            Node<K,V> e;
                            if ((e = oldTab[j]) != null) {
                                oldTab[j] = null;
                                if (e.next == null)
                                    newTab[e.hash & (newCap - 1)] = e;
                                else if (e instanceof TreeNode)
                                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                                else { // preserve order
                                    Node<K,V> loHead = null, loTail = null;
                                    Node<K,V> hiHead = null, hiTail = null;
                                    Node<K,V> next;
                                    do {
                                        next = e.next;
                                        if ((e.hash & oldCap) == 0) {
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
                                    } while ((e = next) != null);
                                    if (loTail != null) {
                                        loTail.next = null;
                                        newTab[j] = loHead;
                                    }
                                    if (hiTail != null) {
                                        hiTail.next = null;
                                        newTab[j + oldCap] = hiHead;
                                    }
                                }
                            }
                        }
                    }
                    return newTab;
                }
            resize  1.Node<K,V>[] oldTab = table; 在上面知道table是null的，so  oldTab也是null
                    2.int oldCap = (oldTab == null) ? 0 : oldTab.length;   oldCap=0
                    3.int oldThr = threshold;   threshold我们没赋值过，int初始0 ， oldThr=threshold=0
                    4.int newCap, newThr = 0;  不谈
                    5.if (oldCap > 0) {    oldCap=0  if不运行
                    6.else if (oldThr > 0)  oldThr=0  if也不运行
                    7.else {
                            newCap = DEFAULT_INITIAL_CAPACITY;       DEFAULT_INITIAL_CAPACITY静态成员变量，初始 static final int DEFAULT_INITIAL_CAPACITY = 1 << 4     so newCap=16
                            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);   static final float DEFAULT_LOAD_FACTOR = 0.75f;   0.75*16=12   newThr=12
                        }
                    8.  if (newThr == 0) {     newThr=12 if不运行
                    9.  threshold = newThr;    threshold = newThr=12
                    10. Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap]   申明一个16个大小的Node数组
                    11. table = newTab;   看出来了吧，table是成员变量，也就表明，HashMap初始数据结构是一个16的Node数组
                    12.  if (oldTab != null) {   oldTab是1中赋值的null，if不运行
                    13.  return newTab;  返回16大小的node数组
                总结，这一波调用是初次调用其实没干别的事，就是定义了基本的数据结构是16个Node数组，但是这个方法不简单，因为一些if没走
                
        第四句     if ((p = tab[i = (n - 1) & hash]) == null)
            n=16   15&hash 结果肯定是0-15，这里就看出，这是在计算一个key应该在整个数据结构16的数组中的索引了，并赋值给i变量，后面不管整体结构n变多大，这种计算key所在的索引是非常棒的设计。
            现在的状态是初始的 肯定是null的吧  if运行
            
        第五句 tab[i] = newNode(hash, key, value, null); new一个节点Node，放在数组里，i是第四句计算的索引
        第六句 else {  不运行
        第七句  ++modCount;   transient int modCount; 根据注释可以看出，这个是记录数据结构变动次数的，put值肯定是变了的
        第八句  if (++size > threshold)  size=1  threshold在调用resize时赋值12   if不运行
        第九句 afterNodeInsertion(evict);  没干事
        第十句   return null; 不谈

```
3.putVal 再回头详走,第一遍干了很多初始化的事有些东西还没研究到

```
  final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
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
    
        第一句  Node<K,V>[] tab; Node<K,V> p; int n, i; 申明变量不谈
        第二句 if ((tab = table) == null || (n = tab.length) == 0) 这句
            tab = table赋值，table现在是16数组 n=16  if不运行
        第三句 if ((p = tab[i = (n - 1) & hash]) == null) 
            再看就知道了判断当前存的key计算出的索引位置是不是已经存过值了
            没存过就新Node存  和上面一遍一样   我们当已经有值了
            有值其实就意味着发生hash冲突了  比如key分别是a和97 hashCode都是97 冲突
            因此这次我们主要看下一个else里面HashMap是怎么处理冲突的
        第四句     else中内容  即冲突处理
            p是冲突时数组该索引位置的元素
                1. p.hash == hash &&((k = p.key) == key || (key != null && key.equals(k)))
                    判断新元素hash和key是不是都和p相同，相同表示存了一样的key
                    直接赋值给e
                2. p instanceof TreeNode（红黑树，具体的红黑树算法这里就不详细写了，有兴趣可以去学习）
                    怎么猛然来个红黑树，再3里说
                    判断原来元素是不是 TreeNode 类型
                    TreeNode一样是静态内部类，再看看就是红黑树的节点，因此这个地方用到了红黑树
                    putTreeVal 向红黑树中添加元素
                    内部实现，存在相同key就返回赋值给e  不存在就添加并返回null 源码就是红黑树算法
                3.key不同也不是红黑树
                     if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                    }
                    先不看再里面的那个if，这个一看就知道了吧，明显的链表啊，而且数据里的这个元素是链表头
                    整个循环，明显是在从头开始遍历链表，找到相同key或链表找完了新元素挂链表最后
                    
                    但在其中还有这么个if
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                        这是在链表找完了，且新元素已经挂在链表最后了有的一个判断
                        判断循环次数，其实就是链表长度，长度超过TREEIFY_THRESHOLD 默认8则运行treeifyBin(tab, hash);
                    就是这个方法把链表变成红黑树了，具体方法源码不谈了，学红黑树就可以了
            
            最后判断e是不是空，上面的冲突方案看出e不是空就是表示有相同的key进行value覆盖就可以，e空就是无相同key且完成了数据挂载
            
        
        总结这次再走一遍putVal就是为了学习HashMap的冲突处理方案，也看出内存结构是数组、链表、红黑树组成的，红黑树是java8新引进，是基于性能的考虑，在冲突大时，红黑树算法会比链表综合表现更好
```

4.resize 再详走 putVal最后一段size>threshold  threshold初始12 ++size元素数量肯定会有超12个的时候，
这里也就看出了threshold代表HashMap的容量，到上限就要扩容了，默认现在16数组，12元素上限

```
  1.Node<K,V>[] oldTab = table;  16大小
        2.int oldCap = (oldTab == null) ? 0 : oldTab.length;   oldCap=16
        3.int oldThr = threshold;     12
        4.int newCap, newThr = 0;  不谈
        5.if (oldCap > 0) {         oldCap=16运行  oldCap是整体结构数组大小
                if (oldCap >= MAXIMUM_CAPACITY) {    判断数组大小是不是已经到上限1<<30
                    threshold = Integer.MAX_VALUE;  到达上线 threshold 赋值最大值 然后返回 表示之后就不再干别的事了，随便存，随便hash冲突去，就这么大，无限增加红黑树节点了
                    return oldTab;
                }
                else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                         oldCap >= DEFAULT_INITIAL_CAPACITY)   赋值newCap为2倍数组大小，判断如果扩充2倍有没到上限，且不扩充时容量是否大于默认的16
                    newThr = oldThr << 1; // double threshold   满足则赋值  容量改为24
            }
            这段看出到threshold容量了就进行2倍扩容
        6.if (newThr == 0) {    如果运行该if 0 表示5步中扩容2倍到上限或原数组大小小于16
            float ft = (float)newCap * loadFactor;      newCap现在是2倍原大小的*0.75   2倍数组大小时的容量
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);     判断2倍数组大小和2倍后的容量是不是都小于最高值，是则赋值新容量，不是就用整形最大值
            }
        7.  threshold = newThr;  把5 6两步算出的新容量赋值给HashMap  也说明要扩容了
        8.   Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
            和后面的循环主要就是把原数组中的元素，一个一个添加到新数组中，转移的一个过程
        
    总结，这一波调用是了解HashMap的扩容方式，看下来就是2倍扩容直到上限
```
5.总结，到这put就比较详细了，也知道了基本结构是数组、链表、红黑树，链表到8个时转换成红黑树
同时每次进行2倍扩容和数据转移，扩容是用新结构的那显然减少扩容次数会有更好的性能
那就要求每次声明HashMap时最好是指定大小的

#### 一些其他我们需要知道的
1.指定大小的初始化

```
  public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
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
    第一个常用，第二个建议是不用，不去动0.75的这个容量比例，当然不绝对
    这里tableSizeFor是一个很神奇的算法，我非常佩服的一个算法
        static final int tableSizeFor(int cap) {
            int n = cap - 1;
            n |= n >>> 1;
            n |= n >>> 2;
            n |= n >>> 4;
            n |= n >>> 8;
            n |= n >>> 16;
            return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
        }
        这个方法是在找大于等于cap且最小2的幂
        比如cap=1   结果 2 0次方 1
        cap=2  2
        cap=3 4
        cap=9  16
        分析下等于9
        cap - 1  第一步结果8
        00000000000000000000000000001000    8
        00000000000000000000000000000100    右移1位 
        
        00000000000000000000000000001100    或运算 结果
        00000000000000000000000000000011    右移2位
        00000000000000000000000000001111    或运算 结果
                                            
        00000000000000000000000000001111    右移 4 8 16没用全是0结果还是这个15
        最终 +1   16
        
        分析下等于大点 12345678
        00000000101111000110000101001110  12345678
        00000000101111000110000101001101  -1结果   12345677
        00000000010111100011000010100110  右移1位 
        
        00000000111111100111000111101111  或运算 结果
        00000000001111111001110001111011  右移2位
        
        00000000111111111111110111111111  差不多了在移0就没了都是1了，+1不是肯定是2的倍数了
        
        再说开始-1原因这是为了防止，cap已经是2的幂。
        如果cap已经是2的幂， 又没有执行这个减1操作，则执行完后面的几条无符号右移操作之后，返回的capacity将是这个cap的2倍。如果不懂，要看完后面的几个无符号右移之后再回来看看
```
2.HashMap数组结构为什么用2的倍数
高速的索引计算，使用HashMap肯定是冲突越少越好，就要求分部均匀，最好的用取模 h % length，但是近一步如果用2的幂h & (length - 1) == h % length 是等价的，效率缺差却别非常大
综合衡量用空间换了时间，且是值得的

3.线程安全问题
线程不安全，就put来看全程没考虑线程问题，肯定不安全，现在随便并发一下resize会混乱吧，put链表，红黑树挂载基本都会出问题