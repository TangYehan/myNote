参考资料:

+ [生么是AST](https://juejin.cn/post/6844903725228621832#heading-1)

+ [参考资料二](https://segmentfault.com/a/1190000016231512)



### AST（抽象语法树）的构建过程

#### 词法解析

+ 扫描代码，按照规则生成一个个标识符（tokens)，同时移除空白符、注释等，生成一个类似于一维数组的列表、

  ```js
  const a = 1
  [{value:'const',type:'keyword'},{value:'a',type:'identifier'}...]
  ```

  

#### 语法解析

+ 将词法分析出来的东西生成树形表达式，并验证有没有语法错误，如果有，则抛出错误。

+ 当生成树的时候，并不是100%转换，会删除掉一些没必要的标识token，比如不完整的括号。



### AST应用

#### babel

+ babel是一个JS编译器，分为解析(parsing)、转译(transforming)、生成(generation)这三个阶段运行代码

+ babel的原理是，先生成AST树，然后根据规则对生成的树进行遍历修改，根据修改好的树生成新的代码

```js
//一个简单的反转变量名的babel转换
import traverse from 'babel-traverse'

//遍历AST树，根据规则修改树
traverse(ast,{
    enter(path){
        if(path.node.type === 'Identifier') path.node.name = name.split("").reverse().join("")
    }
})
```



#### jscodeshift / codemods

+ jscodeshift是一个跑codemods的工具，codemods是一段描述要将AST转化成什么样的代码
+ 可以用这个工具迁移代码



#### prettier

+ 一个格式化代码的工具

+ 首先，将代码生成AST。之后依然是处理AST，最后生成代码。但是，中间过程其实并不像它看起来那么简单。

















