### link标签

+ rel属性（[属性值](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Link_types)表示`<link>`项的链接方式与包含它的文档之间的关系。）
  + [preload](https://developer.mozilla.org/en-US/docs/Web/HTML/Link_types/preload)：很快就会需要的资源，希望能在页面的生命周期中尽早加载，在浏览器的主渲染机制介入前就进行预加载。这一机制使得资源可以更早的得到加载并可用，且更不易阻塞页面的初步渲染，进而提升性能。
  + prefetch：是为了提示浏览器，用户未来的浏览有可能需要加载目标资源，所以浏览器有可能通过事先获取和缓存对应资源，优化用户体验。（网络线程空闲时，会优先加载这些资源）
  + dns-prefetch（实验中）：提示浏览器该资源需要在用户点击链接之前进行 DNS 查询和协议握手。