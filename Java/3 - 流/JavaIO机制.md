# Java IO流



## BIO

即 Block-IO：InputStream 和 OutputStream，Reader 和 Writer，以及java.net包下的socket等。主要基于字节流字符流



## NIO

NonBlock-IO：主要基于Channel，Selectors，Buffers等构建多路复用的、同步非阻塞的IO操作

![](../Resources/java-selector.jpg)



## AIO

基于事件和回调机制  