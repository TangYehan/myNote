### 什么是webpack

+ 这个过程核心完成了 **内容转换 + 资源合并** 两种功能



### 基本结构

```
module.exports = {
  entry: "./src/index.js",
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname, "dist"),
  },
  mode: "development",
  module: {
    rules: [
      {
        test: /\.txt$/i,
        use: ["a-loader", "b-loader", "c-loader"],
      },
    ],
  },
  resolveLoader: {
    modules: [
      path.resolve(__dirname, "node_modules"),
      path.resolve(__dirname, "loaders"),
    ],
  },
  plugins:[]
};
```



### style-loader 

核心逻辑相当于：

```js
const content = `${样式内容}`
const style = document.createElement('style');
style.innerHTML = content;
document.head.appendChild(style);
```



### 资源模块配置



### 优化

+ https://juejin.cn/post/6844904071736852487#heading-14
