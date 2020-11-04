## 版本

### HTTP/0.9

HTTP最初的版本。它只有一个方法（GET），没有首部，其设计目标也无非是获取 HTML（也就是说没有图片，只有文本）。

### HTTP/1.0

新增

- `POST`和`HEAD`请求方式
- `Content-Type`
- 状态码
- 重定向
- 内容编码（压缩）。如果正文是压缩过的，通过`Content-Encoding`表明压缩方式；如果未压缩，则不允许返回该字段。客户端通过`Accept-Encoding`表明接受哪些压缩方式

弊端

- 默认短连接，每次传输完数据就会断开连接，即每次请求都需要重新三次握手。可以使用非标准的`Connection: keep-alive`告诉服务器别关闭连接
- 不支持`host`

### HTTP/1.1

新增

- 各种请求方式

- `host`。随着技术的发展，一个物理服务器可以提供多个web服务，但是它们的IP都一样，为了区分具体请求的是哪一个web服务（我的理解就是端口号），增加了`host`字段

- 持久化连接，不需要再声明`Connection: keep-alive`

- 管道机制（流水线）。客户端收到HTTP响应报文之前就能够接着发送新的请求报文（相当于服务器端有一个请求队列，服务器挨个响应），于是一个接一个的请求报文达到服务器后，服务器就可连续发回响应报文。但是，如果前面的响应需要很长时间，则会有"队头堵塞"（Head-of-line blocking）问题，因此浏览器基本上禁用了这个选项，而是通过建立多个TCP连接实现并发传输，但是对一个web服务只能建立少数连接，多了还是要阻塞。并发请求问题到HTTP/2.0得到解决。

  ![image-20201103194151500](https://gitee.com/p8t/picbed/raw/master/imgs/20201103194152.png)

- 断点续传，即分块传输。通过`Range`和`Content-Range`实现。前者用于请求头，表明要传输的指定字节区间。后者用于响应头，表明当前传输的字节区间以及文件总大小。

- 缓存增强

### HTTP/2.0

- 二进制分帧层（Binary Framing Layer）

  HTTP/2.0 将报文分成 HEADERS 帧和 DATA 帧，它们都是二进制格式的。

  ![HTTP/2 binary framing layer](https://gitee.com/p8t/picbed/raw/master/imgs/20201104153601.svg)

  在通信过程中，只会有一个 TCP 连接存在，它承载了任意数量的双向数据流（Stream）。

  - 一个数据流（Stream）都有一个唯一标识符和可选的优先级信息，用于承载双向信息。
  - 消息（Message）是与逻辑请求或响应对应的完整的一系列帧。
  - 帧（Frame）是最小的通信单位，来自不同数据流的帧可以交错发送，然后再根据每个帧头的数据流标识符重新组装。

  ![HTTP/2 streams, messages, and frames](https://gitee.com/p8t/picbed/raw/master/imgs/20201104153949.svg)

- 多路复用 (Multiplexing)

  解决了HTTP/1.1并发请求问题。通过二进制分帧，一个TCP连接可以传输多个流。相当于把原来的多个请求打散并发的在一个TCP连接内传输，这样每一个请求都不会阻塞。这项技术的实现基于二进制分帧层，解决了HTTP/1.1队头阻塞问题。

  ![HTTP/2 request and response multiplexing within a shared connection](https://gitee.com/p8t/picbed/raw/master/imgs/20201104160507.svg)

- 首部压缩（Header Compression）

  HTTP/1.1 的首部带有大量信息，而且每次都要重复发送。

  HTTP/2.0 要求客户端和服务器同时维护和更新一个包含之前见过的首部字段表，从而避免了重复传输。

  不仅如此，HTTP/2.0 也使用 `Huffman `编码对首部字段进行压缩。

  ![Figure 12-6. HPACK: Header Compression for HTTP/2](https://gitee.com/p8t/picbed/raw/master/imgs/20201104154454.svg)

- 服务端推送（Server Push）

  HTTP/2.0 在客户端请求一个资源时，会把相关的资源一起发送给客户端，客户端就不需要再次发起请求了。例如客户端请求 `page.html`页面，服务端就把 `script.js` 和 `style.css` 等与之相关的资源一起发给客户端。

  ![Server initiates new streams (promises) for push resources](https://gitee.com/p8t/picbed/raw/master/imgs/20201104154318.svg)
  
- 流优先级（Stream Prioritization）

  支持多路复用后，流的权重以及依赖项决定了服务器优先响应哪一个流的帧。

  - 每个流可以分配一个整数权重介于 1 和 256 之间
  - 每个流可以指定对另一个流的显式依赖关系

  流依赖项和权重的组合允许客户端构造和通信一个"优先级树"，该树表示它希望如何接收响应。反过来，服务器可以使用此信息通过控制 CPU、内存和其他资源的分配来确定流处理的优先级，一旦响应数据可用，分配带宽以确保对客户端的最佳高优先级响应。

  ![HTTP/2 stream dependencies and weights](https://gitee.com/p8t/picbed/raw/master/imgs/20201104161324.svg)

## 报文

### 请求报文

![image-20201104170246310](https://gitee.com/p8t/picbed/raw/master/imgs/20201104170247.png)

```
GET https://www.bilibili.com HTTP/1.1

accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
accept-encoding: gzip, deflate, br
accept-language: en,zh-CN;q=0.9,zh;q=0.8,en-GB;q=0.7,en-US;q=0.6
cache-control: max-age=0
upgrade-insecure-requests: 1
user-agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.111 Safari/537.36 Edg/86.0.622.61
```

### 响应报文

![image-20201104170319131](https://gitee.com/p8t/picbed/raw/master/imgs/20201104170320.png)

```
HTTP/1.1 200 OK

cache-control: max-age=30
content-encoding: gzip
content-type: text/html; charset=utf-8
date: Wed, 04 Nov 2020 09:04:43 GMT
expires: Wed, 04 Nov 2020 09:05:13 GMT
gear: 1
status: 200
support: nantianmen
vary: Origin,Accept-Encoding
x-cache-webcdn: MISS from cn-jstz-dx-w-01

<!DOCTYPE html>
<html lang="zh-CN">
...
```

## 状态码

| 状态码 |          分类           |     含义     |
| :----: | :---------------------: | :----------: |
|  1xx   | Informational responses | 请求正在处理 |
|  2xx   |  Successful responses   |   成功响应   |
|  3xx   |        Redirects        |    重定向    |
|  4xx   |      Client errors      |  客户端错误  |
|  5xx   |      Server errors      |  服务端错误  |

### 1xx

- **100 Continue** ：这个临时响应表明，迄今为止的所有内容都是可行的，客户端应该继续请求，如果已经完成，则忽略它。

### 2xx

- **200 OK** ：请求成功。
- **204 No Content** ：服务器成功处理了请求，但返回的响应报文不包含响应体部分。一般在只需要从客户端往服务器发送信息，而不需要返回数据时使用。
- **206 Partial Content ** ：表示客户端进行了范围请求（断点续传 / 分块传输），响应报文包含由 `Content-Range` 指定范围的实体内容。

### 3xx

- **301 Moved Permanently** ：永久性重定向

- **302 Found** ：临时性重定向

- **303 See Other** ：和 302 有着相同的功能，但是 303 明确要求客户端应该采用 GET 方法获取资源。

  > 注：虽然 HTTP 协议规定 301、302 状态下重定向时不允许把 POST 方法改成 GET 方法，但是大多数浏览器都会在 301、302 和 303 状态下的重定向把 POST 方法改成 GET 方法。

- **304 Not Modified** ：与HTTP缓存相关。如果请求报文首部包含一些条件，例如：If-Match，If-Modified-Since，If-None-Match，If-Range，If-Unmodified-Since，如果不满足条件，则服务器会返回 304 状态码。
- **307 Temporary Redirect** ：临时重定向，与 302 的含义类似，但是 307 要求浏览器不会把重定向请求的 POST 方法改成 GET 方法。

### 4xx

- **400 Bad Request** ：请求报文中存在语法错误。
- **401 Unauthorized** ：该状态码表示发送的请求需要有认证信息（BASIC 认证、DIGEST 认证）。如果之前已进行过一次请求，则表示用户认证失败。
- **403 Forbidden** ：服务器拒绝此次访问（访问权限有问题，比如反爬虫，黑名单）。
- **404 Not Found** ：服务器上无法找到请求的资源。

### 5xx

- **500 Internal Server Error** ：服务器正在执行请求时发生错误。
- **503 Service Unavailable** ：服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。

## 请求方式

| 方式    | 概述                                |
| ------- | ----------------------------------- |
| GET     | 获取资源                            |
| HEAD    | 和 GET 类似，但是不返回报文实体部分 |
| POST    | 传输数据                            |
| PUT     | 修改资源                            |
| PATCH   | 对资源进行部分修改                  |
| DELETE  | 删除指定的资源                      |
| OPTIONS | 查询支持的方法                      |
| CONNECT |                                     |
| TRACE   |                                     |

## Cookie



## 前端缓存






## 参考资料

[HTTP/2](https://hpbn.co/http2)