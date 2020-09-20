# Cookie 和 Session

## Cookie

客户端保存用户信息的一种机制，用来记录用户的一些信息，实际上Cookie是服务器在**本地机器**上存储的一小段文本，并随着每次请求发送到服务器。

Cookie会根据响应报文（来自服务器）里的一个叫做Set-Cookie的首部字段信息，通知客户端保存Cookie。当下客户端再向服务端发起请求时，客户端会自动在请求报文中加入Cookie值之后发送出去.

之后服务端发现客户端发送过来的Cookie后，会检查是那个客户端发送过来的请求，然后对服务器上的记录，最后得到了之前的状态信息。



## Session

session机制是一种服务器端的机制，服务器使用一种类似于散列表的结构来保存信息。

当程序需要为某个客户端的请求创建一个session时，服务器首先检查这个客户端的请求里是否已包含了一个session标识（称为session id），如果已包含则说明以前已经为此客户端创建过session，服务器就按照session id把这个session检索出来使用（检索不到，会新建一个），如果客户端请求不包含session id，则为此客户端创建一个session并且生成一个与此session相关联的session id，session id的值应该是一个既不会重复，又不容易被找到规律以仿造的字符串，这个session id将被在本次响应中返回给客户端保存。



## 区别

- **存取方式的不同**

    Cookie只能保存ASCII字符串，如果需要存取Unicode或二进制，需要先进行编码。

    Session能够存取任何类型的数据，包括String，Integer，List，Map等，甚至Java类。

- **隐私策略的不同**

    Cookie存储在客户端阅读器，对客户端可见，客户端的一些程序可能会窥探复制甚至修改Cookie中的内容。

    Session存储在服务器，对客户端透明，不存在敏感信息泄露的风险。

- **有效期上的不同**

    Cookie可以设置过期时间为很大，接近于永久保存。

    Session依赖于JSESSIONID的Cookie，其过期时间默认为-1，关闭阅读器该Session就会失效，所以Session不能永久保存信息。

- **服务器压力的不同**

    Cookie保存在客户端，不占用服务器资源。对于并发用户多的情景使用方便。

    Session保存在服务端，每个用户都会产生一个Session，假如并发访问的用户十分多，会产生十分多的Session，耗费大量的内存。

- **浏览器支持的不同**

    Cookie需要浏览器支持，如果客户端禁用Cookie，或者不支持Cookie，会话跟踪就会失效。

    假如客户端浏览器不支持Cookie，需要运用Session以及URL地址重写。需要注意的是一切的用到Session程序的URL都要进行URL地址重写，否则Session会话跟踪还会失效。

- **跨域支持上的不同**

    Cookie支持跨域名访问，例如将domain属性设置为“.biaodianfu.com”，则以“.biaodianfu.com”为后缀的一切域名均能够访问该Cookie。

    Session不支持，只能在他所在的域名内访问。





## Reference

- https://juejin.im/post/6844903575684907016
- https://juejin.im/entry/6844903434366222350