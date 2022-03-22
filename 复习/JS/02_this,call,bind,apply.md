### this

+ ES5中，this永远指向最后调用他的那个对象，且**运行时**才能确定this指向
+ 匿名函数/回调函数的this指向window
+ ES6箭头函数this的指向在**创建时**确定，指向最近一层非箭头函数的this值
+ `Foo(1)`实际上是`Foo.call(undifined,a)`的简写

```js
function foo(a){
    console.log(this);
}

foo(1)  //输出 window
foo.call(undefined,1) //输出 window
```

+ 如果你传的 context 就 null 或者 undefined，那么 window 对象就是默认的 context（严格模式下默认 context 是 undefined）



### new 操作符

```js
let f = new myFunction("Li","Cherry“)
new myFunction{
    let obj = {};
    obj.__proto__ = myFunction.prototype;
    let result = myFunction.call(obj,"Li","Cherry");
    return typeof result === 'object'? result : obj;
}
```



### call

```js
Function.prototype.my_call = function(context,...args){
    const symbol = Symbol("context");
    context = context === null || context === undefined ? window:context;
    
    //如果为基本数据类型，则转化为object(context)
    if(!/^(object|function)$/i.test(typeof context)){
        context = Object(context);
    }
    
    context[symbol] = this;
    const result = context[symbol](...args);
    delete context[symbol];
    return result
}
```



### apply

```js
Function.prototype.my_apply = function(context,args){
    const symbol = Symbol('context');
    context = context === undefined || context === null ? window : context;
    
    if(!/^(object|function)$/i.test(typeof context)){
        context = Object(context)
    }
    
    context[symbol] = this;
    const result = context[symbol](...args);
    delete context[symbol];
    return result;
}
```



### bind

```js
Function.prototype.bind = function bind(context,...outerArgs){
    let _this = this;
    return function(...innerArgs){
        _this.call(context,...outerArgs.concat(innerArgs))
    }
}
```

