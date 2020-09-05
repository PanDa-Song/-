# HashMap


## 概述

- java7: 数组 + 链表； java8: 数组 + 红黑树


## 插入元素过程

![](https://img-blog.csdnimg.cn/2020031712385760.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poZW5nd2FuZ3p3,size_16,color_FFFFFF,t_70)

1. 判断数组是否为空，为空进行初始化;
2. 不为空，计算 k 的 hash 值，通过(n - 1) & hash计算应当存放在数组中的下标 index;
3. 查看 table[index] 是否存在数据，没有数据就构造一个Node节点存放在 table[index] 中；
4. 存在数据，说明发生了hash冲突(存在二个节点key的hash值一样), 继续判断key是否相等，相等，用新的value替换原数据(onlyIfAbsent为false)；
5. 如果不相等，判断当前节点类型是不是树型节点，如果是树型节点，创造树型节点插入红黑树中；(如果当前节点是树型节点证明当前已经是红黑树了)
6. 如果不是树型节点，创建普通Node加入链表中；判断链表长度是否大于 8并且数组长度大于64， 大于的话链表转换为红黑树；
7. 插入完成之后判断当前节点数是否大于阈值，如果大于开始扩容为原数组的二倍。


## 初始化

如果未给定初始容量，则默认容量16，负载因子0.75，若传入初始参数，则将其转化为大于他最近的**2的幂指数**

```java
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

## 计算hash值

``` java
static final int hash(Object key) {
    int h;
    // 1. h = key的 32 位 hashcode值
    // 2. h 和 h 向右移16位做异或操作
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```
设计的原因：
1. 尽可能降低hash碰撞，越分散越好；
2. 算法尽可能高效，因为这是高频操作, 因此采用位运算

>右移16位进行异或操作，使自身的前半部分和后半部分混合，加大低位（后半部分）的的随机性并且保留了部分高位的特征


## 插入元素操作

```java
if ((p = tab[i = (n - 1) & hash]) == null)
    tab[i] = newNode(hash, key, value, null);
```
在插入时，元素下标的选择是`n-1 & hash`的方法，因此n也就是容量取为2的整数次幂可以让n-1的值二进制全为1，起到一个掩码的作用

## 1.7和1.8对比

1. 数组+链表改成了数组+链表或红黑树；
    - 防止发生hash冲突，链表长度过长，将时间复杂度由O(n)降为O(logn)



2. 链表的插入方式: 从头插法-->尾插法
    - 插入时，如果数组位置上已经有元素，1.7将新元素放到数组中，原始节点作为新节点的后继节点，1.8遍历链表，将元素放置到链表的最后
    - 1.7头插法扩容时，头插法会使链表发生反转，多线程环境下会产生环；
3. 扩容的时候1.7需要对原数组中的元素进行重新hash定位在新数组的位置，1.8采用更简单的判断逻辑，位置不变或索引+旧容量大小；
    - 因为扩容是扩为原有的容量的2倍，用于计算数组位置的掩码正好仅仅是高位多了一个1，若元素的该高位为0，则rehash数值不变，若为1，则rehash值
4. 在插入时，1.7先判断是否需要扩容，再插入，1.8先进行插入，插入完成再判断是否需要扩容；





