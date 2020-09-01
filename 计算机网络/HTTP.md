## HTTP

## HTTP协议
- Hyper Text Transmission Protocol，超文本传输协议
- 客户端与服务端传输的内容称为报文
客户端-->服务端         请求报文
服务端-->客户端         响应报文
- HTTP协议即规定了一种规则，用于规定两种报文格式

### 请求报文格式

1. 例子：
```
Host: www.youtube.com
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:73.0) Gecko/20100101 Firefox/73.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Cookie: VISITOR_INFO1_LIVE=aGRTIWwhyq8; GPS=1; YSC=1-k7OMjUmBE
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0
TE: Trailers
```
2. 报文结构

- 请求首行：
    - 请求方法(Method) 
    - 资源路径(path) 
    - 协议版本(schema)
- 请求的头部信息（键值对）
    - Host：请求的是哪个服务器
    - User-agent：用户代理，显示用户客户端的详细信息
    - Accept：接收文件类型
- 空行
- 请求体
3. GET方法没有请求空行和请求体。POST方法有空行和请求体
另外，GET方法将请求数据放于首行，有长度限制(255)，而POST请求的请求数据位于请求体，适合提交大量内容


### 响应报文格式

1. 例子：
```
HTTP/2 200 OK
strict-transport-security: max-age=31536000
x-content-type-options: nosniff
content-encoding: br
expires: Tue, 27 Apr 1971 19:44:06 GMT
cache-control: no-cache
x-frame-options: SAMEORIGIN
content-type: text/html; charset=utf-8
date: Wed, 08 Apr 2020 01:41:40 GMT
server: YouTube Frontend Proxy
x-xss-protection: 0
alt-svc: quic=":443"; ma=2592000; v="46,43",h3-Q050=":443"; ma=2592000,h3-Q049=":443"; ma=2592000,h3-Q048=":443"; ma=2592000,h3-Q046=":443"; ma=2592000,h3-Q043=":443"; ma=2592000,h3-T050=":443"; ma=2592000
X-Firefox-Spdy: h2
```
2. 报文结构
- 响应首行
    - 协议版本 
    - 响应状态码 
    - 响应提示信息
- 响应头信息
    - server：服务器
    - content-type：服务器传回数据的内容类型
- 空行
- 响应体(服务器传递给客户端的响应数据，如html页面等)

3. 响应状态码

- 各数字开头的含义
    - 2：表示成功
    - 3：表示需要重新请求另一个资源
    - 4：表示资源未找到（请求地址出错）
    - 5：服务器内部错误（代码出错）
- 常见状态码
    - 200：请求成功，浏览器显示响应体内容
    - 404：请求的资源未找到，说明客户端请求了不存在的资源
    - 500：请求资源找到了，但服务器内部出现了错误
    - 302：重定向。服务器要求浏览器重新再发一个请求，服务器会发送一个响应头location，指向了新请求的URL地址




