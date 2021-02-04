## 一、版本

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

- 管道机制（流水线）。客户端收到HTTP响应报文之前就能够接着发送新的请求报文（相当于服务器端有一个请求队列，服务器挨个响应），于是一个接一个的请求报文达到服务器后，服务器就可连续发回响应报文。但是，如果前面的响应需要很长时间，则会有"队头堵塞"（Head-of-line blocking）问题，因此浏览器基本上禁用了这个选项，而是通过建立多个TCP连接实现并发传输，但是对一个web服务只能建立少数连接（chrome下最多建立6个连接），多了还是要阻塞。并发请求问题到HTTP/2.0得到解决：通过多路复用，多个资源不是串行传输，而是并行传输，可以根据流优先级先传输重要的资源。

  ![image-20201103194151500](https://gitee.com/p8t/picbed/raw/master/imgs/20201103194152.png)

- 断点续传，即分块传输。通过`Range`和`Content-Range`实现。前者用于请求头，表明要传输的指定字节区间。后者用于响应头，表明当前传输的字节区间以及文件总大小。

- 缓存增强

### HTTP/2.0

- 二进制分帧层（Binary Framing Layer）

  HTTP/2.0 将报文分成 HEADERS 帧和 DATA 帧，它们都是二进制格式的。

  ![image-20201105222734931](https://gitee.com/p8t/picbed/raw/master/imgs/20201105222736.png)

  在通信过程中，只会有一个 TCP 连接存在，它承载了任意数量的双向数据流（Stream）。

  - 一个数据流（Stream）都有一个唯一标识符和可选的优先级信息，用于承载双向信息。

  - 消息（Message）是与逻辑请求或响应对应的完整的一系列帧。

  - 帧（Frame）是最小的通信单位，来自不同数据流的帧可以交错发送，然后再根据每个帧头的数据流标识符重新组装。

    ![image-20201105222800618](https://gitee.com/p8t/picbed/raw/master/imgs/20201105222801.png)

- 多路复用 (Multiplexing)

  解决了HTTP/1.1并发请求问题。通过二进制分帧，一个TCP连接可以传输多个流。相当于把原来的多个请求打散并发的在一个TCP连接内传输，这样每一个请求都不会阻塞。这项技术的实现基于二进制分帧层，解决了HTTP/1.1队头阻塞问题。

  ![image-20201105222824260](https://gitee.com/p8t/picbed/raw/master/imgs/20201105222825.png)

- 首部压缩（Header Compression）

  HTTP/1.1 的首部带有大量信息，而且每次都要重复发送。

  HTTP/2.0通过简化首部字段表示来减少冗余

  静态表：对于常用的请求key用数字来代替，比如User-Agent用数字5来代表。

  动态表：对于不常用的请求key，可以让客户端和服务器同时维护和更新一个包含之前见过的首部字段表，从而避免了重复传输。

  不仅如此，HTTP/2.0 也使用 `Huffman `编码对首部字段进行压缩。

  ![image-20201105222902434](https://gitee.com/p8t/picbed/raw/master/imgs/20201105222903.png)

- 服务端推送（Server Push）

  HTTP/2.0 在客户端请求一个资源时，会把相关的资源一起发送给客户端，客户端就不需要再次发起请求了。例如客户端请求 `page.html`页面，服务端就把 `script.js` 和 `style.css` 等与之相关的资源一起发给客户端。

  ![image-20201105222923268](https://gitee.com/p8t/picbed/raw/master/imgs/20201105222924.png)
  
- 流优先级（Stream Prioritization）

  支持多路复用后，流的权重以及依赖项决定了服务器优先响应哪一个流的帧。

  - 每个流可以分配一个整数权重介于 1 和 256 之间
  - 每个流可以指定对另一个流的显式依赖关系

  流依赖项和权重的组合允许客户端构造和通信一个"优先级树"，该树表示它希望如何接收响应。反过来，服务器可以使用此信息通过控制 CPU、内存和其他资源的分配来确定流处理的优先级，一旦响应数据可用，分配带宽以确保对客户端的最佳高优先级响应。

  ![image-20201105222941590](https://gitee.com/p8t/picbed/raw/master/imgs/20201105222942.png)

## 二、HTTP报文

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

[HTTP Message](https://www.w3.org/Protocols/rfc2616/rfc2616-sec4.html)

## 三、状态码

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

## 四、HTTP首部

分为4种

- 通用首部字段（General headers）
- 请求首部字段（Request headers）
- 响应首部字段（Response headers）
- 实体首部字段（Entity headers）

### 通用首部字段

| 首部字段名        | 说明                                       |
| ----------------- | ------------------------------------------ |
| Cache-Control     | 控制缓存的行为                             |
| Connection        | 控制不再转发给代理的首部字段、管理持久连接 |
| Date              | 创建报文的日期时间                         |
| Pragma            | 报文指令                                   |
| Trailer           | 报文末端的首部一览                         |
| Transfer-Encoding | 指定报文主体的传输编码方式                 |
| Upgrade           | 升级为其他协议                             |
| Via               | 代理服务器的相关信息                       |
| Warning           | 错误通知                                   |

### 请求首部字段

| 首部字段名          | 说明                                            |
| ------------------- | ----------------------------------------------- |
| Accept              | 用户代理可处理的媒体类型                        |
| Accept-Charset      | 优先的字符集                                    |
| Accept-Encoding     | 优先的内容编码                                  |
| Accept-Language     | 优先的语言（自然语言）                          |
| Authorization       | Web 认证信息                                    |
| Expect              | 期待服务器的特定行为                            |
| From                | 用户的电子邮箱地址                              |
| Host                | 请求资源所在服务器                              |
| If-Match            | 比较实体标记（ETag）                            |
| If-Modified-Since   | 比较资源的更新时间                              |
| If-None-Match       | 比较实体标记（与 If-Match 相反）                |
| If-Range            | 资源未更新时发送实体 Byte 的范围请求            |
| If-Unmodified-Since | 比较资源的更新时间（与 If-Modified-Since 相反） |
| Max-Forwards        | 最大传输逐跳数                                  |
| Proxy-Authorization | 代理服务器要求客户端的认证信息                  |
| Range               | 实体的字节范围请求                              |
| Referer             | 对请求中 URI 的原始获取方                       |
| TE                  | 传输编码的优先级                                |
| User-Agent          | HTTP 客户端程序的信息                           |

### 响应首部字段

| 首部字段名         | 说明                         |
| ------------------ | ---------------------------- |
| Accept-Ranges      | 是否接受字节范围请求         |
| Age                | 推算资源创建经过时间         |
| ETag               | 资源的匹配信息               |
| Location           | 令客户端重定向至指定 URI     |
| Proxy-Authenticate | 代理服务器对客户端的认证信息 |
| Retry-After        | 对再次发起请求的时机要求     |
| Server             | HTTP 服务器的安装信息        |
| Vary               | 代理服务器缓存的管理信息     |
| WWW-Authenticate   | 服务器对客户端的认证信息     |

### 实体首部字段

| 首部字段名       | 说明                   |
| ---------------- | ---------------------- |
| Allow            | 资源可支持的 HTTP 方法 |
| Content-Encoding | 实体主体适用的编码方式 |
| Content-Language | 实体主体的自然语言     |
| Content-Length   | 实体主体的大小         |
| Content-Location | 替代对应资源的 URI     |
| Content-MD5      | 实体主体的报文摘要     |
| Content-Range    | 实体主体的位置范围     |
| Content-Type     | 实体主体的媒体类型     |
| Expires          | 实体主体过期的日期时间 |
| Last-Modified    | 资源的最后修改日期时间 |

## 五、请求方式

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

## 六、Cookie

HTTP 协议是无状态的，主要是为了让 HTTP 协议尽可能简单，使得它能够处理大量事务。HTTP/1.1 引入 Cookie 来保存状态信息。

Cookie 是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器之后向同一服务器再次发起请求时被携带上，用于告知服务端两个请求是否来自同一浏览器。由于之后每次请求都会需要携带 Cookie 数据，因此会带来额外的性能开销（尤其是在移动环境下）。

Cookie 曾一度用于客户端数据的存储，因为当时并没有其它合适的存储办法而作为唯一的存储手段，但现在随着现代浏览器开始支持各种各样的存储方式，Cookie 渐渐被淘汰。新的浏览器 API 已经允许开发者直接将数据存储到本地，如使用 Web storage API（本地存储和会话存储）或 IndexedDB。

### 流程

![image-20201105160309547](https://gitee.com/p8t/picbed/raw/master/imgs/20201105160311.png)

```
Server: GitHub.com
Set-Cookie: user_session=Nerl6jr-NC7Kiz_dGxUxO-QaszGb3O4O69D42EEkn70rlma; path=/; expires=Thu, 19 Nov 2020 08:15:41 GMT; secure; HttpOnly; SameSite=Lax
Set-Cookie: __Host-user_session_same_site=Nerl6jr-NC7Kiz_dGxUxO-QasqGb3O4O69D42EEkn70rlma; path=/; expires=Thu, 19 Nov 2020 08:15:41 GMT; SameSite=Strict; secure; HttpOnly
Set-Cookie: has_recent_activity=1; path=/; expires=Thu, 05 Nov 2020 09:15:41 GMT; secure; HttpOnly; SameSite=Lax
Set-Cookie: _gh_sess=bIr7RpQFXOu%2BmrkbyVTDiZSShtQVrht%2BMKqIwcTHqt8GU6ec7aKg9Z2%2BVSKbw80VHSCmikIRtlhRIDK2N%2BZLTE4cnL3neGPG%2Fk4MqEXxgu44ykXWUYsFB%2F%2Fw7NdvghZgHgFI%2FLWwlx4ju2JzW3tMX%2BvUZjhPzZRx%2FuZ%2FdMNsZIpgs0zTQ43FUXTH9c8%2FOMjuZBxjs4R1NcM1rZnjj%2BSaqlzJGDagLsm5H0xCcm0n9Xn3ubB8MZRh2wKsycSNgUj0uGUuy67eiLc%2BIN%2BffX0jlSW3ZtfwuV9jwBnnVm0eFyvV7NVjpZwsHrekJsRGUdb0FMYz2t17a5Nz5e1nfWfUXp1ySZCLt2vbzX4Y1XnWckxJOSkwhBRuHqgz%2FhjocEcTxni4SVsEisgr%2BS6JYf6iMgibjTz%2BqTIRFiv%2FnC%2BnBPxDIcj8iG%2FlgLwUM0zctgfeXmckITj%2Fu2JsHmnb3USSFTMORz8FW%2FzBQl7O%2BKLcRuo9T3guEWr4jPWrwPWef0CNT99A51C6XVWEBuZuAbyX%2B4Z6jVNACmaTseCH%2F2mtnw5lgnwXAhMa2ZepdYCCEbu%2FPsGp5qbudh4Dv%2F6VlpCPFeNM3wJPyen%2BUKGxbwalwYBwx%2BGcMQ2DN1wLXH5rm1p%2BpTG9iC0JJi5XNs6k6MDXoPVpIzI%2B0hudznbSmFSs9%2FIELzM0S9L1BQoLDYjq090KsnDEEjDDs2uO%2Fu2cD9pNrt7CiPE%2BTrIFtUKL9U0KW7mKwALLbWLxxyyeQF4tCWnOL51kc4VlPSFgJJc--ZJ0RpG7aKmhSVYWD--KZXmZqpk1TPGS%2Fq8SLTMVA%3D%3D; path=/; secure; HttpOnly; SameSite=Lax
```

```
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate, br
Accept-Language: en,zh-CN;q=0.9,zh;q=0.8,en-GB;q=0.7,en-US;q=0.6
Cache-Control: max-age=0
Connection: keep-alive
Cookie: _octo=GH1.1.21530934.1603156343; _ga=GA1.2.1496646255.160156346; tz=Asia%2FShanghai; _device_id=83431d990fa0ca582f1784abe0a523ba; has_recent_activity=1; user_session=Nerl6jr-NC7Kiz_dGxUxO-QasqzGb3O4O69D42EEkn70rlma; tz=Asia%2FShanghai; logged_in=yes; dotcom_user=p8tz; _gh_sess=4Ongo9JmLkm5bx4qVNET7WdPnZp2fdGUMNbWabIe7SbZfWqAqZuugAvRA7Jg0nZIbaZAhM74bmPXc4qMvSWoJQdogUUohYtlwzL8319m%2BfK3fzKgexB9NBNGEJpJ4xRAZJL97mrEoGLJ0wAWbzbeR2fwDAlVO8IqYzuv5ce4mj6PXrh%2FpAQBZsMpbsXxJN3MV32qHnfTxusAzlicHbDJ7M%2BY7VBDlUSlnrt3EIyzgMByM60OziPsqc0K7BU9Olm06jPOJTSCd00fX6hc9%2B5RHy6zIfnKwPHan8VyNmN0xSbZCAwlc2Ww%2B9OvEbovCL6At65oOmOTiKfNAn5ZCvQylsfK8jd2OvZyoOVeqVrUUs6aIV4kGVyejkvOl9kG11154HDR5IoqPAal0eGzUXwENn%2BW423MUcABmp0MvTigZ60db7%2BheKIYgeWnMBVx72btjeXM5bJz9UuVpWCN7s%2Fcz58h2IH7wVO817wOGH%2FBfs24e6HJR42Um%2FWb3%2FfQUnV%2F2WibgP8QnwihVf0WPBBLs%2FYlSVcUmiTgwIlS2hFvlxklPvsm8VbQpzSDyqAeQqPBiEJuwg1HBCth2kvb2EpqVbmZVJJp1r8PRo8PSM%2FIqq4FKpQscTPFDKW99eqd8JkeD5yAVYrgduDV%2FbwZzcc3ay%2Fe%2Fpit830Of%2B2zKSff5acEFKQ%2BbcM7%2BwEMRzh8gEIYXQCZZJrLhEbl4cPdoLMY95v2uXxmNZ4Et3DsShGLlg1sELdlwHWeFjBCJZQpn%2Fj2lygx8VKls3xWDir9--4l8L5nWuZ%2BoRusRR--4YDRiPeKkReT%2FgzO5Pt6RA%3D%3D
Host: github.com
```

### 分类

- 会话期 Cookie：浏览器关闭之后它会被自动删除，也就是说它仅在会话期内有效。
- 持久性 Cookie：指定过期时间（Expires）或有效期（max-age）之后就成为了持久性的 Cookie。

```html
Set-Cookie: expires=Thu, 05 Nov 2020 09:15:41 GMT;
```

### 作用域

Domain 标识指定了哪些主机可以接受 Cookie。如果不指定，默认为当前文档的主机（不包含子域名）。如果指定了 Domain，则一般包含子域名。例如，如果设置 Domain=mozilla.org，则 Cookie 也包含在子域名中（如 developer.mozilla.org）。

Path 标识指定了主机下的哪些路径可以接受 Cookie（该 URL 路径必须存在于请求 URL 中）。以字符 %x2F ("/") 作为路径分隔符，子路径也会被匹配。例如，设置 Path=/docs，则以下地址都会匹配：

- /docs
- /docs/Web/
- /docs/Web/HTTP

```
Set-Cookie: path=/;
```

### HttpOnly

标记为 HttpOnly 的 Cookie 不能被 JavaScript 脚本调用。跨站脚本攻击 (XSS) 常常使用 JavaScript 的 `document.cookie` API 窃取用户的 Cookie 信息，因此使用 HttpOnly 标记可以在一定程度上避免 XSS 攻击。

```html
Set-Cookie: secure; HttpOnly; SameSite=Lax
```

### Secure

标记为 Secure 的 Cookie 只能通过被 HTTPS 协议加密过的请求发送给服务端。但即便设置了 Secure 标记，敏感信息也不应该通过 Cookie 传输，因为 Cookie 有其固有的不安全性，Secure 标记也无法提供确实的安全保障。

```
Set-Cookie: secure; HttpOnly; SameSite=Lax
```

## 七、Session

可以通过Cookie把信息存在浏览器中，也可以通过Session存在服务器中，更加安全。

流程如下

- 用户登录后，服务端为用户生成一个SessionID
- 通过set-cookie的方式传到客户端，保存在浏览器中
- 用户接下来的请求在Cookie中携带SessionID，服务器就能区分出不同用户

一般来说，Session的本质就是通过Cookie实现在服务端存储用户信息的手段，并不是实际存在的东西，而Cookie是实打实存在的。

如果浏览器禁用Cookie，则只能通过别的方式实现Session，比如通过url传递

## 八、前端缓存

### 分类

- Service Worker
- Memory Cache
- Disk Cache
- 网络请求

请求顺序从上至下

### 1、memory cache

网页上几乎所有的资源都进memory cache，短期存储。常规情况下，浏览器的 TAB 关闭后该次浏览的 memory cache 便告失效。而如果极端情况下 (例如一个页面的缓存就占用了超级多的内存)，那可能在 TAB 没关闭之前，排在前面的缓存就已经失效了。

memory cache 机制保证了一个页面中如果有两个相同的请求 (例如两个 `src` 相同的 `<img>`，两个 `href` 相同的 `<link>`) 都实际只会被请求最多一次，避免浪费。

在从 memory cache 获取缓存内容时，浏览器会忽视例如 `max-age=0`, `no-cache` 等头部配置。例如页面上存在几个相同 `src` 的图片，即便它们可能被设置为不缓存，但依然会从 memory cache 中读取。这是因为 memory cache 只是短期使用，大部分情况生命周期只有一次浏览而已。而 `max-age=0` 在语义上普遍被解读为“不要在下次浏览时使用”，所以和 memory cache 并不冲突。

### 2、disk cache

disk cache 也叫 HTTP cache，顾名思义是存储在硬盘上的缓存，因此它是持久存储的，是实际存在于文件系统中的。而且它允许相同的资源在跨会话，甚至跨站点的情况下使用，例如两个站点都使用了同一张图片。

disk cache 会严格根据 HTTP 头信息中的各类字段来判定哪些资源可以缓存，哪些资源不可以缓存；哪些资源是仍然可用的，哪些资源是过时需要重新请求的。当命中缓存之后，浏览器会从硬盘中读取资源，虽然比起从内存中读取慢了一些，但比起网络请求还是快了不少的。绝大部分的缓存都来自 disk cache。

#### 2.1 强制缓存 (也叫强缓存)

强制缓存的含义是，当客户端请求后，会先访问缓存数据库看缓存是否存在。如果存在则直接返回；不存在则请求真的服务器，响应后再写入缓存数据库。

**强制缓存直接减少请求数，是提升最大的缓存策略。** 它的优化覆盖了文章开头提到过的请求数据的全部三个步骤。如果考虑使用缓存来优化网页性能的话，强制缓存应该是首先被考虑的。

##### Expires

这是 **HTTP 1.0** 的字段，表示缓存到期时间，是一个绝对的时间 (当前时间+缓存时间)，如

```text
Expires: Thu, 10 Nov 2017 08:45:11 GMT
```

在响应消息头中，设置这个字段之后，就可以告诉浏览器，在未过期之前不需要再次请求。

但是，这个字段设置时有两个缺点：

1.  由于是绝对时间，用户可能会将客户端本地的时间进行修改，而导致浏览器判断缓存失效，重新请求该资源。此外，即使不考虑自信修改，时差或者误差等因素也可能造成客户端与服务端的时间不一致，致使缓存失效。
2.  写法太复杂了。表示时间的字符串多个空格，少个字母，都会导致非法属性从而设置失效。

##### Cache-control

已知Expires的缺点之后，在**HTTP/1.1**中，增加了一个字段Cache-control，该字段表示资源缓存的最大有效时间，在该时间内，客户端不需要向服务器发送请求

这两者的区别就是前者是绝对时间，而后者是相对时间。如下：

```text
Cache-control: max-age=2592000
```

下面列举一些 `Cache-control` 字段常用的值：（完整的列表可以查看 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cache-Control)）

- `max-age`：即最大有效时间，在上面的例子中我们可以看到
- `must-revalidate`：如果超过了 `max-age` 的时间，浏览器必须向服务器发送请求，验证资源是否还有效。
- `no-cache`：虽然字面意思是“不要缓存”，但实际上还是要求客户端缓存内容的，只是是否使用这个内容由后续的对比来决定。
- `no-store`: 真正意义上的“不要缓存”。所有内容都不走缓存，包括强制和对比。
- `public`：所有的内容都可以被缓存 (包括客户端和代理服务器， 如 CDN)
- `private`：所有的内容只有客户端才可以缓存，代理服务器不能缓存。默认值。

#### 2.2 对比缓存 (也叫协商缓存)

当强制缓存失效（超过规定时间）时，就需要使用对比缓存，由服务器决定缓存内容是否失效。

流程上说，浏览器先请求缓存数据库，返回一个缓存标识。之后浏览器拿这个标识和服务器通讯。如果缓存未失效，则返回 HTTP 状态码 304 表示继续使用，于是客户端继续使用缓存；如果失效，则返回新的数据和缓存规则，浏览器响应数据后，再把规则写入到缓存数据库。

**对比缓存在请求数上和没有缓存是一致的**，但如果是 304 的话，返回的仅仅是一个状态码而已，并没有实际的文件内容，因此 **在响应体体积上的节省是它的优化点**。它的优化覆盖了请求数据三个步骤（请求，处理，响应）中的最后一个：“响应”。通过减少响应体体积，来缩短网络传输时间。所以和强制缓存相比提升幅度较小，但总比没有缓存好。

对比缓存是可以和强制缓存一起使用的，作为在强制缓存失效后的一种后备方案。实际项目中他们也的确经常一同出现。

对比缓存有 2 组字段(不是两个)：

##### Last-Modified & If-Modified-Since

1.  服务器通过 `Last-Modified` 字段告知客户端，资源最后一次被修改的时间，例如
    `Last-Modified: Mon, 10 Nov 2018 09:10:11 GMT`
    
2.  浏览器将这个值和内容一起记录在缓存数据库中。
    
3.  下一次请求相同资源时时，浏览器从自己的缓存中找出“不确定是否过期的”缓存。因此在请求头中将上次的 `Last-Modified` 的值写入到请求头的 `If-Modified-Since` 字段
    
4.  服务器会将 `If-Modified-Since` 的值与 `Last-Modified` 字段进行对比。如果相等，则表示未修改，响应 304；反之，则表示修改了，响应 200 状态码，并返回数据。

但是他还是有一定缺陷的：

- 如果资源更新的速度是秒以下单位，那么该缓存是不能被使用的，因为它的时间单位最低是秒。
- 如果文件是通过服务器动态生成的，那么该方法的更新时间永远是生成的时间，尽管文件可能没有变化，所以起不到缓存的作用。

##### Etag & If-None-Match

为了解决上述问题，出现了一组新的字段 `Etag` 和 `If-None-Match`

`Etag` 存储的是文件的特殊标识(一般都是 hash 生成的)，服务器存储着文件的 `Etag` 字段。之后的流程和 `Last-Modified` 一致，只是 `Last-Modified` 字段和它所表示的更新时间改变成了 `Etag` 字段和它所表示的文件 hash，把 `If-Modified-Since` 变成了 `If-None-Match`。服务器同样进行比较，命中返回 304, 不命中返回新资源和 200。

### 3、service worker

上述的缓存策略以及缓存/读取/失效的动作都是由浏览器内部判断 & 进行的，我们只能设置响应头的某些字段来告诉浏览器，而不能自己操作。

但 Service Worker 的出现，给予了我们另外一种更加灵活，更加直接的操作方式。

Service Worker 能够操作的缓存是有别于浏览器内部的 memory cache 或者 disk cache 的。我们可以从 Chrome 的 F12 中，Application -> Cache Storage 找到这个单独的“小金库”。除了位置不同之外，这个缓存是永久性的，即关闭 TAB 或者浏览器，下次打开依然还在(而 memory cache 不是)。有两种情况会导致这个缓存中的资源被清除：手动调用 API `cache.delete(resource)` 或者容量超过限制，被浏览器全部清空。

如果 Service Worker 没能命中缓存，一般情况会使用 `fetch()` 方法继续获取资源。这时候，浏览器就去 memory cache 或者 disk cache 进行下一次找缓存的工作了。注意：经过 Service Worker 的 `fetch()` 方法获取的资源，即便它并没有命中 Service Worker 缓存，甚至实际走了网络请求，也会标注为 `from ServiceWorker`。这个情况在后面的第三个示例中有所体现。

## 九、GET 和 POST 对比

### 作用

GET用于查询数据

POST用于提交数据（请求体）

### 参数

GET放在URL中（请求行），有大小限制（实际上是浏览器对URL的限制，比如chrome把URL长度限制为2M。此外，服务器也有限制，）。参数有中文需要转UTF8

![20201105181244](https://gitee.com/p8t/picbed/raw/master/imgs/20201105181244.jpg)

POST放在请求体，无大小限制。无需转码

### 幂等性与缓存

幂等的意味着对同一URL的多个请求应该返回同样的结果。

GET用于获取资源，对同一资源连续调用多次结果相同，是幂等的。没有副作用，是**可缓存的**。比如查询订单记录。

POST用于添加资源，多次post会添加多个记录，不是幂等的。可能会产生副作用，**不可缓存**。比如，如果下单可以被缓存，那么之后的每次下单直接从缓存取，得到下单成功的信息，显然是不合理的。

具体缓不缓存参照**前端缓存**一节

总的来说，GET是安全的，不会改变服务器的状态。POST则不是。

### 几次请求

一般来说，GET没有请求体，需要的控制信息都在请求行和请求头中了，一次性发送过去就行了。

对于POST请求，可能请求体数据量很大，但服务器又不一定会接受，直接发送可能会造成资源浪费。因此，浏览器可以做个判断，如果POST数据大于1K，先只发送请求头，等服务器验证（服务端返回100说明可以继续），如果允许发送再发送请求体内容。

总的来说，浏览器可能会对POST请求进行优化，导致一次数据包分开发送。

## 十、HTTPS

HTTP安全问题

- 数据传输使用明文
- 无法验证报文完整性，报文可能被篡改
- 不验证通信方的身份，通信方的身份有可能遭遇伪装

HTTPS 并不是新协议，而是让 HTTP 先和 SSL（Secure Sockets Layer）通信，再由 SSL 和 TCP 通信，也就是说 HTTPS 使用了隧道进行通信。

通过使用 SSL，HTTPS 具有了**加密**（防窃听）、**认证**（防伪装）和**完整性保护**（防篡改）。

![image-20201105223758435](https://gitee.com/p8t/picbed/raw/master/imgs/20201105223759.png)

### 加密

对称密钥加密方式的传输效率更高，但是无法安全地将客户端生成的密钥 Secret Key 传输给通信方。而非对称密钥加密方式可以保证传输的安全性，因此我们可以利用非对称密钥加密方式将 Secret Key 传输给通信方。HTTPS 采用混合的加密机制：

- 使用非对称密钥加密方式，传输对称密钥加密方式所需要的 Secret Key，从而保证安全性;

- 获取到 Secret Key 后，再使用对称密钥加密方式进行通信，从而保证效率。（下图中的 Session Key 就是 Secret Key）

  ![20201106160509](https://gitee.com/p8t/picbed/raw/master/imgs/20201106160509.png)

### 认证

通过使用 **证书** 来对通信方进行认证。

数字证书认证机构（CA，Certificate Authority）是客户端与服务器双方都可信赖的第三方机构。

服务器的运营人员向 CA 提出公开密钥的申请，CA 在判明提出申请者的身份之后，会对已申请的公开密钥做数字签名，然后分配这个已签名的公开密钥，并将该公开密钥放入公开密钥证书后绑定在一起。

进行 HTTPS 通信时，服务器会把证书发送给客户端。客户端取得其中的公开密钥之后，先使用数字签名进行验证，如果验证通过，就可以开始通信了。

![20201106163035](https://gitee.com/p8t/picbed/raw/master/imgs/20201106163035.png)

### 数据完整性

SSL 通过对报文hash生成摘要来保证数据完整性，因为传输的是加密报文，而摘要是基于原始数据生成的，因此不怕被人篡改加密报文的同时篡改摘要。

HTTP 也提供了 MD5 报文摘要功能，但不是安全的。例如报文内容被篡改之后，同时重新计算 MD5 的值，通信接收方是无法意识到发生了篡改。

HTTPS 的报文摘要功能之所以安全，是因为它结合了加密和认证这两个操作。试想一下，加密之后的报文，遭到篡改之后，也很难重新计算报文摘要，因为无法轻易获取明文。

## 攻击技术

### 1、XSS

跨站脚本攻击（`Cross-Site Scripting, XSS`），可以将代码注入到用户浏览的网页上，这种代码包括 `HTML`和 `JavaScript`。

**分类**

- 反射型：直接注入到`URL`参数
- 存储型：脚本代码注入到数据库，有用户访问时，自动执行脚本
- `DOM`型：不经过后端，直接在前端操作`DOM`元素

**解决方案**

- 对特殊字符编码。或者设置标签白名单，只允许用户使用指定的标签
- 设置`Cookie`为`HttpOnly`，禁止通过`JS`获取`Cookie`

### 2、CSRF

跨站请求伪造（`Cross-site request forgery，CSRF`），是攻击者通过一些技术手段欺骗用户的浏览器去访问一个自己曾经认证过的网站并执行一些操作（如发邮件，发消息，甚至财产操作如转账和购买商品）。由于浏览器曾经认证过，所以被访问的网站会认为是真正的用户操作而去执行。

`XSS`利用的是用户对指定网站的信任，`CSRF`利用的是网站对用户浏览器的信任。

**解决方案**

- 使用验证码，确保是用户行为，而非黑客行为
- 检查`Referer`首部字段，检验访问站点来源
- 使用`Anti CSRF Token`，服务端给网页表单设置一个`token`值，表单提交时把这个`token`值带回去，服务端进行验证
- 加入自定义`Header`

## 参考资料

[HTTP/2](https://hpbn.co/http2)

[CS-Notes](https://cyc2018.github.io/CS-Notes/#/notes/HTTP)

[一文读懂前端缓存](https://www.zhihu.com/collection/604768854)

[GET 和 POST 到底有什么区别？](https://www.zhihu.com/question/28586791)