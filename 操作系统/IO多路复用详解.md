# I/O 多路复用详解



通过一种机制，一个进程可以监视多个描述符，一旦某个描述符就绪（一般是读就绪或者写就绪），能够通知程序进行相应的读写操作



适用场景：

- 当客户处理多个描述符时（一般是交互式输入和网络套接口），必须使用I/O复用。

- 当一个客户同时处理多个套接口时，而这种情况是可能的，但很少出现。

- 如果一个TCP服务器既要处理监听套接口，又要处理已连接套接口，一般也要用到I/O复用。

- 如果一个服务器即要处理TCP，又要处理UDP，一般要使用I/O复用。

- 如果一个服务器要处理多个服务或多个协议，一般要使用I/O复用。



目前支持I/O多路复用的系统调用主要有 **select，pselect，poll，epoll**



## Select

### 原理

select 函数监视的文件描述符分3类，分别是writefds、readfds、和exceptfds。调用后select函数会阻塞，直到有描述符就绪（有数据 可读、可写、或者有except），或者超时（timeout指定等待时间，如果立即返回设为null即可），函数返回。当select函数返回后，可以通过遍历fdset，来找到就绪的描述符。

### API

```c
// 数据结构 (bitmap)
typedef struct {
    unsigned long fds_bits[__FDSET_LONGS];
} fd_set;

// API
int select(
    int max_fd, 
    fd_set *readset, 
    fd_set *writeset, 
    fd_set *exceptset, 
    struct timeval *timeout
)                              // 返回值就绪描述符的数目

```

### 缺点

- 单个进程所打开的FD是有限制的，通过FD_SETSIZE设置，默认1024
- 每次调用select，都需要把fd集合从用户态拷贝到内核态，这个开销在fd很多时会很大
- 对socket扫描时是线性扫描，采用轮询的方法，效率较低（高并发时）



## Poll

### 原理

和Select基本相同，将用户传入的数组拷贝到内核空间，然后查询每个fd对应得设备状态，如果设备就绪则在设备等待队列中加入一项并继续遍历，如果遍历完所有fd后没有发现就绪设备，则挂起当前线程，知道设备就绪或者主动超时，被唤醒后又再次遍历fd，这个过程经历多次无谓的遍历。相比select，poll没有最大连接数限制，因为是基于链表来存储。

### API

```C
// 数据结构
struct pollfd {
    int fd;                         // 需要监视的文件描述符
    short events;                   // 需要内核监视的事件
    short revents;                  // 实际发生的事件
};

// API
int poll(struct pollfd fds[], nfds_t nfds, int timeout);

```

### 缺点

1. 大量的fd的数组被整体复制于用户态和内核地址空间之间，而不管这样的复制是不是有意义。
2. poll还有一个特点是“水平触发”，如果报告了fd后，没有被处理，那么下次poll时会再次报告该fd



## Epoll

### 原理

epoll使用一个文件描述符管理多个描述符，将用户关系的文件描述符的事件存放到内核的一个事件表中，这样在用户空间和内核空间的copy只需一次。

epoll支持水平触发和边缘触发，最大的特点在于边缘触发，它只告诉进程哪些fd刚刚变为就绪态，并且只会通知一次。还有一个特点是，epoll使用“事件”的就绪通知方式，通过epoll_ctl注册fd，一旦该fd就绪，内核就会采用类似callback的回调机制来激活该fd，epoll_wait便可以收到通知。

### API

```C
// 数据结构
// 每一个epoll对象都有一个独立的eventpoll结构体
// 用于存放通过epoll_ctl方法向epoll对象中添加进来的事件
// epoll_wait检查是否有事件发生时，只需要检查eventpoll对象中的rdlist双链表中是否有epitem元素即可
struct eventpoll {
    /*红黑树的根节点，这颗树中存储着所有添加到epoll中的需要监控的事件*/
    struct rb_root  rbr;
    /*双链表中则存放着将要通过epoll_wait返回给用户的满足条件的事件*/
    struct list_head rdlist;
};

// API

int epoll_create(int size); // 内核中间加一个 ep 对象，把所有需要监听的 socket 都放到 ep 对象中
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event); // epoll_ctl 负责把 socket 增加、删除到内核红黑树
int epoll_wait(int epfd, struct epoll_event * events, int maxevents, int timeout);// epoll_wait 负责检测可读队列，没有可读 socket 则阻塞进程

```

### 优点

- 没有最大并发连接的限制，能打开的fd的上限远大于1024（1G的内存上能监听约10万个端口）。

- 效率提升，不是轮询的方式，不会随着FD数目的增加效率下降。只有活跃可用的FD才会调用callback函数；即Epoll最大的优点就在于它只管你“活跃”的连接，而跟连接总数无关，因此在实际的网络环境中，Epoll的效率就会远远高于select和poll。

- 内存拷贝，利用mmap()文件映射内存加速与内核空间的消息传递；即epoll使用mmap减少复制开销。

### 缺点

- Linux才有

### LT 和 ET模式

epoll对文件描述符的操作有两种模式，即水平触发模式（level trigger）和边缘触发模式（edge trigger），默认为LT模式。

#### LT模式

当 epoll_wait 检测到描述符事件发生并将此事件通知应用程序，应用程序可以不立即处理该事件。下次调用 epoll_wait 时，会再次响应应用程序并通知此事件。

在这种做法中，内核告诉你一个文件描述符是否就绪了，然后你可以对这个就绪的 fd 进行 IO 操作。如果你不作任何操作，内核还是会继续通知你的。

同时支持block和no-block socket。

#### ET模式

当 epoll_wait 检测到描述符事件发生并将此事件通知应用程序，应用程序必须立即处理该事件。如果不处理，下次调用 epoll_wait 时，不会再次响应应用程序并通知此事件。

在这种模式下，当描述符从未就绪变为就绪时，内核通过epoll告诉你。然后它会假设你知道文件描述符已经就绪，并且不会再为那个文件描述符发送更多的就绪通知，直到你做了某些操作导致那个文件描述符不再为就绪状态了(比如，你在发送，接收或者接收请求，或者发送接收的数据少于一定量时导致了一个 EWOULDBLOCK 错误）。但是请注意，如果一直不对这个 fd 作 IO 操作(从而导致它再次变成未就绪)，内核不会发送更多的通知 ( only once )。

**必须使用非阻塞套接口**，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死





## 三者区别

|              |                            select                            |             poll             |                            epoll                             |
| :----------: | :----------------------------------------------------------: | :--------------------------: | :----------------------------------------------------------: |
|   数据结构   |                            bitmap                            |           链表数组           |                            红黑树                            |
|  最大连接数  | 由 FD_SETSIZE 宏定义，大小为32个证书的大小（32位机器上即32\*32，64位机器上是32\*64），如果修改则需重新编译内核，可能影响性能 | 无上限，由于是链表的存储方式 |    虽然连接数有上限，但很大，1G内存可以打开10万左右的连接    |
|  fd拷贝方式  |                      每次调用select拷贝                      |           同select           |      fd首次调用epoll_ctl拷贝，每次调用epoll_wait不拷贝       |
| 消息传递方式 |         内核需要将消息传递到用户空间，都需要拷贝动作         |           同select           |          epoll通过内核和用户空间共享一块内存来实现           |
|     效率     | O(n)，因为每次调用时都会对连接进行线性遍历，随着fd的增加会造成遍历速度慢，即轮询 |           同select           | O(1)，epoll内核中实现是根据每个fd上的callback函数来实现，只有活跃的socket才会主动调用callback。所以在活跃fd较少的情况（即有较多idle-connection或dead-connection），使用epoll没有前两者的性能下降问题，但是在所有socket都很活跃的情况下，可能会有性能问题，因为epoll通知机制需要很多函数回调 |



## Reference

- https://www.jianshu.com/p/dfd940e7fca2
- https://juejin.im/post/6844904200141438984