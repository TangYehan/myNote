### 参考资料

+ [fetch MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API/Using_Fetch)
+ [fetch属性配置](https://developer.mozilla.org/zh-CN/docs/Web/API/fetch)
+ [fetch进阶指南](http://louiszhai.github.io/2016/11/02/fetch/)

+ [为什么有了 `XMLHttpRequest`，还要设计一套 `fetch API`?](https://juejin.cn/post/6847009771170562062)



### 基本使用

```js
fetch(url,{
    header:"content-type:application/json",
    method:'post',
    credentials:'no-cache',
    body:JSON.stringify(data)
})
.then(response=>response)
.catch(err=>alert(err))
```



### `fetch`  vs  `xhr`

+ 兼容性：fetch不兼容部分IE低版本
+ `xhr`不符合关注点分离原则（response和request都在同一个对象上）
+ fetch是基于promise的，更加友好

+ ```js
  self.addEventListener('fetch',e=>{
      if(e.request.url === new URL('/',location).href){
          e.responseWith(re Response('<h1>123</h1>'),{headers})
      }
   )
  ```

  这意味着可以直接在客户端实现 `response`，而不是让浏览器去请求网络，这样可以结合 `cache` 实现某些灵活功能，这是 `XHR` 不能实现的。

+ fetch的一些基本特性（比如abort等）应该如何实现备受争议



### 一个`timeout`的实现

```js
function _fetch(fetch_promise,timeout){
    let abort_fn = null
    let abort = new Promsie((resolve,reject)=>{
    	abort_fn = reject
	})
    
    let abortable_promise = Promsie.race([abort,fetch_promise])
    
    setTimeout(()=>{abort_fn("time out")},timeout)
    
    return abortable_promise
}

_fetch(fetch(url,{}),3000)
.then(response=>response)
```

