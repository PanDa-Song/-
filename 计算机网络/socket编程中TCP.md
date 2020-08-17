## socket编程中的API

## 实现流程

### TCP:
- server

    socket() -> bind() -> listen() -> accept() -> read()/recv() -> write()/send() -> close()

- client

    socket() -> connect() -> write()/send() -> read()/recv() -> close()

### UDP

不区分server与client，两者等价。
直接connect()


## 三次握手和四次挥手

- connect会完成TCP的三次握手，客户端调用connect后，由内核中的TCP协议完成TCP的三次握手；

- close操作会完成四次挥手

