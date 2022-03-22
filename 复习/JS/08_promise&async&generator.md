[参考资料](https://juejin.cn/post/6844904096525189128)

### Promise

```js
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";

class MyPromise {
  constructor(fn) {
    this.PromiseState = PENDING;
    this.value = undefined;
    this.resolveQueue = [];
    this.rejectQueue = [];

    const resolve = (res) => {
      const run = () => {
        if (this.PromiseState !== PENDING) return;
        this.PromiseState = FULFILLED;
        this.value = res;

        while (this.resolveQueue.length) {
          const callback = this.resolveQueue.shift();
          callback(res);
        }
      };
      setTimeout(run);
    };

    const reject = (res) => {
      const run = () => {
        if (this.PromiseState !== PENDING) return;
        this.PromiseState = REJECTED;
        this.value = res;

        while (this.rejectQueue.length) {
          const callback = this.rejectQueue.shift();
          callback(res);
        }
      };
      setTimeout(run);
    };

    try {
      fn(resolve, reject);
    } catch (e) {
      reject(e);
    }
  }

  then(successCallback, failCallback) {
    typeof successCallback !== "function"
      ? (successCallback = (value) => value)
      : null;
    typeof failCallback !== "function"
      ? (failCallback = (reason) => {
          throw new Error(reason instanceof Error ? reason.message : reason);
        })
      : null;

    return new MyPromise((resolve, reject) => {
      const newSuccessCallback = (res) => {
        try {
          const x = successCallback(res);
          x instanceof MyPromise ? x.then(resolve, reject) : resolve(x);
        } catch (e) {
          reject(e);
        }
      };

      const newFailCallback = (res) => {
        try {
          const x = failCallback(res);
          x instanceof MyPromise ? x.then(resolve, reject) : reject(x);
        } catch (e) {
          reject(e);
        }
      };

      if (this.PromiseState === PENDING) {
        this.resolveQueue.push(newSuccessCallback);
        this.rejectQueue.push(newFailCallback);
      } else if (this.PromiseState === FULFILLED) {
        newSuccessCallback(this.value);
      } else {
        newFailCallback(this.value);
      }
    });
  }

  catch(failCallback) {
    return this.then(undefined, failCallback);
  }

  finally(callback) {
    let P = this.constructor;
    return this.then(
      (value) => P.resolve(callback()).then(() => value),
      (reason) =>
        P.resolve(callback()).then(() => {
          throw reason;
        })
    );
  }

  static resolve(value) {
    if (value instanceof MyPromise) return value;
    if (
      Object.prototype.toString.call(value) === "[object Object]" &&
      typeof value.then === "function"
    ) {
      return new MyPromise(value.then);
    } else {
      return new MyPromise((resolve) => resolve(value));
    }
  }

  static reject(value) {
    return new Promise((resolve, reject) => reject(value));
  }

  static all(args) {
    let res = [],
      count = 0;
    return new MyPromise((resolve, reject) => {
      args.forEach((e, index) => {
        MyPromise.resolve(e).then(
          (curres) => {
            res[index] = curres;
            count++;
            if (count === args.length) {
              resolve(res);
            }
          },
          (err) => reject(err)
        );
      });
    });
  }

  static race(args) {
    return new MyPromise((resolve, reject) => {
      args.forEach(
        (e) => {
          MyPromise.resolve(e).then((res) => {
            resolve(res);
          });
        },
        (err) => reject(err)
      );
    });
  }
}

```



### 疑惑

```js
new Promise(resolve => {
 resolve();
})
 .then(() => {
 new Promise(resolve => {
 resolve();
 })
 .then(() => {
 console.log(777);
 })
 .then(() => {
 console.log(888);
 });
 })
 .then(() => {
 console.log(666);
 });  //我的：786 promise:768
```



### 细节问题

+ 如果promise1里resolve方法传递的是promise2，那么promise2的状态会转移给promise1的状态，也就是说promise2的状态决定了promise1的状态

+ 如果Promise状态已经resolve，再抛出错误是无效的。Promise 在`resolve`语句后面，再抛出错误，不会被捕获，等于没有抛出。因为 Promise 的状态一旦改变，就永久保持该状态，不会再变了。

+ 一般来说不在then里定义第二个回调，总是使用catch方法

+ 如果没有使用`catch()`方法指定错误处理的回调函数，Promise 对象抛出的错误不会传递到外层代码，即不会有任何反应，不会让程序终止

+ setTimeout里通过reject抛出的错误会被捕获，而直接抛出的错误不会被捕获。因为相当于是在promise函数体外抛出的，会冒泡到最外层，成了未捕获的错误

+ 后面的then报错与前面的catch无关

+ promise.race返回第一个状态改变的promise,不管是resolve还是reject

+ promise.any返回第一个resolve的promise，如果都reject了才返回reject

+ promise.allSettle返回所有promise处理完后的结果，{status:"fulfillied",value:"value"}/{status:"rejected",reason:"reason"}

  



### async await

```js
function run(gen) {
  return new Promise((resolve, reject) => {
    let g = gen();

    function _next(val) {
      let res = null
      try {
        res = g.next(val);
      } catch (err) {
        reject(err);
      }

      if (res.done) return resolve(res.vale);

      Promise.resolve(res.value).then(
        (res) => _next(res),
        (err) => g.throw(err)
      );
    }

    _next();
  });
}
```



### Generator

+ 原理是把函数分成多个switch-case块，匹配值返回某一块。

+ Generator实现的核心在于**上下文的保存**，函数并没有真的被挂起，每一次yield，其实都执行了一遍传入的生成器函数，只是在这个过程中间用了一个context对象储存上下文，使得每次执行生成器函数的时候，都可以从上一个执行结果开始执行，看起来就像函数被挂起了一样

简单版实现：

```js
// 生成器函数根据yield语句将代码分割为switch-case块，后续通过切换_context.prev和_context.next来分别执行各个case
function gen$(_context) {
  while (1) {
    switch (_context.prev = _context.next) {
      case 0:
        _context.next = 2;
        return 'result1';

      case 2:
        _context.next = 4;
        return 'result2';

      case 4:
        _context.next = 6;
        return 'result3';

      case 6:
      case "end":
        return _context.stop();
    }
  }
}

// 低配版context  
var context = {
  next:0,
  prev: 0,
  done: false,
  stop: function stop () {
    this.done = true
  }
}

// 低配版invoke
let gen = function() {
  return {
    next: function() {
      value = context.done ? undefined: gen$(context)
      done = context.done
      return {
        value,
        done
      }
    }
  }
} 

// 测试使用
var g = gen() 
g.next()  // {value: "result1", done: false}
g.next()  // {value: "result2", done: false}
g.next()  // {value: "result3", done: false}
g.next()  // {value: undefined, done: true}
```



### 异常捕获

参考资料：[优雅处理异常](https://juejin.cn/post/6844903998969872392)

#### async await 异常捕获

```js
function to(promise){
	return promise.then(
    	res => [null,res]
    )
    .catch(err=>[err])
}

async function a(){
    let [err,res] = await to(promise)
    if(err) return alert("one have question")
    
    let [err2,res2] = await to(promise)
    if(err2) return alert("two have question")
}

a().catch(e=>console.log(e))

```



#### promise.all异常捕获

+ 方法一

```js
Promise.all([promise1,promise2].map(item=>item.catch(e=>console.log(e))))
```

这样就算promise1里出现了错误，会先进入catch模块，错误被catch先捕获。这相当于promise.catch.then，then里面收到的就是一个新的reject的promise

+ 方法二

出现错误之后不使用`reject()`，而是使用特殊格式的`resolve`

```js
new promise((resolve,reject)=>{
    if(err) resolve("err")
})
```



+ 方法三：ES2020新特性，[promsie.allSettled](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled)

与 `Promise.all` 一样，参数是一组包含 `Promise` 实例的数组，返回值是一个新的 `Promise` 实例，其实例在调用 `then` 方法中的回调函数的参数仍是一个数组。不同之处在于无论参数实例 `resolve` 还是 `reject` ， `Promise.allSettled` 都会执行 `then` 方法的第一个回调函数（意思就是不会 `catch` 到参数实例的 `reject` 状态），其回调函数的参数返回的数组的每一项是一个包含 `status` 和 `value` 或者 `reason` 的一组对象。 `status` 代表对应的参数实例状态值，取值只有 `fulfilled（resolve状态）` 和 `rejected（reject状态）` ，当 `status` 的值为 `rejected` ，对应的另一个对象属性就是 `reason` 了，也就是被 `reject` 的原因，而成功返回的 `status` 的值则是 `fulfilled` ，对应的另一个对象属性便是 `value` ，对应的值就是 `resolve` 的任意值。

