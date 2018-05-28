---

layout:		post

title:		LinkedList源码分析（二）

subtitle:	读读源码

date:		2018-05-28

author:		L

header-img:	img/post/2018-05/28.jpg

tags:

    - Java源码解析

---

### 正文之前

> 在上一篇文章中已经讲述了[链表基于列表的基本用法](http://hxblog.site/2018/05/22/LinkedList%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E4%B8%80/)，所以，这一篇文章要讲述链表基于队列的基本用法

主要内容

> 1. 基本概念
> 2. 基于单向队列的操作
> 3. 基于双向队列的操作

### 正文

#### 1. 基本概念

既然是基于队列的操作，就要介绍一下队列的概念：

单向队列(Queue)是先进先出(First-In-First-Out)的线性结构，就好比我们生活中的排队，先来排队的人先办理事务，队列只支持在队尾添加元素，在队首删除元素

双向队列(Deque，double-ended queue)，也叫做双端队列，是基于队列和栈的结构，可以在队列两边进行添加和删除操作

在链表中使用基于队列的方法时，其实是调用上一篇文章中定义的方法，只不过在不同的位置使用来达到模拟的效果
<br>

#### 2. 基于单向队列的操作

主要内容：

> * 添加节点
>   * offer()
> * 删除节点
>   * poll()
>   * remove()
> * 查找元素
>   * peek()
>   * element()

在这几个基本的操作中，添加和删除是针对节点而言，而查找虽然是针对节点，但是查询的结果是返回节点中的元素的

##### 1. 添加节点

使用add()，在add()中又有linkLast()，结果就是在末尾添加节点：

```
    /**
     * Adds the specified element as the tail (last element) of this list.
     *
     * @param e the element to add
     * @return {@code true} (as specified by {@link Queue#offer})
     * @since 1.5
     */
    public boolean offer(E e) {
        return add(e);
    }
```

##### 2. 删除节点

删除元素有两种方法，区别就是，一个先判断元素是否为null，而另一个不进行判断：

* poll()

先判断元素是否存在，然后调用unlinkFirst()，删除第一个节点：

```
    /**
     * Retrieves and removes the head (first element) of this list.
     *
     * @return the head of this list, or {@code null} if this list is empty
     * @since 1.5
     */
    public E poll() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }
```


* remove()

调用removeFirst()，最终也是调用unlinkFirst()，删除第一个节点：

```
    /**
     * Retrieves and removes the head (first element) of this list.
     *
     * @return the head of this list
     * @throws NoSuchElementException if this list is empty
     * @since 1.5
     */
    public E remove() {
        return removeFirst();
    }
```

##### 3. 修改元素

在基于队列的操作中是没有修改元素的方法的，不管是单向还是双向队列：

##### 4. 查找元素

* peek()

先检查节点是否为空，若不为空就返回第一个节点的元素：

```
    /**
     * Retrieves, but does not remove, the head (first element) of this list.
     *
     * @return the head of this list, or {@code null} if this list is empty
     * @since 1.5
     */
    public E peek() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }
```

* element()

直接返回第一个节点的元素，可能会抛出异常：

```
    /**
     * Retrieves, but does not remove, the head (first element) of this list.
     *
     * @return the head of this list
     * @throws NoSuchElementException if this list is empty
     * @since 1.5
     */
    public E element() {
        return getFirst();
    }
```

<br>

#### 3. 基于双向队列的操作

主要内容：

> * 添加节点
>   * offerFirst()
>   * offerLast()
>   * push()
> * 删除节点
>   * pollFirst()
>   * pollLast()
>   * pop()
>   * removeFirstOccurrence()
>   * removeLastOccurrence()
> * 查找元素
>   * peekFirst()
>   * peekLast()

##### 1. 添加节点

* 添加节点至队首

调用addFirst()，然后是linkFirst()，添加节点至队首：

```
    public boolean offerFirst(E e) {
        addFirst(e);
        return true;
    }
```

* 添加节点至队尾

和上述一样，都是连着再次调用其他方法来达到效果：

```
    public boolean offerLast(E e) {
        addLast(e);
        return true;
    }
```

* 模拟栈操作的添加一个节点至队首，最终还是调用linkFirst()：

```
    public void push(E e) {
        addFirst(e);
    }
```

##### 2. 删除节点

* 删除第一个节点，调用unlinkFirst()：

```
    public E pollFirst() {
        final Node<E> f = first;
        return (f == null) ? null : unlinkFirst(f);
    }
```

* 删除最后一个节点， 调用unlinkLast()：

```
    public E pollLast() {
        final Node<E> l = last;
        return (l == null) ? null : unlinkLast(l);
    }
```

* 模拟栈操作的删除第一个节点，最终还是调用unlinkFirst()：

```
    public E pop() {
        return removeFirst();
    }
```

* 删除某个节点的元素，在它第一次出现的位置

remove()方法是从队首还是进行遍历，如果找到与指定元素相同的节点元素，就返回，所以是在元素第一次出现的位置将其删除：

```
   public boolean removeFirstOccurrence(Object o) {
        return remove(o);
    }
```

* 删除某个节点的元素，在它最后一次出现的位置

其实这个方法的写法和remove()是类似的，只不过是从队尾开始遍历：

```
    public boolean removeLastOccurrence(Object o) {
        if (o == null) {
            for (Node<E> x = last; x != null; x = x.prev) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = last; x != null; x = x.prev) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
```

##### 3. 查找元素

* 得到队首、队尾的节点的元素

```
    public E peekFirst() {
        final Node<E> f = first;
        return (f == null) ? null : f.item;
    }

    public E peekLast() {
        final Node<E> l = last;
        return (l == null) ? null : l.item;
    }
```

这一篇文章就结束了，接下来会是迭代器的分析，作为链表系列文章的最后一篇，然后会有一个关于**ArrayList**和**LinkedList**的对比分析
