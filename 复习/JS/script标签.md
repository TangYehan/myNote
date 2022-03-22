### script标签的属性

+ async：表示立即开始异步下载脚本，不会阻止其他页面动作（只对外部脚本有效）。会在页面的 load 事件前执行，但可能会在` DOMContentLoaded`之前或之后。
+ charset
+ crossorigin: `crossorigin="anonymous"` 配置文件请求不必设置凭据标 志。 `crossorigin="use-credentials"` 设置凭据标志，意 味着出站请求会包含凭据。
+ defer:立即下载，但解析显示完成后(`</html>`)再执行脚本。都会在 `DOMContentLoaded `事件之前执行。 不过在实际当中，推迟执行的脚本不一定总会按顺序执行或者在` DOMContentLoaded` 事件之前执行，因此最好只包含一个这样的脚本。
+ integrity: 比对接收到的资源和指定的加密签名 以验证子资源完整性
+ language: 弃用
+ src
+ type: 代替language `text/javascript` `module`, `module`模式下才能出现`import/ export`



### 动态添加script属性

+ 动态添加会默认为`async`方式，但不是所有浏览器支持`async`，建议显示设置为`scipt.async = false`

+ 这种方式获取的资源对浏览器预加载器不可见，建议设置

  `<link rel="preload" href="gibberish.js">`



