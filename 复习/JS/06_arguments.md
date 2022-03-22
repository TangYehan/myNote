### 类数组对象

+ 拥有一个`length`属性和若干索引属性的对象
+ 可以像数组那样读取，遍历，修改，但是不能使用数组方法例如`push`等



### arguments对象

```js
function fn(){console.log(arguments)}
fn("hi","arguments!")
/**
0: "hi"
1: "arguments!"
callee: ƒ fn()
length: 2
Symbol(Symbol.iterator): ƒ values()
[[Prototype]]: Object
**/
```

+ `length`为实参的个数，而`函数名.length`为形参的个数

+ `callee`：通过它可以调用函数自身(现在几乎不用)

  ```js
  var data = [];
  
  for (var i = 0; i < 3; i++) {
      (data[i] = function () {
         console.log(arguments.callee.i) 
      }).i = i;
  }
  
  data[0]();
  data[1]();
  data[2]();
  
  // 0
  // 1
  // 2
  ```

  

### arguments 和对应参数的绑定

+ 非严格模式下，arguments和**已传递**形参的值共享

```js
function foo(name, age, sex, hobbit) {
    console.log(name, arguments[0]); // name name
    // 改变形参
    name = 'new name';
    console.log(name, arguments[0]); // new name new name
    // 改变arguments
    arguments[1] = 'new age';
    console.log(age, arguments[1]); // new age new age
    // 测试未传入的是否会绑定
    console.log(sex); // undefined
    sex = 'new sex';
    console.log(sex, arguments[2]); // new sex undefined
    arguments[3] = 'new hobbit';
    console.log(hobbit, arguments[3]); // undefined new hobbit
}

foo('name', 'age')
```

+ 在严格模式下，实参和 arguments 是不会共享的



### arguments转为数组

+ `function fn (...arguments){ console.log(arguments)}`