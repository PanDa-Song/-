# ReentrantLock

位于 java.util.concurrent.locks 包，基于 AQS 实现



## 原理





## 区别

- synchronized 是关键字，ReentrantLock 是类
- ReentrantLock 可以对获取锁的等待时间进行设置，避免死锁
- ReentrantLock 可以获取锁的各种信息

- ReentrantLock 可以灵活实现多路通知
- 底层原理上，synchronized 操作对象头的 Mark Word，ReentrantLock 调用 Unsafe 类的 park() 方法