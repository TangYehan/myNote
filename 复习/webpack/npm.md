### -d & -s

+ `npm -s`：保存到`dependencies`里，生产环境下也会使用到。
+ `npm -d`：保存到`devDependencies`里，仅在开发环境下使用。例如gulp,babel,webpack等辅助性工具，在实际运行中不需要，所以用-D
+ 啥也不写不会进入到`package.json`中



### npm & npx

+ [阮一峰npx介绍](http://www.ruanyifeng.com/blog/2019/02/npx.html)

+ npx主要解决的时项目内模块调用的问题。npx将检查<命令>是否存在于$PATH或本地项目bin文件中，并执行该命令。如果没有找到<命令>，将在执行之前安装它。

  ```js
  # 手动档执行
  $ node node_modules/.bin/mocha **/*.test.js
  
  # 改装党,配置 zsh 别名：alias n='PATH=$(npm bin):$PATH'
  $ n mocha **/*/test.js
  
  #{scripts:{"test":"mocha **/*.test.js"}}
  $ npm test
  
  # 临时执行某个bin
  $ npx mocha **/*.test.js
  ```

+ npx避免全局安装的模块

+ npx可以规定使用或不使用本地安装的模块（-no-install & -ignore-existing)