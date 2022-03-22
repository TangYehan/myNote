### 参考资料

+ [浅说XSS和CSRF](https://juejin.cn/post/6844903638532358151#heading-0)
+ [美团前端安全系列：如何防止XSS攻击](https://tech.meituan.com/2018/09/27/fe-security.html)
+ [美团前端安全系列：如何防止CSRF攻击](https://juejin.cn/post/6844903689702866952)

+ [常见六大Web安全攻防解析](https://juejin.cn/post/6844903772930441230)

### XSS

+ 用户通过网页漏洞，向客户端注入恶意代码，从而在用户浏览网页时，对用户进行控制或者获取用户隐私数据的攻击方式。

#### 分类

##### 反射型

反射型 XSS 的攻击步骤：

1. 攻击者构造出特殊的 URL，其中包含恶意代码。
2. 用户打开带有恶意代码的 URL 时，网站服务端将恶意代码**从 URL 中取出**，拼接在 HTML 中返回给浏览器。
3. 用户浏览器接收到响应后解析执行，混在其中的恶意代码也被执行。
4. 恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。

**需要诱导用户点击URL**

```js
//当用户点击进入http://localhost:5000这个链接时，客户端会收到返回的script内容并执行
const http = require('http');
function handleReequest(req, res) {
    res.setHeader('Access-Control-Allow-Origin', '*');
    res.writeHead(200, {'Content-Type': 'text/html; charset=UTF-8'});
    res.write('<script>alert("反射型 XSS 攻击")</script>');
    res.end();
}

const server = new http.Server();
server.listen(5000);
server.on('request', handleReequest);
```



##### 存储型

存储型 XSS 的攻击步骤：

1. 攻击者将恶意代码提交到目标网站的数据库中。
2. 用户打开目标网站时，网站服务端将恶意代码**从数据库取出**，拼接在 HTML 中返回给浏览器。
3. 用户浏览器接收到响应后解析执行，混在其中的恶意代码也被执行。
4. 恶意代码窃取用户数据并发送到攻击者的网站，或者冒充用户的行为，调用目标网站接口执行攻击者指定的操作。



##### 基于DOM

**属于前端 JavaScript 自身的安全漏洞，而其他两种 XSS 都属于服务端的安全漏洞**

```html
<input id="ipt" />
<button id="send">发请求</button>
<div id="root"></div>

<script>
      const root = document.getElementById("root");
      const ipt = document.getElementById("ipt");
      let value = "";
      ipt.addEventListener("input", function (e) {
        value = e.target.value;
      });

      const send = document.getElementById("send");
      send.addEventListener("click", function () {
        root.innerHTML = `<a href=${value}>click me!</a>`;
      });
</script>

<!--
 当输入'' onclick=alert(/xss/)时，生成的链接将会执行onclick里的内容
-->
```



#### 防止方法

+ 可以看出XSS攻击有两大关键步骤：1、攻击者提交恶意代码；2、浏览器执行恶意代码
+ 针对第一点，可以进行**输入输出过滤**
+ 针对第二点，尽量避免服务端的渲染，改为**前端渲染**
+ [`Content Security Policy`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Security-Policy)：控制资源的来源域

+ `HTTP-only`属性，禁止 JavaScript 读取某些敏感 Cookie，攻击者完成 XSS 注入后也无法窃取此 Cookie。



### CSRF

一个典型的CSRF攻击有着如下的流程：

- 受害者登录a.com，并保留了登录凭证（Cookie）。
- 攻击者引诱受害者访问了b.com
- b.com向 a.com 发送了一个请求：a.com/act=xx。浏览器会…
- a.com接收到请求后，对请求进行验证，并确认是受害者的凭证，误以为是受害者自己发送的请求。
- a.com以受害者的名义执行了act=xx。
- 攻击完成，攻击者在受害者不知情的情况下，冒充受害者，让a.com执行了自己定义的操作。



#### 特点

+ 一般发生在第三方网站
+ 攻击者并不能拿到用户信息，只能”冒名顶替“用户发某些请求（利用`<img> `标签，隐藏的`form`表单自动提交等方式）



#### 防止方法

##### 验证码

##### token认证

##### 同源检测

##### [SameSite](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Set-Cookie/SameSite)

+ 可以对 Cookie 设置 SameSite 属性。该属性表示 Cookie 不随着跨域请求发送，可以很大程度减少 CSRF 的攻击，但是该属性目前并不是所有浏览器都兼容。

##### 使用Origin Header确定来源域名

+ 一般只存在于`cors`跨域中
+ 用来说明请求从哪里发起的，包括，且仅仅包括**协议和域名**。 

+ 如果origin存在直接使用确认来源域名就行。（302重定向，IE11同源策略这两种情况下不存在origin字段）

##### 使用Referer Header确定来源域名

+ 在HTTP头中有一个字段叫Referer，记录了该HTTP请求的来源地址。
+ 包含协议域名和参数
+ 如果origin存在直接使用确认来源域名就行。（协议为本地文件；当前请求页面采用的是非安全协议，而来源页面采用的是安全协议这两种情况下不会发送referer）



### url跳转漏洞

+ 直接利用某些网站的跳转接口，骗取用户信任，然后使用户跳转到不安全的网页，例如`https://link.juejin.cn/?target=https%3A%2F%2Fwww.w3cplus.com%2Fcss3%2Fautoprefixer-css-vender-prefixes.html`，或跳转到`target`所对应的网址去
+ 防护方法：
  + 使用referer header确定来源于域再决定是否跳转，以免被不法分子利用
  + 增加token验证



### SQL注入

+ 防护方法：

  + 严格限制Web应用的数据库的操作权限，给此用户提供仅仅能够满足其工作的最低权限，从而最大限度的减少注入攻击对数据库的危害

  + 后端代码检查输入的数据是否符合预期，严格限制变量的类型，例如使用正则表达式进行一些匹配处理。

  + 对进入数据库的特殊字符（'，"，\，<，>，&，\*，; 等）进行转义处理，或编码转换。基本上所有的后端语言都有对字符串进行转义处理的方法，比如 lodash 的 lodash._escapehtmlchar 库。

  + 所有的查询语句建议使用数据库提供的参数化查询接口，参数化的语句使用参数而不是将用户输入变量嵌入到 SQL 语句中，即不要直接拼接 SQL 语句。例如 Node.js 中的 mysqljs 库的 query 方法中的 ? 占位参数。

  