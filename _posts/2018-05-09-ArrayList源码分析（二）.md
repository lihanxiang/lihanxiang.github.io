---
layout:		post
title:		ArrayList源码分析（二）
subtitle:	读读源码
date:		2018-05-09
author:		L
header-img:	img/post/2018-05/9.jpg
tags:
    - Java源码解析
    - 集合类

---
### 正文之前

> 上一篇文章分析了[ArrayList的构造器和基本操作的源码](http://hxblog.site/2018/05/04/ArrayList%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E4%B8%80/)，这一次来讲一讲ArrayList常用方法剩下的源码（迭代器的留到下次说）
>
> 主要内容
> 1. 容器容量
> 2. 查找特定元素，包含特定元素，克隆之类的基本用法

### 正文

开始正题之前，需要先分清楚两个概念，容器的capacity和size，capacity指的是容器的容量，最多能装多少，而size指的是当前容器中元素的数量

接下来直奔正题

#### 1. 扩容

* 设定容量

```
    public void ensureCapacity(int minCapacity) {
        //如果初始时有设定容量，就按设定的来，没有就按默认的来
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default element table
            ? 0
            // larger than default for default empty table. It's already
            // supposed to be at default size.
            : DEFAULT_CAPACITY;
        //如果给定的比上述的要大，就按给定的来
        if (minCapacity > minExpand) {
            //这个方法在下面
            ensureExplicitCapacity(minCapacity);
        }
    }
    //私有方法，用来确定容量
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
        //增加容量
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
```

* 增长

```
/**
     * The maximum size of array to allocate.
     * Some VMs reserve some header words in an array.
     * Attempts to allocate larger arrays may result in
     * OutOfMemoryError: Requested array size exceeds VM limit
     */
    //注解说是因为有的虚拟机要保留数组的前几位，这个常量的值为2147483639
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * Increases the capacity to ensure that it can hold at least the
     * number of elements specified by the minimum capacity argument.
     *
     * @param minCapacity the desired minimum capacity
     */
    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        //右移一位，相当于除以2
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //两者相比，用更大的一个作为容量
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        //容量特别大时的操作
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        //按照新容量复制数组
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

* 大容量

```
    private static int hugeCapacity(int minCapacity) {
        //溢出
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```
<br>

#### 2. 调整大小

如果发现容量多了很多，有点浪费，就可以调整容器大小到当前大小

```
    public void trimToSize() {
        modCount++;
        //如果确实有多余部分
        if (size < elementData.length) {
            //如果size不为0，则按照size复制数组
            elementData = (size == 0)
              ? EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size);
        }
    }
```
<br>

#### 3. 列表元素数量

```
    public int size() {
        return size;
    }
```
<br>

#### 4. 列表是否为空

```
    public boolean isEmpty() {
        return size == 0;
    }
```
<br>

#### 5. 查找列表中特定元素的位置

```
    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            //遍历数组查找
            for (int i = 0; i < size; i++)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```
<br>

#### 6. 判断列表是否包含特定元素

```
    public boolean contains(Object o) {
        return indexOf(o) >= 0;
    }
```
<br>

#### 7. 如果列表中有多个特定元素

```
    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            //从尾部开始遍历 
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }
```
<br>

#### 8. 克隆

需要先说明一个概念：浅拷贝和深拷贝

浅拷贝，两个ArrayList在内存中的地址是不一样的，但是他们中的元素都指向同一个地址，手绘一个图解释一下：

![](https://upload-images.jianshu.io/upload_images/3426615-fe5135fa0eac5629.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

深拷贝，不仅ArrayList在内存中的地址是不一样的，他们中的元素也不指向同一个地址，也手绘个图解释一下：

![](https://upload-images.jianshu.io/upload_images/3426615-2eaa529c4d75f2d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

深拷贝可能好理解一些，就是所有的东西都拷贝一下，完全分离的状态

ArrayList的**clone()**方法是浅拷贝，我们来做个实验：

```
public class Test {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            list.add(String.valueOf(i));
        }
        //强制类型转换
        ArrayList<String> list1 = (ArrayList<String>)list.clone();
        System.out.println("list" + list);
        System.out.println("list1" + list1);
        System.out.println("二者对于某个特定元素是否相同 " + (list.get(1) == list1.get(1)));
        System.out.println("二者的地址是否相同 " + (list == list1));
    }
}
```

![](https://upload-images.jianshu.io/upload_images/3426615-c982020574740d1c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<br>

#### 9. 转换为数组

ArrayList是一个列表，和数组不一样，不能直接用下标访问元素，转换为数组就可以了，转换为数组有两种方法：

```
    //按照列表中元素的顺序，将列表中元素复制到一个数组
    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }
```

![](https://upload-images.jianshu.io/upload_images/3426615-2758119e63ef391c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<br>

这一种方法可能会抛出数组存储异常和空指针异常，这里讲述两种情况（数组大小小于列表元素数量）：

1. 传入的数组类型和列表元素相同或者是列表元素的基类型，直接将列表中元素赋值在数组内

2. 传入的数组类型不是列表元素的基类型，抛出数组存储异常

```
    public <T> T[] toArray(T[] a) {
        //传入的数组大小更小
        if (a.length < size)
            //创建一个数组，类型是数组a的运行时类型
            // Make a new array of a's runtime type, but my contents:
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }
```

传入的数组 s 是Object类型的，符合上述的情况一：

![](https://upload-images.jianshu.io/upload_images/3426615-47e4fb7279f616f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来是情况二：

![](https://upload-images.jianshu.io/upload_images/3426615-3a3506b06f27cb29.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样子，ArrayList常用的方法就讲完了，下一篇就会是与ArrayList有关的迭代器的使用了
