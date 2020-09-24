## ConcurrentHashMap



HashMap的线程安全实现主要有Hashtable，SynchronizedMap，ConcurrentHashMap。



## Hashtable 和 SychronizedMap

### Hashtable 

Hashtable 是早期 Java 类库提供的哈希表实现，基于 Dictionary 类，其线程安全的方法是在涉及到修改 Hashtable 的方法，使用 synchronized 修饰。但由于其串行化方式运行，性能较差

### SynchronizedMap

SynchronizedMap 是Collections 包下的线程安全类。他的加锁方式是有一个 mutex 对象作为互斥锁，在每个 public 方法下都用 synchronized 关键字对该对象加锁。本质上和 Hashtable 实现方法一样，只是加锁的对象略有不同。



## ConcurrentHashMap

主要方法：CAS + synchronized 使锁更细化



### put

不允许 key 为 null

1. 判断 Node[] 数组是否初始化，没有则进行初始化操作

2. 通过 hash 定位数组的索引坐标，是否有Node 节点，如果没有则用 CAS 进行添加（链表的头结点），添加失败则进入下次循环

3. 若检查到内部正在扩容，就帮助他一块扩容

4. 如果 f != null，则使用 synchronized 锁住 f 元素（链表/红黑树的头元素）

5. 如果是 Node 则执行链表的添加操作

6. 如果是 TreeNode 则执行树添加操作

7. 判断链表长度已经达到临界值 8，当超过时把链表传换成树