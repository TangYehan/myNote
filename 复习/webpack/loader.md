参考资料：[a](https://juejin.cn/post/6992754161221632030#heading-0)

### 什么是loader

+ loader本质上是一个函数，作用是把输入里面的内容进行转化

+ **只作用在编译阶段**，作用是文件的转化工作

  ```
  /**
   * @param {string|Buffer} content 源文件的内容
   * @param {object} [map] 可以被 https://github.com/mozilla/source-map 使用的 SourceMap 数据
   * @param {any} [meta] meta 数据，可以是任何内容
   */
  function webpackLoader(content, map, meta) {
    // 你的webpack loader代码
  }
  module.exports = webpackLoader;
  ```

  

+ 在 Webpack 中，loader 可以被分为 4 类：pre 前置、post 后置、normal 普通和 inline 行内。



### loader的大概执行流程

1. 初始化每个loader对象

   ```json
   {
       path:'',
       pitch:'',
       raw:'' ,
       ...
   }
   ```

2. 将初始化的loader对象放入`loaderContext`里

   ```js
   loaderContext.loaders = loader
   loaderContext.index = 0
   ```

3. 选择合适的方法加载第一个loader

   ```js
   require('xxx')
   ```

4. 处理加载的结果，挂载属性

   ```js
   loader.raw = module.row
   loader.pitch = module.pitch
   ```

5. 执行pitch方法，执行时判断有没有返回值。如果没有返回值，处理index++ 个loader；如果有则不往下处理loader，而是开始执行index-- 个loader的 normal 方法（**熔断机制**）

   ```js
   runSyncOrAsync(
   	  fn, ...,
   	   function(err) {
   	     if(err) ...
   	      var args = Array.prototype.slice.call(arguments, 1);
   	      var hasArg = args.some(function(value) {
   		return value !== undefined;
   	      });
   	      if(hasArg) {  //如果有其他参数，是fn的返回值
   		loaderContext.loaderIndex--;
   		iterateNormalLoaders(options, loaderContext, args, callback);//返回去处理Normal
   	       } else {
   		iteratePitchingLoaders(options, loaderContext, callback); //处理下一个loader
   	       }
   	    }
   	  );
       });                                   
   ```

6. 执行Normal Loader ，在这之前，会先调用 `convertArgs` 函数对参数进行处理。

   convertArgs根据raw属性来对args[0]进行处理

   ```js
   function convertArgs(args, raw) {
     if(!raw && Buffer.isBuffer(args[0]))
         args[0] = utf8BufferToString(args[0]);
     else if(raw && typeof args[0] === "string")
         args[0] = Buffer.from(args[0], "utf-8");
   }
   
   // 把buffer对象转换为utf-8格式的字符串
   function utf8BufferToString(buf) {
     var str = buf.toString("utf-8");
     if(str.charCodeAt(0) === 0xFEFF) {
        return str.substr(1);
     } else {
        return str;
     }
   }
   ```

7. 从右往左（index--）执行normal loader

8. 解析返回结果

   在 `this.doBuild` 方法的回调函数中，会使用 `JavascriptParser` 解析器对返回的内容进行解析操作，而底层是通过 [acorn](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Facornjs%2Facorn) 这个第三方库来实现 JavaScript 代码的解析。而解析后的结果，会继续调用 `handleParseResult` 函数进行进一步处理。

   

	### 函数里的this

+ 因为都是通过fn.apply()的方式调用
+ this默认是个空对象

```js
function runSyncOrAsync(fn, context, args, callback) {
	var isSync = true; // 默认是同步类型
	var isDone = false; // 是否已完成
	var isError = false; // internal error
	var reportedError = false;
  
	context.async = function async() {
	  if(isDone) {
	    if(reportedError) return; // ignore
	    throw new Error("async(): The callback was already called.");
	  }
	  isSync = false;
	  return innerCallback;
	};
    
    var innerCallback = context.callback = function() {
	  if(isDone) {
	    if(reportedError) return; // ignore
	    throw new Error("callback(): The callback was already called.");
	  }
	  isDone = true;
	  isSync = false;
	  try {
	    callback.apply(null, arguments);
	  } catch(e) {
	    isError = true;
	    throw e;
	  }
   };
}
```

在 `runSyncOrAsync` 函数内部，最终会通过 `fn.apply(context, args)` 的方式调用 Loader 函数。即会通过 `apply` 方法设置 Loader 函数的执行上下文。



### webpack默认配置是在哪处理的，loader有什么默认配置么？

+ 在webpack初始化的时候，会将用户配置与默认配置合并。里面也包含loader的默认配置，虽然不开放给开发者使用，但是包含了loader的默认配置。