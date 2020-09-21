# GET 和 POST 详解

![](https://github.com/PanDa-Song/CS-basic-notes/raw/master/Resources/GET%26POST.png)



## 概括

- GET用于获取信息，无副作用，幂等，且可缓存
- POST用于修改服务器上的数据，有副作用，非幂等，不可缓存

> 副作用（安全性）：指请求方法是否会破坏（秀改）服务器上的资源
>
> 幂等：多次执行相同的操作，结果是否相同

## 报文上的区别

- GET方法没有请求空行和请求体。

    POST方法有空行和请求体

- GET方法将请求数据（参数）放于首行与URL一起，有长度限制(255)。

    POST请求的请求数据位于请求体，适合提交大量内容

