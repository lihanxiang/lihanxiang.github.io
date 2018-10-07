---
layout:		post
title:		Java基础知识梳理（六）—— 初识 HashMap
subtitle:	知识点回顾
date:		2018-10-07
author:		L
header-img:	img/post/2018-10/07.jpg
tags:
    - Java基础
    - 集合类
---

### 正文之前

虽然平时也都有用到 HashMap（JDK 1.8），但一直都是处于只会用的状态，一直不知道操作是如何实现的，以及扩容的方式等等，刚好在周末的空闲时间，来学习一下它的源码，一篇肯定说不完，这一篇就先做个介绍吧：

> 1. 基本概念
> 2. 容量
> 3. 基本结构
> 4. 成员变量
> 5. 构造器
> 6. 插入和查找
> 7. 总结

### 正文

#### 1. 基本概念

在 HashMap 一开始的一段注释中，已经对这个类解释的很清楚了：

* 提供所有的 map 操作
* 可以有空的 key-value 对
* 与 Hashtable 大致相同（除了非线程安全和允许空值）
* 元素顺序不保证
* 如果元素散列得当，get 和 put 操作的时间是固定的
* 性能受初始容量和负载系数的影响
* 当前容量 * 负载系数 < 元素数量时，就扩容
* 负载系数默认为 0.75，这是时间消耗和空间消耗的一个折衷点
* 非线程安全，如有需要，用 `Collections.synchronizedMap` 包装

#### 2. 容量

关于容量，源码中的注释说到了一定要为 2 的幂次，这样有利于解决哈希碰撞（实现均匀分布），在文末会有关于为什么的链接：

```
    //默认的初始化容量
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
    //最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30;
    //默认负载系数
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

在链表和红黑树之间转化时，有这么两个参数：

```
    //超过 8 时进化为红黑树
    static final int TREEIFY_THRESHOLD = 8;
    //小于 6 时退化为链表
    static final int UNTREEIFY_THRESHOLD = 6;
```

#### 3. 基本结构

在普通桶中使用的单链表的基本结构，存储哈希值，键值对以及下一个节点的引用：

```
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

        //节点的哈希值为 key 的哈希值异或 value 的哈希值 
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
```

#### 4. 成员变量

HashMap 中有几个成员变量，在下文中会用到，这里就不解释，下面的方法中会有说明：

```
    //table 的大小是要为 2 的幂次，在初次使用时才初始化（延迟加载）
    transient Node<K,V>[] table;

    transient Set<Map.Entry<K,V>> entrySet;

    transient int size;

    transient int modCount;

    int threshold;

    final float loadFactor;
```

#### 5. 构造器

```
    //自定义容量以及负载系数的初始化
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

    //自定义容量
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

    //默认容量 16 以及负载系数 0.75
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }

    //复制一个 Map
    public HashMap(Map<? extends K, ? extends V> m) {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        putMapEntries(m, false);
    }
```

tableSizeFor() 这个方法单独拎出来说一下：

前面说到 table 的值要是 2 的幂次，如果输入的数字不符合要求呢？那就找到比这个数大的**最小的**2次幂，接下来给出源码，并用数字 9 举个例子：

```
    /**
     * Returns a power of two size for the given target capacity.
     */
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

![](https://upload-images.jianshu.io/upload_images/3426615-a45bdce464503061.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 6. 插入和查找

插入操作，代码中有分为单链表和红黑树两种情况：

```

    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //若 table 还未分配空间，就初始化
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        //将节点散列至相应位置
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            //插入至红黑树
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    //插入至链表
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        //转化为红黑树
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
            //已经存在对应的值，就更新
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        //扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```

查找操作：

```
    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }

    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            //总是先检查根节点
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                //在红黑树中查找
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    //在链表中查找
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```

#### 7. 总结

其实这一篇只能算是科普文，因为最重点的扩容、红黑树的转化等操作都还没说，接下来的文章将会全面说明这些重点内容

参考资料：

容量为什么是 2 的幂次：[https://blog.csdn.net/qq_36523667/article/details/79657400](https://blog.csdn.net/qq_36523667/article/details/79657400)

关于负载系数：[https://www.jianshu.com/p/64f6de3ffcc1](https://www.jianshu.com/p/64f6de3ffcc1)
