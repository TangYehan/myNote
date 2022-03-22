参考资料：

+ [手写`webpack`核心原理(牛逼极了！！)](https://juejin.cn/post/6854573217336541192)



### 主要步骤

1. 读取需要解析的文件（入口文件）
2. 将文件解析为`AST`树，以便于收集其中的依赖（import）
3. 将收集到的依赖存储起来，并利用递归，依次解析依赖文件
4. 利用babel将`ES6`语法转化为`ES5`语法
5. 依次执行代码，执行这些代码时会遇到`require/export`无法解析的问题
6. 自己写个`require/export`
7. 成功解析运行完成！！！



### 解析require/exports

```js
const bundle = (file) => {
  const depsGraph = JSON.stringify(parseModules(file));

  return `(function (graph) {
    function require(file) {
      //将相对路径转换为绝对路径，并执行file里面的内容
      function absRequire(relPath) {
        return require(graph[file].deps[relPath]);
      }

      //处理export
      let exp = {};

      //立刻执行file里面的内容，当解析到require时，事实上使用的是absRequire函数解析
      (function (require, exports, code) {
        eval(code);
      })(absRequire, exp, graph[file].code);

      return exp;
    }

    require('${file}');
 })(${depsGraph})`;
```



### 为什么浏览器不能解析`require/exports`

+ 原因在于目前模块之间的依赖树的处理方式上还有一个明显的难题没有解决：
   因为只有在浏览器完全下载完一个js文件，并且宿主引擎解析到`require`或`import`这些关键字的时候，才知道还有依赖需要下载并解析。然而该文件依赖的这个模块可能还依赖于其他模块，理论上依赖树可以有无限长，目前这种依赖的同步加载方式**会带来严重的进程阻塞和极高的网络开销**

+ 目前并没有很好的解决方案使得浏览器端自然的使用各个模块系统，只能使用webpack等工具预先将所有的依赖打包，最终在浏览器换进中运行



### 完整代码+理解

```js
const fs = require("fs"); //读取文件
const path = require("path");
const parser = require("@babel/parser"); //解析成AST语法树
const traverse = require("@babel/traverse").default; //用来遍历AST语法树
const babel = require("@babel/core"); //将ES6转化成ES5

const getModuleInfo = (file) => {
  const body = fs.readFileSync(file, "utf-8");

  //转换为AST语法树
  const ast = parser.parse(body, {
    sourceType: "module",
  });

  const deps = {};

  //遍历AST语法树，收集用到的依赖（解析里面的import语句）
  traverse(ast, {
    ImportDeclaration({ node }) {
      const dirname = path.dirname(file);
      const abspath = "./" + path.join(dirname, node.source.value);
      deps[node.source.value] = abspath;
    },
  });

  //利用babel ES6->ES5
  const { code } = babel.transformFromAst(ast, null, {
    presets: ["@babel/preset-env"],
  });

  const moduleInfo = { file, deps, code };
  return moduleInfo;
};

//递归用getModuleInfo解析文件（包括文件里的依赖）
const parseModules = (file) => {
  let entry = getModuleInfo(file);
  let temp = [entry];
  let depsGraph = {};

  for (let i = 0; i < temp.length; i++) {
    let deps = temp[i].deps;
    for (let item in deps) {
      if (deps.hasOwnProperty(item)) {
        let res = getModuleInfo(deps[item]);
        temp.push(res);
      }
    }
  }

  temp.forEach((item) => {
    depsGraph[item.file] = {
      deps: item.deps,
      code: item.code,
    };
  });

  return depsGraph;
};

//执行代码的函数,前面两个算是解析代码
//处理两个关键词require（此时的import已经因为babel转换为了require)和export
//因为前面已经解析过每个文件了，包括所有的依赖文件，因此这里从入口文件开始执行代码时，遇到require就相当于直接去执行之前解析的这个依赖文件的代码
const bundle = (file) => {
  const depsGraph = JSON.stringify(parseModules(file));

  return `(function (graph) {
    function require(file) {
      //将相对路径转换为绝对路径，并执行file里面的内容
      function absRequire(relPath) {
        return require(graph[file].deps[relPath]);
      }

      //处理export
      let exp = {};

      //立刻执行file里面的内容，当解析到require时，事实上使用的是absRequire函数解析
      (function (require, exports, code) {
        eval(code);
      })(absRequire, exp, graph[file].code);

      return exp;
    }

    require('${file}');
  })(${depsGraph})`;
};

const content = bundle("./src/index.js");

//将解析完成的代码写道./dist目录下
fs.mkdirSync("./dist");
fs.writeFileSync("./dist/bundle.js", content);
```

