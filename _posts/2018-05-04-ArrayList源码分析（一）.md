---
layout:		post
title:		ArrayList源码分析（一）
subtitle:	读读源码
date:		2018-05-04
author:		L
header-img:	img/post/2018-05/4.jpg
tags:
    - Java源码解析

---

### 正文之前

这篇博客来自[我的简书](https://www.jianshu.com/p/1766dc3b25f5)

> 最近在复习Java基础，感觉又学到了好多，集合作为Java的一大重点，之前是看着《Java编程思想》学的，这次刚好配合着源码重新学一遍，首先学的是ArrayList，顺便做点总结（迭代器无关）：
> 1. ArrayList的基本概念
> 2. ArrayList的源码剖析（常量，构造器，常用方法）
>
> 关于ArrayList的源码，至少要写三四篇才够吧，这源码可是有足足1500行左右（一半是注释），慢慢积累吧

### 正文

#### ArrayList基本概念

刚开始学的时候管它叫动态数组，因为它的大小是根据数据量而自动变化的，它的结构是基于动态数组(Dynamic array)

##### 注释

我在IDEA中之间右键点击ArrayList查看了源码，先将注释作为HTML放在浏览器中看看：

![](https://upload-images.jianshu.io/upload_images/3426615-16421ad35dfcd914.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

把它称作**Resizable-array**就可以看出它的性质了

接下来说说这一整页的大概内容：

1. 它实现了**List**接口，实现了所有**List**的方法，它可以容纳所有元素类型（包括**null**），ArrayList除了不是多线程同步之外，其余的内容都与**Vector**大致相同

2. size, isEmpty, get, set, iterator 和 listIterator 操作用时是常数级别的，add 操作耗时是和插入位置有关，除此之外，其他操作的时间是线性增长的，下面会做个实验验证一下

3. Arraylist有着容量的概念，容量随着数据量的增长而增长，也可以在直接定义这个容器的大小，使用**ensureCapacity**操作，能够一次增长所需要的容量，提高效率，避免一个个添加从而不断增长容量

4. **划重点**，上面说过这个容器不是同步的，如果在多线程中使用，有一个线程要改变容器的结构时，就需要在容器外部进行同步，官方推荐的做法是用同步的容器**将它包装起来**：

```
List list = Collections.synchronizedList(new ArrayList(...));
```

5. 注释中接下来关于迭代器的我们就先跳过，之后会有专门分析迭代器的源码

###### Demo: 

测试一下第2点中的运行时间：

* 先算一下**add**操作的运行时间：

```
import java.util.ArrayList;
import java.util.List;

public class Test {
    public static void main(String[] args) {

        List<String> list = new ArrayList<>();
        long startTime = 0, endTime = 0;

        //插入操作
        startTime = System.currentTimeMillis();
        //需要大一点的数据量才能算出时间
        for (int i = 0; i < 100000; i++) {
            list.add(String.valueOf(i));
        }
        endTime = System.currentTimeMillis();
        System.out.println(endTime - startTime);

        //
        startTime = System.currentTimeMillis();
        for (int i = 100000; i < 200000; i++) {
            list.add(String.valueOf(i));
        }
        endTime = System.currentTimeMillis();
        System.out.println(endTime - startTime);
        list.clear();
    }
}
```

![](https://upload-images.jianshu.io/upload_images/3426615-f7db36bb17079292.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从十万的位置开始插入，多用了一毫秒，说明运行时间和插入位置是有关的

* get操作

```
import java.util.ArrayList;
import java.util.List;

public class Test {
    public static void main(String[] args) {

        List<String> list = new ArrayList<>();
        long startTime = 0, endTime = 0;
        String index;

        //插入操作
        startTime = System.currentTimeMillis();
        for (int i = 0; i < 2000000; i++) {
            list.add(String.valueOf(i));
        }
        endTime = System.currentTimeMillis();
        System.out.println("插入两百万数据时间: ");
        System.out.println(endTime - startTime);

        startTime = System.currentTimeMillis();
        for (int i = 0; i < 1000000; i++) {
            index = list.get(i);
        }
        endTime = System.currentTimeMillis();
        System.out.println("得到前一百万数据时间: ");
        System.out.println(endTime - startTime);


        startTime = System.currentTimeMillis();
        for (int i = 1000000; i < 2000000; i++) {
            index = list.get(i);
        }
        endTime = System.currentTimeMillis();
        System.out.println("得到前一百万数据时间: ");
        System.out.println(endTime - startTime);
        list.clear();
    }
}
```

![](https://upload-images.jianshu.io/upload_images/3426615-5ad55e66f035254d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

get操作时间在相同数据量的前提下是一样的

其他的都是用差不多的方法来验证，就不做了

#### 源码剖析

1. 类

```
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

说明这个类是实现了几个接口：
* List
* RandomAccess
* Cloneable
* java.io.Serializable

第一个不说，说说后面三个，这三个接口都是空的，只是用来说明：

支持快速随机访问

![](https://upload-images.jianshu.io/upload_images/3426615-d7ea7ec302b9d44f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)<br>

能够使用Object.clone()方法

![](https://upload-images.jianshu.io/upload_images/3426615-00df11968b343723.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)<br>

可序列化

![](https://upload-images.jianshu.io/upload_images/3426615-098ae15104dc5253.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)<br>

2. 常量

常量部分在方法中会使用，直接截取源码部分：

* 默认容量为10
```
    private static final int DEFAULT_CAPACITY = 10;
```

* 定义一个数组，存放元素
```
    private static final Object[] EMPTY_ELEMENTDATA = {};
```

* 定义一个数组，用处和上面类似，下文将会说明
```
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
```

* 不序列化的数组，存放元素
```
transient Object[] elementData;
```

* 容器大小
```
private int size;
```
<br>

3. 构造器

构造器共有三种：
> 1. public ArrayList(int initialCapacity) {}
> 2. public ArrayList() {}
> 3. public ArrayList(Collection<? extends E> c) {}

* 给定容量参数的构造器

用到上面所说的常量中的两个

```
public ArrayList(int initialCapacity) {
        //如果给定数值为正的容量值，数组初始化就是用这个值        
        if (initialCapacity > 0) {  
            this.elementData = new Object[initialCapacity];
        //否则就定义一个空的数组
        } else if (initialCapacity == 0) {
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }
```

* 无参构造器

这个简单，直接按照默认容量10，定义一个大小为10的数组

```
public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
```

* 带泛型参数的构造器

```
public ArrayList(Collection<? extends E> c) {
        //先将参数转为数组类型，toArray()方法重载自AbstractCollection<E>和List<E>类
        elementData = c.toArray();
        if ((size = elementData.length) != 0) {
            // c.toArray might (incorrectly) not return Object[] (see 6260652)
            //这是一个官方的bug，说不一定能够返回Object[]类
            if (elementData.getClass() != Object[].class)
                //如果真出现了这个bug，就强制转回Object[]类
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // replace with empty array.
            //如果传入的容器参数的容量为0，就替换为空数组
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }
```
<br>

4. 常用方法

这里先给出的都是平常使用的方法，不包括迭代器的，关于迭代器的内容是需要单独拿一篇来说的

先从**增删改查** 下手，最后会有一个Demo

* 增

增加元素有好几种，末尾添加，定点添加，以及直接添加一整个容器的元素

末尾添加：

```
    //直接在列表末尾添加元素
    public boolean add(E e) {
		//先扩容
        ensureCapacityInternal(size + 1);  // Increments modCount!!
		//把下标为size的位置的值设为e，然后size自增
        elementData[size++] = e;
        return true;
    }
```

末尾添加其他容器元素：

```
public boolean addAll(Collection<? extends E> c) {
        //转换为数组
        Object[] a = c.toArray();
        int numNew = a.length;
        //扩容
        ensureCapacityInternal(size + numNew);  // Increments modCount
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }
```

定点添加：

这里有一个检查数组下标是否越界的方法，需要先说明一下：

```
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```

关于异常信息，源码是这样的：

![](https://upload-images.jianshu.io/upload_images/3426615-edce0a0e0de4cf89.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
    //在指定位置添加元素，可能抛出数组越界，数组存储异常和空指针异常
    public void add(int index, E element) {
        //先检查给定的位置是否越界
        rangeCheckForAdd(index);
        //扩容
        ensureCapacityInternal(size + 1);  // Increments modCount!!
		//将以这个下标开始的元素全部向后复制一位，下文讲解此方法
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
		//复制完数组后，添加元素
        elementData[index] = element;
		//数组大小加1
        size++;
    }
```

关于**System.arraycopy()**，我查了一下源码中的解释：

![](https://upload-images.jianshu.io/upload_images/3426615-14071afc8c702bcb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里面的参数就是：1. 原始数组    2. 原始位置     3. 目标数组    4. 目标位置    5. 需要移动的元素数量 

其实上面添加的方法就是在同一个数组里面移动元素

然后还有直接添加其他容器的元素到指定位置

```
    //在指定位置添加其他容器的元素,可能会抛出空指针异常和数组下标越界异常
    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);
        //将容器转换为数组形式
        Object[] a = c.toArray();
        int numNew = a.length;
        //按照转换后的数组长度来扩容
        ensureCapacityInternal(size + numNew);  // Increments modCount

		//先确定要移动几个数字
        int numMoved = size - index;
        //如果要移动，往后移
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);
		//然后将其他容器的元素复制过来
        System.arraycopy(a, 0, elementData, index, numNew);
        //改变大小
        size += numNew;
		//判断列表结构是否改变
        return numNew != 0;
    }
```
<br>

* 删

关于删除，有定点删除，有删除特定元素，批量删除，还有清空列表，思想就是把要删除的位置的值设为NULL，然后让垃圾回收器处理

定点删除：

首先还是要给出一个检查数组是否越界的方法，和上面的类似：

```
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }
```


```
    //在指定位置删除元素,
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
		//得到要删除的元素
        E oldValue = elementData(index);

		//需要移动的元素数量
        int numMoved = size - index - 1;
        if (numMoved > 0)
            //这些元素全部前移一位
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work

		//返回删除的值
        return oldValue;
    }
```

删除特定元素

需要先说明一个私有方法，下面会用到：

```
    //快速删除，不检查是否有数组下标越界
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        //先删除，再减值
        elementData[--size] = null; // clear to let GC do its work
    }
```

```
    public boolean remove(Object o) {
        //如果没找到特定元素，数组就不变化
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            //遍历数组
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }
```

批量删除：

这个方法是私有的，实现它的有另外两个方法 **removeAll()**和**retainAll()**，批量删除方法中的布尔类型参数就和这两个实现方法有关：

传入一个容器，如果**complement**为true，就只保留和容器元素
相同的元素，如果**complement**为false，就删除和容器元素相同的元素

这个测试中，第一个**complement**为true，第二个**complement**为false：

![](https://upload-images.jianshu.io/upload_images/3426615-08a6bc2d2ae6dec4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
private boolean batchRemove(Collection<?> c, boolean complement) {
        //复制数组来存放筛选后的数据
        final Object[] elementData = this.elementData;
        //r用来遍历数组，w用来给数组赋值
        int r = 0, w = 0;
        //判断结构是否改变
        boolean modified = false;
        try {
            //遍历
            for (; r < size; r++)
                //如果传入的容器含有和原先数组相同的元素，就向新数组赋值
                if (c.contains(elementData[r]) == complement)
                    elementData[w++] = elementData[r];
        } finally {
            // Preserve behavioral compatibility with AbstractCollection,
            // even if c.contains() throws.
            //如果抛出异常，将r位置之后的元素复制到w位置之后
            if (r != size) {
                System.arraycopy(elementData, r,
                                 elementData, w,
                                 size - r);
                w += size - r;
            }
            //在筛选完之后，如果w位置后面有空位，就清理掉
            if (w != size) {
                // clear to let GC do its work
                for (int i = w; i < size; i++)
                    elementData[i] = null;
                modCount += size - w;
                size = w;
                //结构改变了
                modified = true;
            }
        }
        return modified;
    }
```

接下来就是调用批量删除的方法了：

删除容器元素：

```
    //可能抛出强制类型转换异常和空指针异常
    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, false);
    }
```

保留容器元素：

```
    //也可能抛出同样的异常
    public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, true);
    }
```
<br>

清空列表：

removeAll()方法也可以清空列表，只不过效率低，还是推荐使用下面这个：

```
public void clear() {
        modCount++;

        // clear to let GC do its work
        //遍历数组，设值为NULL
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }
```

* 改

可能会抛出数组下标越界的错误

```
public E set(int index, E element) {
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        return oldValue;
    }
```

* 查

可能会抛出数组下标越界的异常

```
public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }
```

