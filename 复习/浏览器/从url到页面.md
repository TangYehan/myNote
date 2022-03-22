+ http://www.360doc.com/content/18/0520/04/36490684_755346690.shtml

### 网络层面

1、输入`url`

2、构建请求

3、检查是否有强缓存

4、`DNS`解析 (浏览器缓存->hosts文集->本地域名服务器->根域名服务器->顶级域名服务器->权威域名服务器)

6、`TCP`握手

7、发送HTTP请求

8、响应数据

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/15/16f080b095268038~tplv-t2oaga2asx-watermark.awebp)





### 解析层面

1、拿到返回的代码字符串(例如：`type = text/html`)

2、生成DOM树 （即词法解析和语法解析）

+ 标记化
+ 建树

3、样式计算

+ 格式化样式表（综合所有`CSS`来源生成`document.styleSheets`对象）
+ 标准化样式属性（red=>#ff0000）
+ 通过继承和层叠计算每个节点具体样式（挂载到`window.getComputedStyle`）

4、生成布局树



### 渲染过程

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/15/16f080ba7fa706eb~tplv-t2oaga2asx-watermark.awebp)





### URL到页面完整过程

参考：[看评论](https://juejin.cn/post/6928677404332425223#comment)

+ URL判断、解析

  + 判断什么？首先判断用户输入的是合法的URL还是搜索内容，如果是搜索内容就合成URL

  + 为什么要解析？因为根据http协议，参数传输是通过key=value这种键值对的形式，参数之间通过&符号分割，但如果需要传的key或者value中本来就包含这些符号，就会出现歧义，所以需要对URL进行编码转义，在前面加上转义字符%

  + 编码？规定是`0-9a-zA-Z`和几个特殊符号

    + 网址路径的编码为`utf-8`

    + 查询字符串用的是操作系统的默认编码

    + GET和POST方法的编码，用的是网页的编码。

      ...

    + 可以用`encodeURIComponent`保证utf-8编码；与encodeURL区别是对一些保留字符/#的处理不同

+ 构建请求（构建请求信息）

+ 先判断是否有可用的缓存资源（强缓存/协商缓存）

  + 在发出网络请求中之前会先去本地缓存中查找是否有可用的缓存资源，如果有，将缓存资源返回给浏览器；如果没有，进入网络请求阶段

  + （service worker可以理解为一个介于客户端和服务器之间的一个代理服务器。会拦截网络请求，通常用于在离线情况下返回缓存的资源)

    比如拦截客户端的请求、向客户端发送消息、向服务器发起请求等等，离线资源缓存、也就是说当你在浏览器离线时发送一些文件，Service Worker 会在网络连接有效时再把它们上传到外部服务器。只是它的作用之一。 这个缓存是永久性的，即关闭 TAB  或者浏览器，下次打开依然还在(而 memory cache 不是)。有两种情况会导致这个缓存中的资源被清除：手动调用 API  cache.delete(resource) 或者容量超过限制，被浏览器全部清空。 

  + Memory cache ：内存中的缓存

  + disk cache :磁盘缓存

  + push cache(推送缓存，只存在于会话中)

+ DNS解析 例:www.y.abc.com

  + 浏览器缓存中有没有对应IP
  + 操作系统（域名劫持：黑客就该host文件里的IP）
  + 本地域名服务器
  + 本地域名服务器向根域名服务器查询，返回查询`.com`的IP
  + 本地域名服务器向顶级域名服务器查询，返回查看`abc.com`的IP
  + 本地域名服务器去权限域名服务器查询，返回`y.abc.com`的ip
  + 共用8个UDP报文

+ [如果使用了CDN](https://juejin.cn/post/6913704568325046279)

  ![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/6/14/172b2030d04af6c7~tplv-t2oaga2asx-watermark.awebp)

  ![](https://img-blog.csdnimg.cn/20190406100316966.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hlcGhlYw==,size_16,color_FFFFFF,t_70)

  + 找到权威域名服务器后，会查到该域名有个**CNAME**，这个CNAME一般指向CDN的DNS服务器
  + CDN的DNS服务器将CDN的**全局负载均衡设备**IP返回给用户
  + 用户向均衡设配发送对内容URL的请求
  + CDN根据用户地址和所请求的URL向所在**区域负载均衡设备**发起请求
  + 区域均衡设备根据用户IP、URL、各个服务器负载状况等返回一台有用户所需内容的合适服务器IP(减轻服务器压力，减轻网络拥堵)
  + 全局负载均衡设备把服务器的 IP 地址返回给用户。
  + 用户向缓存服务器发起请求

+ 拿到IP发起http请求
  + TCP三次握手 （TLS安全通道连接）
  + 发起请求
  + 拿到响应

+ 拿到响应后
  + 根据`content-type`判断文件类型
    + 如果是stream类，启动下载
    + text/图片类 展示在页面
    + html类 开始解析

+ 解析HTML
  + 一般情况下浏览器的一个tab页面对应一个渲染进程，如果从当前页面打开的新页面并且属于同一站点，这种情况会复用渲染进程，其他情况则需要创建新的渲染进程。
  + 构建DOM树
  + 样式计算 构建出CSSOM渲染树
  + 布局阶段（计算每个元素的具体样式位置和大小，生成绘画记录）
  + 分层（为节点生成图层）
  + 绘制：用浏览器指令逐条绘制页面元素。
  + 光栅化：生成位图数据
  + 合成：所有的图块合并在一起，生成完整的页面

  

重绘回流：

+ **回流**：当页面部分或全部元素的尺寸、结构、或某些属性发生改变时，**重新计算元素的大小和为位置**，重新绘制。
+ **重绘**：页面中元素样式的改变并不影响他在文档流中的位置时（color等），浏览器将新样式赋予给元素并重新绘制。**不需要重新计算**。
+ 回流比重绘代价更高。
+ 如何避免？
  + position absolute或 fixed；脱离文档流
  + dom离线处理：把要操作的dom先设置为display:none ！visibility:hidden只对重绘有影响，不会重排（回流）
  + CSS3硬件加速：（可以让transform/opacity/filters不引起回流重绘）transform/opacity/will-change



### CDN的作用

+ [CDN解析过程](https://mp.weixin.qq.com/s?__biz=Mzg3MjA4MTExMw==&mid=2247486200&idx=1&sn=197c0905028104e1ae32dc6bed7941f5&chksm=cef5f94ef982705874cf1a852e2f3e4879e59cb0705d1aed5a12cd91f845fba1869b58cb8863&scene=21#wechat_redirect)

+ 加速访问，可以就近获取内容

+ 减轻源服务器压力，降低网络拥堵

+ "内容分发网络"就像前面提到的"全国仓配网络"一样，解决了因分布、带宽、服务器性能带来的访问延迟问题，适用于站点加速、点播、直播等场景。使用户可就近取得所需内容，解决 Internet网络拥挤的状况，提高用户访问网站的响应速度和成功率。

  

### 如何判断有没有用CDN

+ 在不同地区ping
+ 如果返回的IP是同一个说明没用，都是源站
+ 如果不同，则是CDN均衡负载系统的功劳

或

+ via是http协议里面的一个header,通过代理和网关记录http请求+ 
+ x-cache用来记录缓存的命中与否 hit （命中）/ miss
