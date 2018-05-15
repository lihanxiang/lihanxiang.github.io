---
layout:		post
title:		ArrayList源码分析（三）
subtitle:	读读源码
date:		2018-05-15
author:		L
header-img:	img/post/2018-05/15.jpg
tags:
    - Java源码解析
---

### 正文之前

> 在先前的[第一篇文章](http://hxblog.site/2018/05/04/ArrayList%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E4%B8%80/)以及[第二篇文章](http://hxblog.site/2018/05/09/ArrayList%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E4%BA%8C/)中已经讲述了ArrayList的常用方法，但是没有使用迭代器，这一篇文章就来讲讲使用迭代器操作ArrayList的基本方法
> 
> 主要内容: 
> 1. 迭代器的构造
> 2. 使用迭代器进行增删改查

### 正文

#### 1. 构造迭代器所需的类

在源码中，有两个私有类：**Itr**以及**ListItr**，第一个是单向操作，第二个能够支持双向操作

需要先看一下这两个类的源码，它们对于迭代器的构造必不可少：

##### Itr类

1. 实现接口 

```
//实现了Iterator接口
private class Itr implements Iterator<E> {}
```
<br>

2. 变量

在这里，关于迭代器的操作，我觉得用指针来形容十分恰当，指针移动到哪一个位置，就表示迭代器操作的位置在哪，在源码中有这么两个变量：

```
        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
```

那我们就把第一个变量称为指针1，指向下一个要返回的元素，把第二个变量称为指针2，指向刚返回的元素，这里的指向，其实就是一个位置的关系，下面慢慢会讲到

3. 判断后一个元素

```
        //判断是否还有下一个元素
        public boolean hasNext() {
            return cursor != size;
        }
```
<br>

4. 向后移动

```
        public E next() {
            //这一语句涉及并发，不参与讨论
            checkForComodification();
            //i = 指针1的位置
            int i = cursor;
            //超出数组长度
            if (i >= size)
                throw new NoSuchElementException();
            //复制
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            //指针后移一位，也就是说，指针1的位置在下一个元素的位置
            cursor = i + 1;
            //设置lastRet = i，然后返回这个位置的值，指针2的位置就在这了
            return (E) elementData[lastRet = i];
        }
```
<br>

5. 删除

如果删除了一个，指针2就会重置，这时候如果再删除指针就会报错

```
        public void remove() {
            //如果下标不符合
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                //指针1移动到指针2的位置
                cursor = lastRet;
                //重置指针2的位置
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
```

做个测试

```
        ArrayList<Integer> list1 = new ArrayList<>();

        for (int i = 0; i < 10; i++) {
            list1.add(i);
        }

        Iterator iterator1 = list1.iterator();

        iterator1.next();
        iterator1.remove();
        System.out.println("进行删除之后的数据：");
        for (int i:
                list1) {
            System.out.print(i + " ");
        }
        System.out.println();
        System.out.println("再次进行删除操作：");
        try {
            iterator1.remove();
        } catch (IllegalStateException e){
            System.out.println("操作失败");
        }
```

结果是：

![](https://upload-images.jianshu.io/upload_images/3426615-6f6361dae4f01a6d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



<br>

##### ListItr

1. 实现接口

```
//这个类继承自上面的**Itr**类，实现**ListIterator**接口，这个接口是继承自**Iterator**接口的
private class ListItr extends Itr implements ListIterator<E> {}

public interface ListIterator<E> extends Iterator<E> {}
```
<br>

2. 构造器

```
        ListItr(int index) {
            super();
            cursor = index;
        }
```
<br>

3. 判断前一个元素

```
        //如果不是第一个元素，就有前一个元素
	    public boolean hasPrevious() {
            return cursor != 0;
        }
```
<br>

4. 判断前后位置

```
    public int nextIndex() {
            return cursor;
        }

        public int previousIndex() {
            return cursor - 1;
        }
```
<br>

5. 获得前一个元素

```
        public E previous() {
            checkForComodification();
            int i = cursor - 1;
            if (i < 0)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            //指针1向前移一位
            cursor = i;
            //指针2指向这个元素
            return (E) elementData[lastRet = i];
        }
```
<br>

6. 修改元素

```
        public void set(E e) {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();
    
            try {
                //将指针2指向的位置的值修改
                ArrayList.this.set(lastRet, e);
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
```
<br>

7. 添加元素

```
        public void add(E e) {
            checkForComodification();

            try {
                int i = cursor;
                //在指针1指向的位置添加元素
                ArrayList.this.add(i, e);
                //指针1向后移一位
                cursor = i + 1;
                //重置指针2
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
```
<br>

#### 2. 迭代器的构造

**Itr**和**ListItr**这两个类都是私有的，他们是为了迭代器的构造而准备的

##### 1. Itr 类的使用

调用iterator()方法就会得到：

```
    public Iterator<E> iterator() {
        return new Itr();
    }
```
<br>

##### 2. ListItr 类的使用

分为两种，指定迭代器初始位置或使用默认位置

1. 指定初始位置

```
    public ListIterator<E> listIterator(int index) {
        if (index < 0 || index > size)
            throw new IndexOutOfBoundsException("Index: "+index);
        //指针1指向输入的index，设定为初始位置
        return new ListItr(index);
    }
```
<br>

2. 使用默认位置

```
    public ListIterator<E> listIterator() {
        //默认位置为首部
        return new ListItr(0);
    }
```
<br>

#### 3. 使用迭代器进行增删改查

每一次测试都初始化一次，保证每次测试的列表元素都是相同的

1. 初始化

```
        ArrayList<Integer> list1 = new ArrayList<>();
        ArrayList<Integer> list2 = new ArrayList<>();

        for (int i = 0; i < 10; i++) {
            list1.add(i);
            list2.add(i);
        }

        ListIterator<Integer> iterator1 = list1.listIterator();
        ListIterator<Integer> iterator2 = list2.listIterator(5);
```
<br>

2. 增加数据

 ```
         //增加数据
        iterator1.add(100);
        for (int i :
                list1) {
            System.out.print(i + " ");
        }
        System.out.println();
        iterator2.add(200);
        for (int i :
                list2) {
            System.out.print(i + " ");
        }
```

![](https://upload-images.jianshu.io/upload_images/3426615-76b6488edb5b0f7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<br>

3. 删除数据

```
        //删除数据
        iterator1.next();
        iterator1.remove();
        for (int i :
                list1) {
            System.out.print(i + " ");
        }
        System.out.println();
        iterator2.next();
        iterator2.remove();
        for (int i :
                list2) {
            System.out.print(i + " ");
        }
```

![](https://upload-images.jianshu.io/upload_images/3426615-8ed2720c12e1cf2f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<br>

4. 修改数据

```
        //修改数据
        iterator1.next();
        iterator1.set(100);
        for (int i :
                list1) {
            System.out.print(i + " ");
        }
        System.out.println();
        iterator2.next();
        iterator2.set(200);
        for (int i :
                list2) {
            System.out.print(i + " ");
        }
```

![](https://upload-images.jianshu.io/upload_images/3426615-5973d494503f9669.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4. 查看数据

使用迭代器遍历数组并输出

```
        //查看数据
        while (iterator1.hasNext()){
            System.out.print(iterator1.next() + " ");
        }
        System.out.println();
        while (iterator2.hasNext()){
            System.out.print(iterator2.next() + " ");
        }
```

![](https://upload-images.jianshu.io/upload_images/3426615-8efaae0d3d9302c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

到此，ArrayList的基本用法的源码解读就告一段落了，接下来会是LinkedList的源码解析
