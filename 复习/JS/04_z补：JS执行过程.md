参考：[a](https://juejin.cn/post/6937835082535141389#heading-12)

### 编译阶段

+ 词法分析：生成词法单元（token)

```js
const a = 1
[
    {
        type:keyword,
        value:const
    },
    {
        type:identifier,
        value:a
    },
    {
        type:punctuator,
        value:=
    }
        ...
]
```

+ 语法分析：生成抽象语法树（AST) 不是百分之百转换，会删除没用的标识符等，如果不符合语法规则，则会报错

```js
{
    type:program,
    body:[
        {
            type:variableDeclaretor,
            id:{
                type:itendifier,
                name:name
            },
            init:{
                type
            }
        }
        ...
    ]
}
```

+ 字节码生成（一边解释，一边执行）（词法[静态]作用域形成）



### 执行阶段

+ 执行上下文创建（只有函数执行上下文、全局执行上下文和eval执行上下文）
  + 确定this
  + 词法环境创建
    + 环境记录：`存储变量`和`函数声明`的实际位置，函数环境下包含`auguments`对象。（let/ const /class/function)
    + 外环境引用：访问其外部的环境，全局环境下为null。（实现作用域链的重要部分）。
  + 变量环境创建（其实也是个词法环境）
    + 环境记录（var）
    + 外环境引用





作用域链就是在**执行上下文创建阶段**确定的。有了执行的环境，才能确定它应该和谁构成作用域链。



```js
let a = {
    b:{
        c:()=>{console.log(this)}
    }
}
a.b.c() //window
```

