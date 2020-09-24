# Java 多线程基础



## 创建线程的方式

- 继承 Thread 类

    ```java
    Thread t = new Thread() {
        public void run() {
            // 要执行的任务
        }
    };
    t.start();
    ```

    

- 实现 Runnable 接口

    ```java
    // 普通写法
    Runnable task1 = new Runnable() {
        public void run() {
            ...
        }
    };
    //lambda写法
    Runnable task2 = () -> System.out.println("TEST");
    Thread t = new Thread(runnable);
    r.start();
    ```

    

- FutureTask 和 Callable 接口

    ```java
    FutureTask<Integer> task = new FutureTask<>(() -> {
        System.out.println("TEST");
        return 100;
    });
    new Thread(task).start();
    ```



## volatile 关键字

### 作用

- 用 volatile 修饰的变量，保证了其在多线程之间的可见性，即每次读到 volatile 变量，一定是最新的数据
- 在 JVM 运行指令时，会为了获取更好的性能而对指令进行重排序，多线程下可能会出现问题。使用 volatile 则会禁止重排序，但也会一定程度降低代码执行效率



## CAS 和 AQS

### CAS （Compare And Swap）

比较 - 替换，假设有三个操作数，内存值 V、旧的预期值 A、要修改的值 B，当且仅当预期值 A 和内存值 V 相同时，才会将内存值修改为 B 并返回 true，否则什么都不做并返回 false。需要和 volatile 一起使用，保证每次拿到的变量是主内存中最新的值，否则旧的预期值 A 对某条线程来说永远是定值，只要某次 CAS 操作失败，永远都不可能成功。



### AQS（AbstractQuebedSynchronizer）

抽象队列同步器，是整个 Java 并发包的核心，ReentrantLock、CountDownLatch、Semaphore 等等都用到了。AQS 实际上以双向队列的形式链接所有的 Entry ，比如 ReentrantLock，所有等待的线程都被放在一个 Entry 中并连成双向队列。