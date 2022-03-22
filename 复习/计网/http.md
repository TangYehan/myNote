### [HTTP的发展](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/Evolution_of_HTTP)

+ [参考资料](https://juejin.cn/post/6844903489596833800#comment)
+ [参考资料（推荐）](https://juejin.cn/post/6844904001528397837#heading-3)

#### HTTP/1.0 VS HTTP/1.1

+ 增加了内容协商机制：通过`request-header`中`Accept-xxx`等字段进行协商
+ 增加了额外的缓存控制机制：`cache-control`
+ 连接可复用：HTTP 1.0一个请求结束后TCP断开连接，然后重新链接进行下一次通信。HTTP 1.1支持长连接：即一个TCP连接上可以传输多个HTTP请求和响应，但是串行方式（一个响应回来了才能进行下一次请求）
+ 增加管线化技术：允许在第一个应答被完全发送之前就发送第二个请求，以降低通信延迟。[HTTP队头阻塞](https://juejin.cn/post/7049296242924322830#heading-0)(管道化要求服务端按照请求发送的顺序返回响应（FIFO），原因很简单，HTTP请求和响应并没有序号标识，无法将乱序的响应与请求关联起来)
+ 支持响应分块
+ 是不同域名能配置在同一IP

#### HTTP/1.1 VS HTTP/2

+ HTTP/2是二进制协议而不是文本协议。TTP1.x的解析是基于文本。基于文本协议的格式解析存在天然缺陷，文本的表现形式有多样性，要做到健壮性考虑的场景必然很多，二进制则不同，只认0和1的组合。基于这种考虑HTTP2.0的协议解析决定采用二进制格式，实现方便且健壮。
+ 多路复用。同一TCP连接上，不用等到上一个请求响应再进行下一次请求。而是并行发送，每个request对应一个Id。
+ 压缩headers。HTTP2.0使用encoder来减少需要传输的header大小，通讯双方各自cache一份header fields表，既避免了重复header的传输，又减小了需要传输的大小。
+ 允许服务器推送。

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2017/8/3/718e6c0340dc43ff55af6f7f08965256~tplv-t2oaga2asx-watermark.awebp)



##### 多路复用的实现

+ 主要是依靠HTTP2.0新增的二进制分帧层

+ 首先，浏览器准备好请求数据，包括了请求行、请求头等信息，如果是POST方法，那么还要有请求体。

+ 这些数据经过二进制分帧层处理之后，会被转换为一个个带有请求ID编号的帧，通过协议栈将这些帧发送 给服务器。

+ 服务器接收到所有帧之后，会将所有相同ID的帧合并为一条完整的请求信息。

+ 然后服务器处理该条请求，并将处理的响应行、响应头和响应体分别发送至二进制分帧层。

+ 同样，二进制分帧层会将这些响应数据转换为一个个带有请求ID编号的帧，经过协议栈发送给浏览器。

+ 浏览器接收到响应帧之后，会根据ID编号将帧的数据提交给对应的请求。

  

### HTTP3



### HTTP之缓存

+ 参考资料
  + [图解HTTP缓存](https://juejin.cn/post/6844904153043435533)
  + [一文读懂前端缓存](https://juejin.cn/post/6844903747357769742#heading-2)
+ 通常来说`CSS` `JS` `HTML`图片资源等，都会被浏览器自动缓存
+ 较大的文件存在`disk cache`里，较小的资源存在`memory cache`
+ 主要作用是可以加快资源获取速度，提升用户体验，减少网络传输，缓解服务端的压力。

![缓存运作整体流程图](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/5/10/171fea0fec0b4668~tplv-t2oaga2asx-watermark.awebp)

#### 强缓存

+ 第一次请求资源时，浏览器会根据响应头`response headers`来判断哪些资源需要缓存，缓存到哪里
+ 当第二次请求相同资源时，先查找浏览器缓存（按顺序查找: Service Worker-->Memory Cache-->Disk Cache-->Push Cache）。若存在对应缓存，则直接返回资源，状态码200。若不存在，根据协商缓存重新请求（状态码200）或返回本地资源（状态码304）。

##### expires

+ 优先级最低

+ **HTTP 1.0**控制网页缓存的字段，值为一个日期（请求结果缓存到期的时间，在此时间之前的请求不会请求服务器，直接返回）
+ 缺点是通过本地时间判断，时间无法统一

##### Cache-Control

+ **HTTP 1.1**控制网页缓存的字段

+ 主要取值有
  + public：可以被任何对象缓存（客户端，代理服务器CDN等。如果被代理服务器缓存，可实现资源共享）
  + private：只能被客户端缓存
  + no-cache：要求使用缓存数据前，先把缓存资源发送给服务端判断是否过期（协商缓存）
  + no-store：不使用缓存
  + max-age=<seconeds>，设置缓存存储的最大周期，超过这个时间缓存被认为过期(单位秒)
  + `must-revalidate`：如果超过了 `max-age` 的时间，浏览器必须向服务器发送请求，验证资源是否还有效。
  + cache-control :no-cache 和 max-age = 0有什么区别？
    + max-age = 0的意思是缓存到期了，应该去重新验证
    + 而no-cache的意思是，必须去重新验证
    + 如果是 `max-age=0, must-revalidate` 就和 `no-cache` 等价了

##### pragma：no-cache

+ 优先级最高

+ **HTTP 1.0**控制网页缓存的字段

+ 与 Cache-Control: no-cache 效果一致。强制要求缓存服务器在返回缓存的版本之前将请求提交到源头服务器进行验证。



#### 协商缓存

+ 协商缓存是当强缓存失效时，通过携带相关标识字段，询问服务器本地的缓存是否还能使用，若能，返回304，使用本地缓存；若不能，返回200和新资源，更新本地缓存。



标识字段有：

##### [ETag / If-None-Match](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/ETag)

+ 此字段优先级高于`Last-Modified`

+ ETag：`response headers`，值为`W/xxx`（弱验证器）或 一串哈希值（自定义算法[md5]得出hash）。该值为版本标识符。将会在协商缓存时，携带到`if-none-match`或`if-match`字段，到服务器验证缓存是否过期。
+ If-None-Match：`request headers`。值为服务器返回的`ETag`值。仅当服务器上没有任何资源的`ETag`属性值时，才重新返回资源。

##### [Last-Modified / If-Modified-Since](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Last-Modified)

+ Last-Modified:`response headers`。值为资源作出修改的日期和时间。精度比`Etag`低，因此为备用机制。比如有时候加上空格又删除空格，资源实际上没变 ，但更改日期变了。
+ If-Modified-Since：`request headers`。携带上`Last-Modified`的值。





![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c82d0049c3f4f57bf66d8effcb25ed5~tplv-k3u1fbpfcp-watermark.awebp)



#### 缓存位置

+ [缓存位置 ](https://juejin.cn/post/6844904062270308359)
+ memory cache/disk cache/service worker/push cache

#### 缓存方案

+ HTML :协商缓存

+ css/js/图片：强缓存，文件名带上hash

  

### TCP

+ [面试官，不要再问我三次握手四次挥手了](https://zhuanlan.zhihu.com/p/86426969)
+ 三次握手：确认客户端和服务器端的收发能力正常
+ 第三次握手可以携带数据：假如第一次握手可以携带数据的话，那对于服务器是不是太危险了，有人如果恶意攻击服务器，每次都在第一次握手中的SYN报文中放入大量数据。而且频繁重复发SYN报文，服务器会花费很多的时间和内存空间去接收这些报文。第二次握手携带数据，客户端不一定能接收到。而第三次握手的时候，客户端已经可以确定服务器的收发能力正常，可以携带数据也没啥问题。



#### SYN攻击（DDOS攻击）

+ 客户端在短时间内伪造大量不存在的IP向服务端发请求
+ 服务端收到请求要要给出响应，但是由于客户端IP是伪造的，服务端需要不断重发，直到超时，而这些伪造的SYN包长时间占用未连接队列，导致正常的SYN请求因为队列满而被丢弃，从而引起网络拥堵甚至瘫痪。
+ 预防
  + 缩短超时时间
  + 增大半连接数
  + 更高级的过滤网关防护



### HTTPS

+ [看完这篇HTTPS，和面试官扯皮没问题](https://juejin.cn/post/6844904089495535624)

+ HTTP为什么不安全？HTTP是明文传输，任何人都可能从中截获、修改或者伪造，所以不安全。

+ HTTPS就是HTTP的安全版本，HTTP+ （原名SSL）TLS = HTTPS，TLS协议主要是用于计算机之间身份验证和加密的一种协议。
+ HTTPS做了什么？ 加密、数据一致性、身份认证（防止中间人攻击并建立用户信任）
+ SSL 是一个独立的协议，不只有 HTTP 可以使用，其他应用层协议也可以使用



#### [https通信过程](https://www.runoob.com/w3cnote/https-ssl-intro.html)

![](https://pic2.zhimg.com/80/v2-2c5ead7e9e4544335d3db4e5d1d4e355_1440w.jpg)

[TLS握手过程](https://juejin.cn/post/6884813913259524104)

+ 客户端发起https请求（其中包含TCP的三次握手建立连接，是先进行三次握手建立可靠连接之后，再进行TLS四次握手,TLS握手相当于一个身份认证和密匙协商的过程！！！[链接](https://blog.csdn.net/weixin_45439683/article/details/121036633)）。生成随机数R1，并将自己支持的一套加密规则发送给服务端。
+ 2）网站从中选出一组加密算法与HASH算法，并将自己的身份信息以证书的形式发回给浏览器。证书里面包含了网站地址，加密公钥，以及证书的颁发机构等信息。并生成随机数R2，一起发给客户端。 
+ 3）浏览器获得网站证书之后浏览器要做以下工作：
  +  a) 验证证书的合法性（颁发证书的机构是否合法，证书中包含的网站地址是否与正在访问的地址一致等），如果证书受信任，则浏览器栏里面会显示一个小锁头，否则会给出证书不受信的提示。 
  + b)  如果证书受信任，或者是用户接受了不受信的证书，浏览器会生成一串随机数R3，并用证书中提供的公钥加密。
  +  c) 使用约定好的HASH算法和随机数R1,R2,R3生成对称加密的密钥，使用密钥计算握手消息，最后将R3和计算后的握手消息发给服务端。 
+ 4）服务端接收浏览器发来的数据之后要做以下的操作：
  +  a) 使用自己的私钥将信息解密取出随机数R3，根据之前约定的HASH计算和R1,R2,R3计算对称加密密钥。使用密码解密浏览器发来的握手消息，并验证是否与浏览器发来的一致。
  +  b) 使用密码加密一段握手消息，发送给浏览器。 
+ 5）浏览器解密并计算握手消息的HASH，如果与服务端发来的HASH一致，此时握手过程结束，之后所有的通信数据将由之前浏览器生成的随机密码并利用对称加密算法进行加密。 

#### 加密套件列表结构（TLS结构）

```txt
ECDHE-ECDSA-AES256-GCM-SHA384
```

基本格式就是 **密钥交换算法 - 签名算法 - 对称加密算法 - 摘要算法** 组成的一个密码串



#### [HTTPS一般使用的加密与HASH算法](https://juejin.cn/post/6844903638117122056#heading-19)

- 非对称加密算法：RSA，DSA/DSS
- 对称加密算法：AES，RC4，3DES
- HASH算法/摘要算法：MD5，SHA1，SHA256，HMAC
  - MD5：无论是多长的输入，`MD5` 都会输出长度为 `128bits` 的一个串
  - SHA1：比 `MD5` 的 **安全性更强**。对于长度小于 `2 ^ 64` 位的消息，`SHA1` 会产生一个 `160` 位的 **消息摘要**。基于 `MD5`、`SHA1` 的信息摘要特性以及 **不可逆** (一般而言)，可以被应用在检查 **文件完整性** 以及 **数字签名** 等场景。
  - HMAC：基于MD5和SHA1



### [HTTP大文件传输](https://juejin.cn/post/7005347768491311134)

#### 开启`gzip`

+ 在用户的`request headers`中`Accept-Encoding`字段中选取一种服务端和客户端都支持的编码方式，例如`gzip`进行压缩。

+ `gzip`对于图片视频资源不太好用

  + 服务端

  ```js
  const fs = require("fs");
  const zlib = require("zlib");
  const http = require("http");
  const util = require("util");
  const readFile = util.promisify(fs.readFile);
  const gzip = util.promisify(zlib.gzip);
  
  const server = http.createServer(async (req, res) => {
    res.writeHead(200, {
      "Content-Type": "text/plain;charset=utf-8",
      "Content-Encoding": "gzip"
    });
    const buffer = await readFile(__dirname + "/big-file.txt");
    const gzipData = await gzip(buffer);
    res.write(gzipData);
    res.end();
  });
  
  server.listen(3000, () => {
    console.log("app starting at port 3000");
  });
  ```

#### 分块传输

+ `Transfer-Encoding: chunked`数据以一系列分块的形式进行发送。 [`Content-Length`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Length) 首部在这种情况下不被发送。在每一个分块的开头需要添加当前分块的长度，以十六进制的形式表示，后面紧跟着 '`\r\n`' ，之后是分块本身，后面也是'`\r\n`' 。终止块是一个常规的分块，不同之处在于其长度为0。终止块后面是一个挂载（trailer），由一系列（或者为空）的实体消息首部构成。

  ```txt
  HTTP/1.1 200 OK
  Content-Type: text/plain
  Transfer-Encoding: chunked
  
  7\r\n
  Mozilla\r\n
  9\r\n
  Developer\r\n
  0\r\n
  \r\n
  ```

+ 示例(手动chunk)：

  ```js
  http
    .createServer(async function (req, res) {
      res.writeHead(200, {
        "Content-Type": "text/plain;charset=utf-8",
        "Transfer-Encoding": "chunked",
        "Access-Control-Allow-Origin": "*",
      });
      for (let index = 0; index < chunks.length; index++) {
        setTimeout(() => {
          let content = chunks[index].join("&");
          res.write(`${content.length.toString(16)}\r\n${content}\r\n`);
        }, index * 1000);
      }
      setTimeout(() => {
        res.end();
      }, chunks.length * 1000);
    })
    .listen(3000, () => {
      console.log("app starting at port 3000");
    });
  ```

+ 示例（流式传输，无需手动chunk)

  ```js
  http
    .createServer((req, res) => {
      res.writeHead(200, {
        "Content-Type": "text/plain;charset=utf-8",
        "Content-Encoding": "gzip",
      });
      fs.createReadStream(__dirname + "/big-file.txt")
        .setEncoding("utf-8")
        .pipe(zlib.createGzip())
        .pipe(res);
    })
    .listen(3000, () => {
      console.log("app starting at port 3000");
    });
  ```

  

### 队头阻塞

#### TCP队头阻塞

+ 原因：TCP是按序传输的通道，在传输只确认连续片段中的最大值，如果中间片段丢失，后面的片段收到了会存放在缓冲区不会传给应用层进行处理，需要等到丢失片段重发收到后才会进行处理，从而造成阻塞。
+ 解决方法：放弃TCP协议，采用其他协议。（例如QUIC协议：基于UDP的协议，多路复用。[简介](https://zhuanlan.zhihu.com/p/32553477)）

#### HTTP/1.x 队头阻塞

+ 原因：HTTP/1.x中，若干个请求排队串行化单线程处理，后面的请求等待前面请求的返回才能获得执行机会，一旦有某请求超时等，后续请求只能被阻塞，毫无办法，也就是人们常说的线头阻塞
+ 解决办法：
  + 并发连接。建立多个长连接，相当于增加任务队列，不至于一个队列中一个任务阻塞其他所有任务。
  + 域名分片。由于浏览器规定一个域名下最多建立六个长连接，通过多分几个域名，来多建立几个连接。

### HTTP  headers

#### content-type

+ 用于指示资源的MIME类型

+ `application/x-www-form-urlencoded`

  + 该值为表单的默认提交编码方式，提交时遵循以下规则：空格转为`+`号，非字母的其它字符转化为`ASKⅡ`编码，换行转化为`CRLF`，数据项之间以`&`链接，数据名与值以`=`连接

  + 示例

    ```txt
    <form action="https://xxx.com/api/submit" method="post">
        <input type="text" name="name" value="Javon Yan">
        <input type="text" name="age" value="18">
        <button type="submit">Submit</button>
    </form>
    
    
    // Request Header 部分省略
    POST /foo HTTP/1.1
    Content-Length: 37
    content-type: application/x-www-form-urlencoded
    
    // Body
    name=Javon+Yan&age=18
    ```



+ `multipart/form-data`

  + 对于二进制文件或者非 ASCII 字符的传输，`application/x-www-form-urlencoded` 是低效的，因为没必要对其进行编码。对于包含文件、二进制数据、非 ASCII 字符的内容，应该使用 `multipart/form-data`。 `multipart/form-data` 的请求体包含多个部分，需要通过 boundary 字符分割（由浏览器自动生成）。

  + `multipart/form-data` 格式最大的特点在于:每一个表单元素都是独立的资源表述

  + 示例

    ```txt
    <form action="https://xxx.com/api/submit" method="post" enctype="multipart/form-data">
        <input type="text" name="name" value="Javon Yan">
        <input type="text" name="age" value="18">
        <input type="file" name="file">
        <button type="submit">Submit</button>
    </form>
    
    
    请求头及请求体：
    // Request Header 部分省略
    POST /foo HTTP/1.1
    Content-Length: 10240
    Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryujecLxDFPt6acCab
    
    // Body
    ------WebKitFormBoundaryujecLxDFPt6acCab
    Content-Disposition: form-data; name="name"
    
    Javon Yan
    ------WebKitFormBoundaryujecLxDFPt6acCab
    Content-Disposition: form-data; name="age"
    
    18
    ------WebKitFormBoundaryujecLxDFPt6acCab
    Content-Disposition: form-data; name="file"; filename="avatar.png"
    Content-Type: image/png
    
    ... (png binary data) ....
    ------WebKitFormBoundaryujecLxDFPt6acCab--
    ```

+ ` application/json`



### gzip

+ [gzip压缩原理](https://segmentfault.com/a/1190000020386580)
+ 大概就是通过算法替换相同字符串，再重新编码



### 状态码

[状态码](https://www.runoob.com/http/http-status-codes.html)

[301 VS 302](https://blog.csdn.net/zhouchangshun_666/article/details/79354193)

+ 相同：都是重定向，浏览器会根据`location`字段重定向。（用户看到的效果就是他输入的地址A瞬间变成了另一个地址B）
+ 不同：
  + 301是永久重定向，302是临时重定向。
  + 02重定向只是暂时的重定向，搜索引擎会抓取新的内容而保留旧的地址，因为服务器返回302，所以，搜索搜索引擎认为新的网址是暂时的。而301重定向是永久的重定向，搜索引擎在抓取新的内容的同时也将旧的网址替换为了重定向之后的网址。

401

状态码 **`401 Unauthorized`** 代表客户端错误，指的是由于缺乏目标资源要求的身份验证凭证，发送的请求未得到满足。

没有收到可以认证权限的字段。



403

客户端没有权限访问。我的理解是，完成了身份认证过程，发现用户权限不足。



405

method not allow



400

+ 前端提交的字段名称与后台实体类不一致
+ 数据类型问题，比如没把JSON转为字符串
