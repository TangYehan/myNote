### 参考视频

+ [`axios`源码分析](https://www.bilibili.com/video/BV1wr4y1K7tq)



### 实现思路

```js
function Axios(config) {
  this.defaults = config;
  this.intercepters = {
    request: new InterceptorManager(), //请求拦截器
    response: new InterceptorManager(), //响应拦截器
  };
}

//拦截器对象
function InterceptorManager() {
   //该对象用于储存拦截器里的成功回调和失败回调们，[{fulfilled,rejected},{fulfilled,rejected}]
  this.handlers = []; 
}

InterceptorManager.prototype.use = function (fulfilled, rejected) {
  this.handlers.push({
    fulfilled,
    rejected,
  });
};

//向原型上添加相关方法
Axios.prototype.request = function (config) {
  console.log("发送请求", config);
  let promise = Promise.resolve(config);

  //undefined是为了占位,因为将拦截器里的函数添加到这里面都是两两成对的，一个成功回调，一个失败回调
  let chains = [dispatchRequest, undefined];

  //将请求拦截器里的回调放到请求前
  this.intercepters.request.handlers.forEach((item) => {
    chains.unshift(item.fulfilled, item.rejected);
  });

  //将响应拦截器里的回调放到请求后
  this.intercepters.response.handlers.forEach((item) => {
    chains.push(item.fulfilled, item.rejected);
  });

  console.log(chains);

  //request拦截器传递的是config
  //response拦截器传递的是结果
  while (chains.length > 0) {
    promise = promise.then(chains.shift(), chains.shift());
  }

  return promise;
};

Axios.prototype.get = function (config) {
  return this.request(config);
};

Axios.prototype.post = function (config) {
  return this.request(config);
};


//声明函数，要求 let axios = createInstance(); 
//既能 axios({url:"xxx",method:"get"})
//又能 axios.get({url:'xxx'})
//因此 axios 实际上等同与request方法，并且在原型上有get,post等方法供使用
function createInstance(config) {
  //实例化对象
  let context = new Axios(config);

  //创建请求函数
  let instance = Axios.prototype.request.bind(context);

  //将Axios原型上的方法添加到instance上
  Object.keys(Axios.prototype).forEach((key) => {
    instance[key] = Axios.prototype[key].bind(context);
  });

  //为instance函数添加属性 default intercepters
  Object.keys(context).forEach((key) => {
    instance[key] = context[key];
  });

  return instance;
}

//dispatchRequest函数
function dispatchRequest(config) {
  return xhrAdapter(config).then(
    (response) => {
      return response;
    },
    (error) => {
      throw error;
    }
  );
}

//xhrAdapter函数，用于创建XMLHttpRequest对象
function xhrAdapter(config) {
  return new Promise((resolve, reject) => {
    let xhr = new XMLHttpRequest();
    xhr.open(config.method, config.url);
    xhr.send();

    xhr.onreadystatechange = function () {
      if (xhr.readyState === 4) {
        if (xhr.status === 200) {
          resolve({
            config: config,
            data: xhr.response,
            Headers: xhr.getAllResponseHeaders(),
            request: xhr,
            status: xhr.status,
            statusText: xhr.statusText,
          });
        } else {
          reject("请求失败");
        }
      }
    };

    //如果传入的配置中有取消请求属性
    if (config.cancelToken) {
      //当cancelToken里维护的promise状态发生变化时，取消请求
      config.cancelToken.promise.then((resolve) => {
        xhr.abort();
      });
    }
    //关于取消请求
  });
}


//取消请求函数
function CancelToken(executor) {
  let resolvePromise;
  this.promise = new Promise((resolve) => {
    //将此resolve函数传递出去，CancelToken实例里维护的promise将保持pending状态，直到调用resolvePromise函数，而调用resolvePromise这个函数时，此promise的状态发生变化，将会调用abort函数取消请求
    resolvePromise = resolve;
  });

  //将改变promise状态的函数传递出去
  executor(function () {
    resolvePromise();
  });
}

/*************************  测试部分  ************************/
let axios = createInstance({ method: "get" });

axios.intercepters.request.use(
  function one() {
    console.log("one resolve");
  },
  function one() {
    console.log("one reject");
  }
);
axios.intercepters.request.use(
  function two() {
    console.log("two resolve");
  },
  function two() {
    console.log("two reject");
  }
);
axios.intercepters.response.use(
  function one() {
    console.log(" resolve1");
  },
  function one() {
    console.log(" reject1");
  }
);
axios.intercepters.response.use(
  function one() {
    console.log("resolve2");
  },
  function one() {
    console.log("reject2");
  }
);

let cancel;
const cancelToken = new CancelToken(function (c) {
  cancel = c;
});
// axios.get({ method: "get",url:'http://localhost:5000/login'});
axios({ method: "post", url: "http://localhost:5000/login", cancelToken }).then(
  (res) => console.log(res)
);

```

