---
layout:		post
title:		LinkedList源码分析（三）
subtitle:	读读源码
date:		2018-06-08
author:		L
header-img:	img/post/2018-06/08.jpg
tags:
    - Java源码解析
---

### 正文之前

>在前面两篇文章中已经讲述了 LinkedList [基于列表](http://hxblog.site/2018/05/22/LinkedList%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E4%B8%80/)和[基于队列](http://hxblog.site/2018/05/28/LinkedList%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E4%BA%8C/)的操作，这一篇文章作为 LinkedList 系列文章的最后一篇，将会讲述迭代器的用法：
>
> 1. ListItr
> 2. DescendingIterator

### 正文

#### 1. ListItr

> 1. 成员变量
> 2. 构造
> 3. 增
> 4. 删
> 5. 改
> 6. 查

在源码中，**ListItr** 对象是私有的，对象的发布是通过 **listIterator()** 方法进行的，关于对象的发布这里不讲述，先看看 **listIterator()** 方法是怎样的：

```
    //index是从1开始的
    public ListIterator<E> listIterator(int index) {
        //先检查给出的索引是否有效
        checkPositionIndex(index);
        //返回 ListItr 对象
        return new ListItr(index);
    }
```

然后看看 **ListItr** 类：

```
    private class ListItr implements ListIterator<E> {}
```

##### 1. 成员变量

```
        //当前节点
        private Node<E> lastReturned;
        //下一个节点
        private Node<E> next;
        //下一个节点的位置
        private int nextIndex;
        private int expectedModCount = modCount;
```

##### 2. 构造

```
        ListItr(int index) {
            // assert isPositionIndex(index);
            //需要判断索引是否有效，然后确定下一个节点
            next = (index == size) ? null : node(index);
            //下一个节点的位置
            nextIndex = index;
        }
```

传入的参数 index 是从1开始的，一试便知：

![](https://upload-images.jianshu.io/upload_images/3426615-897d6574fbe14444.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 3. 增

方法中使用到了之前写的基于列表的操作：

```
        public void add(E e) {
            checkForComodification();
            lastReturned = null;
            //最后一个节点
            if (next == null)
                //末尾添加
                linkLast(e);
            else
                //在指定位置添加
                linkBefore(e, next);
            //把下一个节点的位置往后移
            nextIndex++;
            expectedModCount++;
        }

        
```

##### 4. 删

方法中使用到了之前写的基于列表的操作：

```
        public void remove() {
            checkForComodification();
            //如果当前节点为空
            if (lastReturned == null)
                throw new IllegalStateException();

            //定义一个节点，表示当前节点的下一个节点
            Node<E> lastNext = lastReturned.next;
            //删除节点
            unlink(lastReturned);
            if (next == lastReturned)
                next = lastNext;
            else
                nextIndex--;
            //将当前节点设为null
            lastReturned = null;
            expectedModCount++;
        }
```

##### 5. 改

将当前节点中的元素改为传入的参数：

```
        public void set(E e) {
            //如果节点为空
            if (lastReturned == null)
                throw new IllegalStateException();
            checkForComodification();
            lastReturned.item = e;
        }
```

##### 6. 查

迭代器支持双向操作，所以能够向前或向后移动：

向后移动：

```
        //判断是否有下一个节点
        public boolean hasNext() {
            return nextIndex < size;
        }

        public E next() {
            checkForComodification();
            if (!hasNext())
                throw new NoSuchElementException();
            //将下一个节点表示为当前节点
            lastReturned = next;
            next = next.next;
            //位置向后移动
            nextIndex++;
            //返回当前节点的元素
            return lastReturned.item;
        }

        //返回下一个节点的位置
        public int nextIndex() {
            return nextIndex;
        }
```

向前移动，操作是类似的：

```
        public boolean hasPrevious() {
            return nextIndex > 0;
        }

        public E previous() {
            checkForComodification();
            if (!hasPrevious())
                throw new NoSuchElementException();
            //向前移，并修改位置
            lastReturned = next = (next == null) ? last : next.prev;
            nextIndex--;
            //返回元素
            return lastReturned.item;
        }

        //同样是返回位置
        public int previousIndex() {
            return nextIndex - 1;
        }
```

#### 2. DescendingIterator

反向迭代器对象的发布方式也和上面的一样，通过 **descendingIterator()** 来发布对象：

```
    public Iterator<E> descendingIterator() {
        return new DescendingIterator();
    }

    private class DescendingIterator implements Iterator<E> {}
```

反向迭代器实现的方法就3个，我们先看一眼成员变量：

```
    private final ListItr itr = new ListItr(size());
```

其实反向迭代器是基于 **ListItr** 来实现的，直接在迭代器的初始位置设置在链表的末尾

接下来来看看3个方法：

```
        public boolean hasNext() {
            return itr.hasPrevious();
        }
        public E next() {
            return itr.previous();
        }
        public void remove() {
            itr.remove();
        }
```

因为这是反向的，所以向前方法的名字还是 **next()**，但是，是使用了 **previous** 来进行的

那么，有关 LinkedList 的基本用法的源码就已经解读完毕了，接下来会是其他容器类的源码解读
