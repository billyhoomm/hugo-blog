

**BFC**（`Block Formatting Context`，块级格式化上下文） 是 CSS 渲染过程中进行布局的盒子，所有浮动子元素都在盒子内进行布局。 也就是说 BFC 内的浮动元素不会影响到 BFC 外部，BFC 外部的环境也不会影响 BFC 内的布局。

 `MDN` 共列出 8 类可以生成 BFC 的元素，包括浮动和绝对定位元素、行内块，以及` overflow` 不为 `visible` 的元素。 比如，设置` overflow:hidden` 可以开启一个 BFC。

> HTML 采用流式布局方式，float属性在这种环境下非常重要。 float 常与 overflow 配合使用都是因为 overflow 会创建新的 BFC，进而影响布局。

## 清除环绕

```xml
<div style="border:1px solid #000;width:300px;">
  <div style="float:left;background: yellow;height:100px;width:150px;">float:left</div> 
  <div> Eros. Nunc ac tellus in sapien molestie rhoncus. Pellentesque nisl. Praesent venenatis blandit velit. Fusce rutrum.  Leo diam interdum ligula, eu scelerisque sem purus in tellus.</div>
</div>
```
代码效果如下：
![](http://cdn.billyhu.com/blogUploads/1527144681000.png)

但是当我们需要分栏布局时，就要清除环绕效果。这里有几种方法：
- 给右侧<div>添加`margin-left`。
- 给父容器的::after设置`clear:both`
- 将右侧div设置为`overflow:hidden`

看到`overflow:hidden`，觉得这个方法很神奇，而本质上是因为它创建了一个BFC，因为BFC达到的效果是内部布局不受外部浮动的影响，所以文字就不会再环绕了：
![](http://cdn.billyhu.com/blogUploads/1527144940000.png)

## 包裹浮动元素

在前端开发中我们经常会碰到一个父元素内只包含浮动元素，但是父元素没有设置宽高，无法撑开子元素：
```xml
<div style="background:red;width:300px;">
  <div style="width:150px;height:100px;float:left;border:1px solid #000;">Child</div>
  Parent
</div>
```

![](http://cdn.billyhu.com/blogUploads/1527145390000.png)

加上`overflow:hidden`

```xml
<div style="overflow:hidden;background:red;width:300px;">
  <div style="width:150px;height:100px;float:left;border:1px solid #000;">Child</div>
  Parent
</div>
```

![](http://cdn.billyhu.com/blogUploads/1527145468000.png)

BFC内部元素不能影响外部，因此浮动子元素不可显示在父元素之外。 而父元素高度为零，难道不显示子元素了吗？ CSS决定这时就用父元素来包裹子元素吧！ 于是父元素便和子元素等高了，红色背景也就显示出来。

## 独立布局环境
`overflow:hidden`在经典的两栏布局中非常有用

当整个页面采用两栏布局：左侧是Sidebar定宽且向左浮动，右侧是Content自适应：
```xml
<div style="width:320px;border:1px solid black;">
  <div style="float:left;width:100px;background:yellow;height:200px;">Sidebar</div>
  <div>
    <p style="background:lightblue;"> User: Alice </p>
    <div style="clear: both;"></div>
    <p style="background:lightblue;"> User: Bob </p>
    <div style="clear: both;"></div>
    <p style="background:lightblue;"> User: Charlie </p>
  </div>
</div>
```
当前情况是我假设右侧列表项内部布局存在浮动，我们需要在每个列表项后面清除浮动，但是这样做会将整个页面布局的浮动清除了，产生了下面的效果：
![](http://cdn.billyhu.com/blogUploads/1527145967000.png)

所以我需要给右侧容器创建独立的布局环境，这时候就用到了`overflow:hidden`：
```xml
<div style="width:320px;border:1px solid black;">
  <div style="float:left;width:100px;background:yellow;height:200px;">Sidebar</div>
  <div style="overflow:hidden">
    <p style="background:lightblue;"> User: Alice </p>
    <div style="clear: both;"></div>
    <p style="background:lightblue;"> User: Bob </p>
    <div style="clear: both;"></div>
    <p style="background:lightblue;"> User: Charlie </p>
  </div>
</div>
```
![](http://cdn.billyhu.com/blogUploads/1527146088000.png)


#### 参考文章：
> MDN：https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context

>  Harttle Land：https://harttle.land/2016/05/11/block-formatting-context.html
