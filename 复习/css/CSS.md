[复习大纲](https://juejin.cn/post/6844904019165446158#heading-8)

[常见面试题](https://segmentfault.com/a/1190000013325778)

### 盒模型

+ 概念：CSS盒模型本质上是一个盒子，封装周围的HTML元素

+ 组成：`padding 、margin、border、content`其中`margin`可以取负值，注意margin塌陷问题

+ 分类：标准盒模型 和 IE盒模型

  + 标准盒模型（content-box)：一般在现代浏览器中使用的都是正常盒模型content-box。它所说的width一般只包含内容，不包含padding与margin，并且盒子的大小会以内容优先，自动扩展，子元素可以撑开父元素。可以理解为现实生活中的气球，大小可以随内容的变化而变化。

     **盒子总宽度** **= margin + border + padding + width**

     **背景只会在content里**

  + IE盒模型（border-box):父元素的盒模型确定，子元素无法撑开父元素的盒模型。可以理解为现实生活中的铁箱子，大小不能被内容改变。

      **盒子总宽** **= margin + width**
      
      **背景会延申到border里**

+ 设置：`box-sizing:content-box / border-box`

+ [获取元素的宽高](https://www.jb51.net/article/61460.htm)



### 谈一谈BFC

[知乎-10分钟了解BFC](https://zhuanlan.zhihu.com/p/25321647)

[学习BFC](https://juejin.cn/post/6844903495108132877)

[面试BFC](https://juejin.cn/post/6950082193632788493)

+ 常见定位方案有三种：普通文档流，Float浮动布局，绝对定位

+ 概念：`Block Formatting Context`， 名为 "块级格式化上下文"。简单来说就是，`BFC`是一个完全独立的空间（布局环境），让空间里的子元素不会影响到外面的布局。**具有 BFC 特性的元素可以看作是隔离了的独立容器，容器里面的元素不会在布局上影响到外面的元素，并且 BFC 具有普通容器所没有的一些特性。**

+ 触发：**body根元素**，浮动元素，绝对定位元素，`display`为`inline-block / table-cells / flex`的、`overflow`为`hidden`

+ 特性：
  + **同一个**BFC下margin塌陷问题，解决这个问题，可以让两个元素分别处于不同的BFC里
  + BFC可以包含浮动元素（高度会被撑开）
  + 可以阻止元素被浮动元素覆盖     **用来实现两列自适应布局**
  
+ 应用：
  + 防止margin重叠
  
  + 清除内部浮动
  
  + 自适应多栏布局
  
  + 防止字体环绕
  
    

###  margin塌陷问题怎么解决

+ 让两个元素分别在不同的BFC中，即给两个元素添加父元素，并设置`overflow:hidden`等，让他们处于不同的BFC中
+ 两个都设置float，并给下面一个设置clear:both





### 高度塌陷问题怎么解决

+ 高度塌陷大多是由于子元素设置了浮动
+ 解决方案：
  + 通过让父元素形成`BFC`：给父元素添加 overflow:hidden
  + 添加一个块级子元素，并设置：`float:clear`，他就会排到浮动元素下方
  + 不添加子元素，给父元素添加一个为元素`::before`，设置属性:`clear:both;display:block`



### 水平居中方案

+ 方法一：`margin:0 auto ` 上下margin为0，左右根据宽度自适应相同值

+ 方法二：`flex`布局 

  ```css
  display:flex;
  justify-content:center
  ```

+ 方法三：`text-align:center`

+ 方法四：

  ```css
  position: absolute;
  width: 宽度;
  left: 50%;
  margin-left: -0.5*宽度
  ```

+ 方法五：

  ```css
  position: absolute;
  left: 50%;
  transform: translate(-50%, 0);
  ```

+ 方法六：

  ```css
  .son {
      position: absolute;
      width: 宽度;
      left: 0;
      right: 0;
      margin: 0 auto;
  }
  ```



### 垂直居中

+ 行内元素

  ```css
  .parent {
      height: 高度;
  }
  .son {
      line-height: 高度;
  }
  ```

+ table

  ```css
  .parent {
    display: table;
  }
  .son {
    display: table-cell;
    vertical-align: middle;
  }
  ```

+ flex

  ```css
  .parent {
      display: flex;
      align-items: center;
  }
  ```

+ 绝对定位

  ```css
  .son {
      position: absolute;
      top: 50%;
      height: 高度;
      margin-top: -0.5高度;
  }
  =============================
  .son {
      position: absolute;
      top: 50%;
      transform: translate( 0, -50%);
  }
  ==============================
  .son {
      position: absolute;
      height: 高度;
      top: 0;
      bottom: 0;
      margin: auto 0;
  }
  ```

  

### `width:100%`和`width:auto`

+ width:100%：继承父元素的宽度，如果子元素设置了margin \ padding 可能会发生溢出
+ width:auto：加上margin、padding、border和父元素一样宽



### 画⚪🔺正方形等

三角形

```css
border: 50px solid transparent;
width: 0;
height: 0;
border-bottom: 50px solid yellow
```

[正方形](https://blog.csdn.net/weixin_39357177/article/details/81186498)

```css
//方法一：
height :0px;
width:100px;
padding-botton:100px //和宽一样就行
**注：padding-botton设置为百分比是，意思是占父元素width的百分比**

//方法二：
height:100px;
width:100px
```

某比例长方形

```css
.father{
    width:200px
}

.child{
    width:100%;
    height:0px;
    padding-botton:52.65%
}
```





### background属性

+ `background-position`：确定背景图片的哪部分显示到div中，左上为 0% 0%，右下为100% 100%
+ `background-attachment`
  + scroll：随着页面的滚动背景图片将移动
  + inherit：继承初始值scroll
  + fixed：随着页面滚动，背景不动（闵聪博客）[效果代码](https://www.cnblogs.com/71yishen/p/13792550.html)

```css
background-image: url("./world6.jpg");
background-position: center;
background-attachment: fixed;
```



### spirite（雪碧图）

优点

+ 减少网络请求数量
+ 及高压缩比，减少图片大小
+ 全局变换风格方便（例如变个颜色）

缺点

+ 维护麻烦，有修改时，可能需要全局样式重写
+ 图片合并麻烦



### display:none & visibility:hidden & opacity:0

共同点：都让元素不显示到页面上

不同

display:none VS visibility:hidden

+ `visibility:hidden`元素还在渲染树中，且会**占据**该有的**位置**；`display:none`元素不参与渲染，完全从渲染树消失

+ `visibility:hidden`是继承属性，子代是因为继承了父亲的该属性所以才不显示，如果给某个子代设置`visible`，则子代将会显示出来；而`display:none`后子代不显示是因为节点从渲染树消失
+ 将`visibility:hidden`元素变为`visibility:visible`时只触发重绘；而将`display:none`变为`display:block`时将触发重排

- 读屏器不会读取`display: none`;元素内容；会读取`visibility: hidden;`元素内容
- 可以通过JS获取到页面`display:none`的元素

display:none VS visibility:hidden VS opacity:0

+  opacity:0和 visibility:hidden 的特性相似，唯一的不同在于事件添加方面

+ 只有opacity:0的元素可以添加事件，且触发事件。visibility:hidden元素添加事件不会触发





### link & @import

[参考资料](https://www.cnblogs.com/my--sunshine/p/6872224.html)

+ link优于@import
+ link为HTML标签，而@import为CSS提供的语法规则
+ 加载页面时，link标签的加载先于@import的加载；但会被后面的样式覆盖
+ link标签的兼容性更广泛
+ link 标签可以通过 js 动态插入到文档中，@import不可以



### 权重

CSS 权重优先级顺序简单表示为：
`!important > 行内样式 > ID > 类、伪类、属性 > 标签名 > 继承 > 通配符`



### FOUC

(Flash of Unstyled Content)：裸奔的也页面突然闪烁出现样式

https://juejin.cn/post/6844903474954502151#comment

出现原因：

1.使用import方法导入样式表

2.样式放在页面底部

3.不同样式表放在页面不同位置

解决方法：将样式放到head中



### position

+ static：默认值，没有定位，出现在正常流中

+ absolute：相对于第一个非static的父元素进行定位。亲自实验，如果父元素是sticky，子元素是absolute,父元素也会相对sticky元素定位。
+ fixed：生成绝对定位的元素，相对于浏览器窗口进行定位。
+ relative：生成相对定位的元素，相对于其正常位置进行定位。例如设置`relative; top:100px`则会相对于正常位置往下移100px
+ sticky：当父元素没出现在屏幕里时（在屏幕下没滚上来），不受影响。当父元素将要向滚动时，变成类似于`fixed`，固定在top:xxpx的位置。当父元素向上滚动快要滚出屏幕时，会跟着滚动。也就是不超过父元素的上界和下界，固定离浏览器窗口顶部xxpx的位置。
+ inherit：继承父元素的值



### 行内元素 & 块级元素

```txt
|这是行内元素|
|这是块级元素  我有一整行             |
```

+ 块级元素会另起一行，而行内元素只有排不下的时候才会换行

+ 块级元素会撑满整个父元素宽度，而行内元素宽度随着内容变化

+ 块级元素可以设置`width/height`，而且就算设置了宽度也会独占一行；行内元素设置宽度无效

+ 块级元素可以设置margin/padding，行内元素上下margin无效

+ 常见块级元素：`<div><p><li><h>`

+ 常见行内元素：`<span><a><strong><img`

  

### display

+ inline
+ inline-block：inline-block既具有block的宽高特性又具有inline的同行元素特性。
+ block
+ table



### Flex

```css
display:flex;

/*******   容器属性  ********/
flex-direction:决定主轴的方向 row | row-reverse | column | column-reverse

flex-wrap : 如何换行 nowrap | wrap | wrap-reverse

flex-flow: <flex-direction> <flex-wrap>

justify-content:在主轴上的对齐方式 flex-start | flex-end | center | space-between | space-around

align-items : 交叉轴上的对齐方向 flex-start | flex-end | center | baseline（项目第一行文字的基线对齐）  | stretch(拉伸开来)

align-content：定义了多根轴线的对齐方式 flex-start | flex-end | center | space-between | space-around | strech


/********** 项目属性 ************/
order : 数值 （排序越小，越前面）
flex-grow : 数值 （项目放大比例）
flex-shrink : 缩小比例
align-self:单个项目允许和其他项目有不同的对齐方式 auto (默认）auto | flex-start | flex-end | center | baseline | stretch;
flex:<flex-grow> <flex-shrink> <flex-basic>
```



### display:inline-block 什么时候不会显示间隙？

- 移除空格
- 使用`margin`负值
- 使用`font-size:0`
- `letter-spacing`
- `word-spacing`



### 伪类 & 伪元素

+ 伪类：为了通过选择器找到那些不存在与DOM树中的信息以及不能被常规CSS选择器获取到的信息
  + :active
  + :focus
  + :first-child
  + :hover
  + :link
+ 伪元素：在DOM树中创建了一些抽象元素，这些元素不在文档里
  + ::before
  + ::after



### 动画时间

+ 每秒刷新60次 ，即16.7ms



### base64 编码

+ 优势：减少网络请求
+ 劣势：消耗GPU进行编码

  				文件会增大1/3

### 几种常见布局方式

#### 流体布局

+ 两边固定宽高，中间自适应

```html
   <style>
      .left {
        float: left;
        width: 100px;
        height: 200px;
        background: red;
      }
      .right {
        float: right;
        width: 200px;
        height: 200px;
        background: blue;
      }
      .main {
        margin-left: 120px;
        margin-right: 220px;
        height: 200px;
        background: green;
      }
    </style>

    <div class="container">
      <div class="left"></div>
      <div class="right"></div>
      <div class="main"></div>
    </div>


```



### Sass、LESS是什么？大家为什么要使用他们？

- 他们是`CSS`预处理器。他是`CSS`上的一种抽象层。他们是一种特殊的语法/语言编译成`CSS`。
- 例如Less是一种动态样式语言. 将CSS赋予了动态语言的特性，如变量，继承，运算， 函数. 



### 如何使用CSS实现硬件加速？

> 硬件加速是指通过创建独立的复合图层，让GPU来渲染这个图层，从而提高性能，

- 一般触发硬件加速的`CSS`属性有`transform`、`opacity`、`filter`，为了避免2D动画在 开始和结束的时候的`repaint`操作，一般使用`tranform:translateZ(0)`

  

### 浮动元素引起的问题

- 父元素的高度无法被撑开，影响与父元素同级的元素
- 与浮动元素同级的非浮动元素会跟随其后





### 可继承属性 & 不可继承属性

+ 可继承属性
  + font系列 font-size,font-weight,font-style
  + 文本系列 text-align line-height

+ 不可继承属性
  + display
  + width height margin padding background size
