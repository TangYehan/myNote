+ 什么是`ajax` : Ajax是一种异步请求数据的web开发技术。在不需要重新刷新页面的情况下，Ajax 通过异步请求加载后台数据，并在网页上呈现出来。Ajax的目的是提高用户体验，较少网络数据的传输量。同时，由于AJAX请求获取的是数据而不是HTML文档，因此它也节省了网络带宽，让互联网用户的网络冲浪体验变得更加顺畅

### 参考文献：

+ [Ajax知识体系大梳理](https://juejin.cn/post/6844903469896171533#heading-38)
+ [Ajax原理](https://juejin.cn/post/6844903618764603399#heading-11)

### Ajax的使用

```js
//1、创建XMLHttpRequest对象
var xhr=null;  
if (window.XMLHttpRequest){
    // 兼容 IE7+, Firefox, Chrome, Opera, Safari  
	xhr=new XMLHttpRequest();  
} else{
    // 兼容 IE6, IE5 
    xhr=new ActiveXObject("Microsoft.XMLHTTP");  
} 

//2、规定请求类型，url,是否异步
xhr.open(method,url,async)

//3、设置请求编码类型
xhr.setRequestHeader("content-type","application/x-www-form-urlencoded")

//4、发送请求
xhr.send(null)

//5、状态改变时的回调函数
xhr.onreadystatechange = function(){
    if(xhr.readyState === 4 && xhr.status=== 200 ){
        //xhr.responseText
    }
}
```



### `readyState`

+ `readyState`是`XMLHttpRequest`对象的一个属性，用来标识当前`XMLHttpRequest`对象处于什么状态。` readyState`总共有5个状态值，分别为0~4，每个值代表了不同的含义
  - 0：未初始化 -- 尚未调用.open()方法；
  - 1：启动 -- 已经调用.open()方法，但尚未调用.send()方法；
  - 2：发送 -- 已经调用.send()方法，但尚未接收到响应；
  - 3：接收 -- 已经接收到部分响应数据；
  - 4：完成 -- 已经接收到全部响应数据，而且已经可以在客户端使用了；

+ 每一次`readyState`改变时，都会调用`xhr.onreadystatechange`函数

  

### status

+ `http`状态码，例如404，400，500.....



### 详解

+ 创建`xhr`对象，在`send`之前都是同步进行的。调用`send`方法后，浏览器为将要发生的网络请求创建新的**`http`请求线程**，这个线程独立于`JS`引擎线程，于是网络请求被异步发送出去。此时`JS`引擎线程继续向下执行。等`http`请求收到结果后，相应回调函数被添加到任务队列末尾，直到`JS`引擎线程空闲了，任务队列的任务被执行。

+ 在`onreadystatechange`事件内可能对DOM进行操作，因为GUI渲染线程和`JS`引擎线程是互斥的，`JS`引擎线程将被挂起，转而执行GUI渲染线程，进行重绘或回流。当`JS`引擎重新执行时，GUI线程又被挂起，GUI更新将被保存起来，等`JS`引擎空闲时立即被执行。



### Ajax 与 `setTimeout`

```js
function ajax(url, method){
  var xhr =new XMLHttpRequest();
  xhr.onreadystatechange = function(){
      console.log('xhr.readyState:' + this.readyState);
  }
  xhr.onloadstart = function(){
      console.log('onloadStart');
  }
  xhr.onload = function(){
      console.log('onload');
  }
  xhr.open(method, url, true);
  xhr.setRequestHeader('Cache-Control',3600);
  xhr.send();
}
var timer = setTimeout(function(){
  console.log('setTimeout');
},0);
ajax('https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2017/3/15/c6eacd7c2f4307f34cd45e93885d1cb6.png~tplv-t2oaga2asx-image.image','GET');
console.warn('这里的log并不是最先打印出来的.');
```



执行结果：

```txt
xhr.readyState:1
onloadStart
这里的log并不是最先打印出来的.
setTimeout
xhr.readyState:2
xhr.readyState:3
xhr.readyState:4
onload
```

+ 可见`ajax`并非所有的部分都是异步的, 至少`readyState==1`的 `onreadystatechange` 回调以及 `onloadstart` 回调就是同步执行的. 因此它们的输出排在最前面.























