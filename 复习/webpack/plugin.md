参考资料：

+ [参考资料一](https://juejin.cn/post/6844903713312604173#heading-1)

+ [参考资料二：什么是plugin](https://juejin.cn/post/6844903550623940621)

  





### `plugin`

+ 整个编译周期都在起作用，webpack在运行的生命周期中会广播出许多时间plugin监听这些事件，在合适的时机改变输出结果。

+ webpack相当于一条流水线工程，里面有很多流程，流程之间有依赖关系，必须完成了这个流程才能进入下一个流程。而plugin就相当于在这条流水线里特定的地方插入流程，在特定时机对资源做出修改整理。
+ Webpack 通过 [Tapable](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fwebpack%2Ftapable) 来组织这条复杂的生产线。 Webpack 在运行过程中会广播事件，插件只需要监听它所关心的事件，就能加入到这条生产线中，去改变生产线的运作。 Webpack 的事件流机制保证了插件的有序性，使得整个系统扩展性很好。
+ `webpack`插件是一个具有 `apply` 方法的 JavaScript 对象。`apply 属性会被 webpack compiler 调用`，并且 compiler 对象可在整个编译生命周期访问。